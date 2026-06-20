# Cloud Programming (DLBSEPCP01_D) – Tipps & Vorgaben aus den Live-Sessions

Zusammenfassung der wichtigsten Hinweise aus den Intensive-Live-Session-Transkripten (Oktober 2023 & Februar 2026). Ergänzt die offizielle Aufgabenstellung – bei Widersprüchen gilt immer die offizielle Aufgabenstellung.

---

## 1. Worum es im Kurs wirklich geht

- Bewertet wird **nicht die Programmierung der Anwendung selbst, sondern das Deployment** auf einer Cloud-Plattform. Inhaltlich ist das ein **DevOps-Kurs**: Aufgabe eines DevOps-/Cloud-Ingenieurs ist es, ein bereits fertiges Programm zum Laufen zu bringen – nicht die Anwendungslogik zu entwickeln.
- Das Programm muss **lokal bereits laufen**. Ab da setzt der Kurs an: Wie bringe ich dieses lauffähige Programm in eine Cloud-/Web-Umgebung?
- Cloud = "ein Programm, das im Web läuft" – die Kernfrage ist, wer die Hardware besitzt und wie man die Infrastruktur steuert, pflegt und reproduzierbar ändert (→ Automatisierung, IaC).
- Der Kurs ist **rein praktisch**. Theorie nur dort, wo man praktisch gewählte Tools/Dienste **vergleichen und begründen** muss – und auch dann auf praktischer Basis, nicht literaturbasiert.

## 2. Aufgabe 1 vs. Aufgabe 2 – Abgrenzung

- **Aufgabe 1** = einfache **statische** Webseite (z. B. Hello-World HTML), die hochverfügbar gehostet wird.
- **Aufgabe 2** = alles, was darüber hinausgeht. **Sobald etwas Dynamisches dazukommt** (z. B. eine Datenbank, Nutzerdaten speichern), fällt das Projekt automatisch in Aufgabe 2.
- **Wichtig:** In Kategorie 2 zu landen ist **kein Nachteil** – keine Angst davor. Es ist nur eine andere, umfangreichere Kategorie.
- "Einfache Webseite" + "dynamischer Inhalt mit Datenbank" ist ein **Widerspruch**: Entweder statischer Inhalt (nur statische Speicherung, z. B. Blob/S3 + CloudFront/Static Website Service nötig) – oder echter dynamischer Inhalt (dann Backend/DB).
- Tipp: Man kann mit **rein statischem Hosting** sehr dynamisch wirkende Seiten bauen (JavaScript, Single-Page-Application, alles in S3/Blob Storage). Ein **Backend braucht man erst**, wenn Nutzerdaten serverseitig gespeichert werden müssen.

## 3. Eigene Programmidee ≠ Teil der Aufgabe → "Kontext"

- Was du programmiert hast (z. B. SQL-DB mit Bild-URLs + Text, JavaScript-Logik), gehört **nicht zur bewerteten Aufgabe**, sondern ist nur **Kontext**.
- Kontext **kurz** beschreiben – nicht zu ausführlich. Er ist aber wichtig, weil aus ihm die Deployment-Entscheidungen folgen: Welcher Cloud-Dienst? Wird eine DB gebraucht? Welche Kosten? Welche SLA / Verfügbarkeit / Region?

## 4. Anforderungen ernst nehmen und begründen

- Die Aufgabe gibt **Hochverfügbarkeit** vor – du musst dir ein **plausibles Szenario** ausdenken, *warum* die Lösung hochverfügbar (und ggf. geo-redundant) sein muss. Beispiele:
  - Öffentliche Homepage → **weltweit, ohne geografische Beschränkung** verfügbar.
  - Bank-/Finanzanwendung → muss **immer**, aber **nur für authentifizierte Mitarbeiter** verfügbar sein (Security-Fokus statt breiter Verfügbarkeit).
- **SLA** (Service Level Agreement) aus dem Szenario ableiten: Wie gut muss die Verfügbarkeit sein, für wen, in welcher Region?
- Pflicht-Akzeptanzkriterien, die im Portfolio **explizit erläutert und argumentiert** werden müssen: **Hochverfügbarkeit, automatische Skalierung, Sicherheit, Kosteneffizienz**.
- Bei dynamischen Lösungen zusätzlich **DSGVO / Speicherung von Nutzerdaten** thematisieren.

## 5. Architektur & Cloud-Diagramm

- Mindestens **ein Cloud-Diagramm** (UML oder **C4**), das Komponenten-Interaktion und Datenfluss zeigt. Tool: **draw.io / diagrams.net**.
- Bei der typischen 3-Komponenten-Lösung mindestens **drei Boxen** (nach C4): **Frontend, Backend, Datenbank**.
- In jeder Box zusätzlich zeigen:
  - **Welcher Cloud-Dienst** verwendet wird (Logo oder benanntes Kästchen).
  - **Was von dir** dort liegt (z. B. "mein Code / HTML / CSS" in CloudFront; "meine Function" im Function-Dienst).
