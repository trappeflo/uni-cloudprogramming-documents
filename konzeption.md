# Konzeption: Statische Unternehmenswebsite in der Cloud
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
| Custom Domain | Die Website soll unter einer eigenen Domain erreichbar sein. |
| Globale Erreichbarkeit | Besucher aus der ganzen Welt müssen die Seite ohne spürbare Ladezeiten aufrufen können. |

### 2.2 Nicht-funktionale Anforderungen

| Anforderung | Konkretisierung |
|---|---|
| **Hochverfügbarkeit** | Keine Single Points of Failure. Ziel: SLA ≥ 99,9 %. |
| **Automatische Skalierung** | Lastspitzen müssen ohne manuellen Eingriff abgefangen werden. |
| **Weltweite Performance** | Ladezeiten unabhängig vom geografischen Standort minimal (Ziel: < 100 ms TTFB an Edge-Knoten). |
| **Sicherheit** | Schutz vor DDoS und Layer-7-Angriffen. WAF erforderlich. |
| **Kosteneffizienz** | Serverlose bzw. managed Dienste, kein manuelles Capacity Planning, Pay-per-Use. |
| **Reproduzierbarkeit** | Gesamte Infrastruktur als IaC, vollständig neu aufbaubar. |

### 2.3 Budget / Rahmenbedingungen

- Verfügbares Guthaben: **100 USD** (Azure for Students)
- Azure Front Door und Azure CDN Classic sind für Student-Abos gesperrt (technische Einschränkung)
- Die Infrastruktur wird nach dem Deployment-Test wieder abgebaut (terraform destroy)

---

## 3. Architekturentscheidung

### 3.1 Wahl des Cloud-Anbieters: Microsoft Azure

| Anbieter | Vorteil | Nachteil |
|---|---|---|
| AWS | Sehr großes Service-Angebot | Kreditkarte für Account erforderlich |
| Google Cloud | Gutes Free Tier | Weniger Kursmaterial, kleinere Einsteiger-Community |
| **Azure** | **Studentenguthaben ohne Kreditkarte, passender Service-Stack** | Etwas komplexere IAM-Konzepte |

### 3.2 Architekturevolution: Verworfene Ansätze

**Ansatz 1 (verworfen): Azure Storage + Azure Front Door**
Der ursprüngliche Plan sah Azure Storage Static Website Hosting mit Azure Front Door Standard als CDN/Edge-Schicht vor. Front Door bietet globale PoPs, WAF und HTTPS. Dieser Ansatz wurde verworfen, weil Azure Front Door für Student-Abos gesperrt ist (`BadRequest: Student account is forbidden for Azure Frontdoor resources`).

**Ansatz 2 (verworfen): Azure Storage + Azure CDN Classic**
Als Fallback wurde Azure CDN Classic (Standard_Microsoft) evaluiert. Dieser Ansatz wurde verworfen, weil Microsoft die Erstellung neuer Classic CDN Profile abgekündigt hat (`Azure CDN from Microsoft (classic) no longer support new profile creation`).

**Ansatz 3 (verworfen): Azure Static Web Apps allein**
Azure Static Web Apps ist für Student-Abos verfügbar und kostenlos. Der Dienst bündelt Hosting, integriertes CDN, HTTPS und automatisches Skalieren in einem einzigen verwalteten Dienst. Nachteil: Die CDN-Schicht ist eine Black Box — keine WAF konfigurierbar, keine granulare Kontrolle über Caching und Sicherheitsregeln.

### 3.3 Gewählte Architektur: Azure Static Web Apps + Cloudflare CDN

Die finale Lösung kombiniert **Azure Static Web Apps** als Hosting-Plattform mit **Cloudflare** als vorgelagerter CDN- und Sicherheitsschicht.

| Kriterium | Static Web Apps allein | Static Web Apps + Cloudflare |
|---|---|---|
| CDN | Integriert, nicht konfigurierbar | Cloudflare (300+ PoPs, volle Kontrolle) |
| WAF | Nicht verfügbar | Cloudflare WAF (Free Plan) |
| DDoS-Schutz | Eingeschränkt | Cloudflare DDoS-Schutz (automatisch) |
| Custom Domain | Möglich | Möglich, über Cloudflare DNS verwaltet |
| HTTPS | Automatisch | Cloudflare SSL/TLS (automatisch) |
| IaC-Abdeckung | azurerm Provider | azurerm + cloudflare Provider |
| Kosten | Kostenlos | Kostenlos (Cloudflare Free Plan) |

**Begründung**: Cloudflare ist eines der größten CDN-Netzwerke der Welt mit über 300 PoPs — mehr als Azure Front Door. Durch die Integration werden alle ursprünglichen Anforderungen erfüllt, obwohl Azure-native CDN-Dienste für das Student-Abo nicht verfügbar sind.

---

## 4. Komponentenbeschreibung

### Azure Static Web Apps (Free Tier)
- Hostet die statische HTML-Datei
- Vollständig serverlos, automatisches Skalieren ohne Konfiguration
- Hochverfügbar durch globale Azure-Infrastruktur
- Stellt einen öffentlichen HTTPS-Endpoint bereit (`*.azurestaticapps.net`)

### Cloudflare (Free Plan)
- Sitzt als CDN- und Sicherheitsschicht vor Azure Static Web Apps
- Leitet Besucher zum nächstgelegenen PoP (300+ weltweit)
- Übernimmt HTTPS-Terminierung mit automatisch erneuerten Zertifikaten
- WAF filtert gängige Angriffsmuster (OWASP Top 10)
- DDoS-Schutz auf Layer 3, 4 und 7
- DNS-Verwaltung für die Custom Domain `cloud.trappeonline.xyz`

### Terraform (IaC)
- Zwei Provider: `azurerm` (Azure) und `cloudflare`
- Verwaltet: Resource Group, Static Web App, Cloudflare DNS Record
- Vollständig reproduzierbar: `terraform apply` baut die gesamte Infrastruktur neu auf

---

## 5. Datenfluss

```
Besucher (weltweit)
        │  HTTPS-Anfrage an cloud.trappeonline.xyz
        ▼
Cloudflare Edge PoP (nächstgelegener Standort)
  ├─ Cache Hit  → direkte Auslieferung vom Edge (< 10ms)
  ├─ WAF-Prüfung → bösartige Anfragen werden geblockt
  └─ Cache Miss → Anfrage weitergeleitet an Origin
        │  HTTPS
        ▼
Azure Static Web Apps (Origin)
  └─ Gibt index.html zurück
        │
        ▼
Antwort zurück über Cloudflare Edge an den Besucher
```

---

## 6. Zusammenfassung

| Entscheidung | Gewählt | Begründung |
|---|---|---|
| Cloud-Anbieter | Microsoft Azure | Studentenguthaben ohne Kreditkarte |
| Hosting | Azure Static Web Apps (Free) | Einzige verfügbare Option im Student-Abo mit globalem Hosting |
| CDN / Edge | Cloudflare (Free Plan) | 300+ PoPs, WAF, DDoS-Schutz, vollständig konfigurierbar |
| Custom Domain | cloud.trappeonline.xyz | Bereits bei Cloudflare verwaltet |
| IaC | Terraform (azurerm + cloudflare Provider) | Plattformübergreifend, beide Provider vollständig unterstützt |
| Verworfene Alternativen | Azure Front Door, Azure CDN Classic | Für Student-Abos technisch gesperrt |
