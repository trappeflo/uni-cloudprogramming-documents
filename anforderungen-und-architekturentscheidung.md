# Anforderungsanalyse und Architekturentscheidung
## Kurs: Cloud Programming (DLBSEPCP01_D) – Aufgabe 1

---

## 1. Szenario

Ein Unternehmen beauftragt die Einrichtung einer Cloud-Architektur für seine öffentliche Unternehmenswebsite. Die Website selbst ist eine einfache statische HTML-Seite (Hello-World-Prototyp). Der Fokus liegt nicht auf der Webentwicklung, sondern auf einer robusten, skalierbaren und weltweit performanten Cloud-Infrastruktur, die über Infrastructure-as-Code (IaC) reproduzierbar ist.

---

## 2. Anforderungsanalyse

### 2.1 Funktionale Anforderungen

| Anforderung | Beschreibung |
|---|---|
| Statisches Hosting | Eine einfache HTML-Datei muss öffentlich erreichbar im Web bereitgestellt werden. |
| HTTPS | Die Website muss über eine gesicherte HTTPS-Verbindung erreichbar sein. |
| Globale Erreichbarkeit | Besucher aus der ganzen Welt (Amerika, Europa, Asien) müssen die Seite ohne spürbare Ladezeiten aufrufen können. |

### 2.2 Nicht-funktionale Anforderungen

| Anforderung | Konkretisierung |
|---|---|
| **Hochverfügbarkeit** | Keine Single Points of Failure; die Seite muss auch bei Ausfall einzelner Rechenzentren oder Availability Zones erreichbar bleiben. Ziel: SLA ≥ 99,9 %. |
| **Automatische Skalierung** | Die Infrastruktur muss Lastspitzen (z. B. plötzlich viele gleichzeitige Besucher) ohne manuellen Eingriff abfangen. Kein manuelles Capacity Planning. |
| **Weltweite Performance** | Ladezeiten sollen unabhängig vom geografischen Standort des Besuchers minimal sein (Ziel: < 100 ms TTFB an Edge-Knoten). |
| **Sicherheit** | Schutz vor gängigen Web-Angriffen (DDoS, Layer-7-Angriffe). Zugriff auf die Ursprungsdaten nur über definierte Kanäle. |
| **Kosteneffizienz** | Da es sich um eine statische Seite ohne Backend-Logik handelt, sollen serverlose bzw. managed Dienste verwendet werden, um Leerlaufkosten zu vermeiden. |
| **Reproduzierbarkeit** | Die gesamte Infrastruktur muss als Code (IaC) vorliegen und in einer neuen Umgebung vollständig neu aufgebaut werden können. |

### 2.3 Budget / Rahmenbedingungen

- Verfügbares Guthaben: **100 USD** (Azure for Students)
- Die Infrastruktur wird nach dem Deployment-Test wieder abgebaut (destroy), um Kosten zu minimieren.
- Keine laufenden VM-Kosten erwünscht; Pay-per-Use bevorzugt.

---

## 3. Wahl des Cloud-Anbieters: Microsoft Azure

### Begründung

Microsoft Azure wurde aus folgenden Gründen gewählt:

- **Kein Kreditkartenzwang**: Azure for Students stellt 100 USD Guthaben bereit, ohne dass eine Kreditkarte hinterlegt werden muss. AWS hingegen verlangt für die Kontoerstellung zwingend eine Kreditkarte.
- **Reife Dienste für statisches Hosting**: Azure bietet einen etablierten Stack aus Storage Static Website Hosting und Azure Front Door, der alle gestellten Anforderungen erfüllt.
- **Gute Terraform-Unterstützung**: Der `azurerm`-Provider für Terraform ist vollständig, gut dokumentiert und aktiv gepflegt.
- **Alternatives IaC-Tool nativ verfügbar**: Azure Bicep steht als herstellereigene Alternative zu Terraform bereit (hier nicht primär eingesetzt, aber erwähnenswert).

### Kurzer Vergleich mit alternativen Anbietern

| Anbieter | Vorteil | Nachteil (für diesen Fall) |
|---|---|---|
| AWS | Sehr großes Service-Angebot, weit verbreitet | Kreditkarte für Account erforderlich |
| Google Cloud | Gutes Free Tier | Kleinere Community-Basis für Einsteiger, weniger IU-Kursmaterial |
| Azure | Kostenloses Studentenguthaben, passender Service-Stack | Etwas komplexere IAM-Konzepte |

---

## 4. Gewählte Architektur: Storage Account + Azure Front Door

### 4.1 Überblick der Komponenten

Die Architektur besteht aus zwei zentralen Azure-Diensten:

1. **Azure Storage Account** (mit Static Website Hosting)
2. **Azure Front Door** (Standard-Tier) als globaler Edge- und CDN-Dienst

### 4.2 Komponentenbeschreibung

#### Azure Storage Account – Static Website Hosting