- Von den vorgefertigten draw.io-Provider-Vorlagen wird **abgeraten** – sie sind zu komplexe Enterprise-Lösungen. Lieber selbst ein schlankes Diagramm bauen.
- Diagramm als **Bild (JPEG/PNG/SVG)** exportieren und in die PDF einbetten. **Alle Abkürzungen ausschreiben**, Text muss gut lesbar sein.

## 6. Phasen sind nur Hilfe – nichts ist bindend bis Phase 3

- **Phase 1 & 2 dienen ausschließlich der Kommunikation mit dem/der Tutor:in** (Feedback). Sie zählen **nicht** für die Note und sind **nicht bindend**.
- Du darfst vor Phase 3 (Endphase) **alles komplett ändern** – sogar das ganze Projekt wechseln. (Beispiel: Phase 1 unter Aufgabe 1 abgegeben, dann in Phase 2 als Aufgabe 2 weitermachen → kein Problem.)
- Feedback wird **im Rahmen der Einreichung** des jeweiligen Portfolioteils gegeben.
- Wenn du **nicht weiterkommst**: die konkrete Schwierigkeit **in Phase 2** (in den Slides) darstellen – Screenshots wo es hängt, Fehlermeldung, deine Lösungsversuche. Dann kann der/die Tutor:in direkt helfen. Reine Fragen im CourseFeed reichen für Ergebnis-Probleme nicht.
- Zusätzliche Kanäle: **CourseFeed** (themen-/aufgabensortiert), **E-Mail an Tutor:in**, **monatliche Live-Session**. Vor den Endergebnissen sollten **alle Fragen geklärt** sein.

## 7. Phase 2 – Aufbau der Präsentation (ca. 10–15 Folien)

Maximal ~15 Folien, **niemals 20+ Seiten** (ein Student reichte mal 100 Seiten ein – nicht korrigierbar). Empfohlene Struktur:

1. **Direkt nach der Outline: das komplette Cloud-Architektur-Diagramm** (verbessert aus Phase 1).
2. **Je Funktion eine Folie zur Dienst-Entscheidung** (Frontend / Backend / Datenbank): Welche Cloud-Dienste verglichen, welche Schlussfolgerung, warum so gewählt? → ca. 3 Folien.
3. **Je Funktion eine Folie zur Konfiguration** (z. B. Function konfiguriert, Backend, Cosmos DB) mit **annotierten Screenshot-Snippets**.
4. Optional: Folie(n) zu **Schwierigkeiten / offenen Fragen** und wie gelöst.
5. Optional: Folie zu **Code / Modulstruktur / Verzeichnisstruktur** der IaC.

### Screenshots richtig machen
- **Nicht jeden Klick** screenshoten. **Default-Einstellungen interessieren nicht** – nur das, was du **selbst spezifiziert/geändert/konfiguriert** hast.
- Pro Folie mehrere **kleine, zugeschnittene Snippets** (nicht der ganze Bildschirm), idealerweise mit **Pfeilen** verkettet und kurzer Beschriftung, was das Snippet bewirkt.
- **Sensible Informationen maskieren.**
- Deployment Schritt für Schritt dokumentieren – von der Registrierung mit E-Mail/Abo beim Anbieter bis zum **Nachweis, dass die Software erfolgreich läuft**.
- Schriftliche Zusammenfassung der wichtigsten Design-/Implementierungsentscheidungen: **ca. ½ DIN-A4-Seite**.

## 8. Infrastructure as Code (Terraform)

- IaC ist **Pflichtbestandteil** (Terraform oder anbieter-spezifisch wie CloudFormation / Bicep). Tools brauchen **keine Admin-Rechte** und sind einfach installierbar (z. B. via Chocolatey auf Windows).
- **`terraform validate`** ausführen und das Ergebnis zeigen (prüft auf Syntaxfehler). Schon in Phase 2 sinnvoll – beweist fehlerfreien Code.
- Terraform arbeitet **deklarativ**: vergleicht deine `.tf`-Dateien mit dem real deployten Zustand und ändert nur Unterschiede. Beim Apply werden **drei Zahlen** angezeigt (add / change / destroy) – auch das zeigen.
- **Sauber strukturieren**: pro Funktion ein eigenes **Modul** (Frontend, Backend, Database, Access). **Variables / Outputs trennen** vom Modulinhalt. Verzeichnisstruktur in einer Folie zeigen.
- **State-Management** ist die größte Terraform-Herausforderung:
  - Allein arbeitend reicht der Default (lokale State-Datei, vom VS-Code-Terraform-Plugin verwaltet, liegt versteckt als `.tfstate` im Verzeichnis).
  - Es gibt zusätzlich eine **Lock-Datei** – relevant, wenn mehrere Personen gleichzeitig deployen (verhindert kollidierende Änderungen).
- **Best Practices** zeigen, besonders **Sicherheit/Secrets**: Passwörter/Secrets nicht hart im IaC, Zugriffskontrolle der Cloud-Dienste sauber regeln.

## 9. Sicherheit (Security Groups, SSH, Keys)

- Jeder Public-Cloud-Anbieter schützt Ressourcen über eine **Security Group** (firewall-ähnlich): definiert erlaubte/verbotene IPs und Ports.
- **Eingehende Ports** bewusst und begründet öffnen, z. B. `443` (HTTPS), `80` (HTTP), `22` (SSH). **Ausgehenden Verkehr** i. d. R. komplett erlauben (damit die Instanz ins Internet kann).
- **SSH-Zugriff** braucht ein **Schlüsselpaar** (asymmetrisch, RSA-Prinzip): **Private Key streng geheim**, niemals teilen; Public Key wird auf den/die Server gelegt.
- Key-Generierung: entweder vom Anbieter generieren lassen (bequem, aber Anbieter hat anfangs Zugriff) oder lokal mit **OpenSSH / PuTTY** generieren. Auf Windows: WSL, OpenSSH oder PuTTY möglich.
- Programm/Daten auf den Server bringen via **SCP/SFTP über SSH** oder über den Browser/CLI des Anbieters.

## 10. Kosten – die größte Stolperfalle 💸

- **Free Tier ≠ kostenlos im Betrieb.** Ein neues Konto bekommt ein Guthaben (z. B. ~200–300 €) zum Ausprobieren; laufende Ressourcen verbrauchen es trotzdem.
- **AWS verlangt eine Kreditkarte** (nur Verifizierung, ~1 Cent Test-Buchung). Alternativ Azure (z. B. Azure for Students).
- Nach jeder Präsentation/Demo **Dienste stoppen und Ressourcen freigeben**, damit keine Kosten entstehen.
- **Achtung – Unterschied stoppen vs. beenden:**
  - **Stoppen** (Stop) = Instanz pausiert, installierte Programme bleiben erhalten – **aber Kosten laufen u. U. weiter** (z. B. Storage). Viele bekommen nach einem Monat eine überraschende Rechnung.
  - **Beenden/Terminieren** = Instanz wird gelöscht, **keine Kosten mehr**, aber **alles Installierte ist weg**.
- Für reine Demo-Zwecke: nach dem Screenshot ruhig **terminieren**.
- Lösung muss **skalierbar UND elastisch** sein (hoch- *und* runterskalieren), damit man nicht dauerhaft für Spitzenlast zahlt. **Budgets/Alarme** (Cost Management) einrichten und im Portfolio erläutern.

## 11. Bewertung – was zählt

| Kriterium | Gewicht |
|---|---|
| Problemabgrenzung / Zielsetzung (inkl. Budget, korrektes C4-Diagramm mit sauberer Kapselung) | 10 % |
| Methodik / Idee / Vorgehen (Entscheidungskriterien erläutert, beste Architektur, Alternativen betrachtet) | 20 % |
| **Qualität der Umsetzung** (erfolgreiches Deployment per Screenshots, gute Doku für Nicht-Techniker, lesbares/reproduzierbares IaC, Best Practices) | **40 %** |
| Kreativität / Richtigkeit (kreativer Ansatz statt Minimallösung, Ziel erfüllt) | 20 % |
| Formale Anforderungen (Formalia eingehalten, Lösung in IaC-Framework entwickelt) | 10 % |

- Höchstes Gewicht liegt auf **Qualität der Umsetzung** – funktionierendes Deployment, saubere Doku und sauberes IaC sind entscheidend.
- Punkte gibt es u. a. dafür, dass **Alternativen verglichen** wurden und die **bestmögliche** Architektur/der beste Anbieter für das Problem gewählt wurde. Kein Bonus für einen bestimmten Anbieter/Tool an sich.
- Lösung muss für **nicht-technische Entscheidungsträger:innen** verständlich beschrieben sein.

## 12. Praktische Tipps zum Schluss

- **Kein Container-/Enterprise-Overkill nötig.** Für Hello-World reicht z. B. eine kleine EC2-Instanz (Amazon Linux, Free-Tier-Typ wie 1 vCPU / 1 GB) oder schlichtes Static Hosting (S3 + CloudFront / Azure Static Website).
- IDE-Empfehlung: **VS Code** mit Terraform-Plugin (managt den State automatisch). Editor-Alternative: Notepad++.
- Wer möchte, kann zusätzlich Provider-Zertifikate (AWS/Azure) machen – **Empfehlung des Dozenten: erst, wenn ein Job sie konkret verlangt** (Zertifikate laufen ab). Für praxisorientierte Jobs ist dieses **Portfolio oft mehr wert als ein Zertifikat**.

---

*Quellen: Intensive-Live-Session Oktober 2023 & Februar 2026 (DLBSEPCP01_D). Inhaltlich abgeglichen mit der offiziellen Aufgabenstellung und den Folien zu Einführung sowie Erarbeitungs-/Reflexionsphase.*