- Der Storage Account hostet die HTML-Datei in einem speziellen `$web`-Container.
- Der eingebaute Static Website Endpoint stellt die Datei als HTTP bereit.
- **Hochverfügbarkeit**: Durch Zone-Redundant Storage (ZRS) werden die Daten synchron auf drei Availability Zones innerhalb einer Azure-Region repliziert. Fällt eine Zone aus, bleibt der Dienst verfügbar.
- **Automatische Skalierung**: Storage ist ein vollständig verwalteter, serverloser Dienst. Es gibt keine Kapazitätsgrenzen, die manuell angepasst werden müssten — Azure skaliert den Dienst intern.
- **Kein direkter Internetzugriff auf den Origin notwendig**: Der Storage-Endpunkt kann so konfiguriert werden, dass er nur Anfragen von Front Door akzeptiert (Private Link oder Origin-Validierung).

#### Azure Front Door Standard

- Sitzt als globale Schicht vor dem Storage Account.
- **Globale Performance**: Front Door betreibt weltweit verteilte Edge-Knoten (Points of Presence, PoPs). Die statische HTML-Datei wird dort gecacht und von dem PoP ausgeliefert, der dem Besucher geografisch am nächsten liegt. Das reduziert die Latenz dramatisch.
- **HTTPS-Terminierung**: Front Door übernimmt das TLS-Zertifikat (kostenlos, automatisch erneuert) und erzwingt HTTPS auf dem Weg zum Besucher.
- **Sicherheit**: Front Door unterstützt eine optionale Web Application Firewall (WAF), die Angriffsmuster auf Layer 7 filtert und DDoS-Schutz bietet.
- **Failover**: Sollte der primäre Origin (Storage Account) nicht erreichbar sein, kann Front Door auf einen alternativen Origin umleiten (hier nicht konfiguriert, da Single-Region-Setup, aber architektonisch vorbereitet).

### 4.3 Datenfluss

```
Besucher (weltweit)
        │
        ▼
Azure Front Door (Edge PoP – nächstgelegener Standort)
  ├─ Cache Hit → direkte Auslieferung der HTML-Datei vom Edge
  └─ Cache Miss → Anfrage weitergeleitet an Origin
        │
        ▼
Azure Storage Account ($web-Container)
  └─ Static Website Endpoint gibt index.html zurück
        │
        ▼
Antwort zurück über Front Door an den Besucher
```

---

## 5. Warum keine Azure Static Web Apps?

**Azure Static Web Apps** wäre die offensichtliche Alternative: ein einziger Dienst, der Hosting, globale Verteilung, HTTPS und automatisches Skalieren bereits eingebaut hat und im Free Tier komplett kostenlos ist.

Die Entscheidung fiel dennoch gegen Static Web Apps und für die komponierte Lösung aus Storage + Front Door — aus folgenden Gründen:

| Kriterium | Azure Static Web Apps | Storage + Front Door |
|---|---|---|
| Architekturkontrolle | Gering (Black Box) | Hoch (jede Komponente explizit konfiguriert) |
| Nachvollziehbarkeit im Diagramm | Einzelner Dienst, kaum Datenfluss zu zeigen | Klare Schichten: Edge → Origin |
| IaC-Komplexität | Wenige Ressourcen, kaum Lerneffekt | Mehrere Ressourcen, sinnvolle Modulstruktur |
| Begründbarkeit der Service-Auswahl | Entfällt (alles in einem) | Jede Komponente muss explizit ausgewählt und begründet werden |
| Sicherheitskonfiguration | Eingeschränkt | WAF, Origin-Isolation, Header-Policies konfigurierbar |
| Lehrwert | Niedrig | Hoch |

Die Aufgabenstellung bewertet explizit das **Verständnis der Cloud-Architektur**, die **Begründung der Service-Auswahl** und ein **nachvollziehbares Komponentendiagramm mit Datenfluss**. Eine Black-Box-Lösung wie Static Web Apps hätte zwar funktioniert, aber keinen Raum gelassen, diese Anforderungen zu erfüllen. Der Mehraufwand durch die komponierte Lösung ist bewusst gewählt und didaktisch begründet.

---

## 6. Zusammenfassung der Entscheidungen

| Entscheidung | Gewählt | Begründung |
|---|---|---|
| Cloud-Anbieter | Microsoft Azure | Studentenguthaben ohne Kreditkarte |
| Hosting-Dienst | Azure Storage (Static Website) | Serverlos, ZRS-Hochverfügbarkeit, Pay-per-Use |
| Edge/CDN | Azure Front Door Standard | Globale PoPs, HTTPS, WAF, Caching |
| IaC-Tool | Terraform (azurerm Provider) | Plattformübergreifend, weit verbreitet, gut dokumentiert |
| Replikation | Zone-Redundant Storage (ZRS) | Hochverfügbarkeit ohne Multi-Region-Overhead |
| Alternative bewusst abgelehnt | Azure Static Web Apps | Zu wenig Architekturkontrolle für die Lernziele dieses Kurses |
