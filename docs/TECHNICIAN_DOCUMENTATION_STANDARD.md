# DocumentationEngine – Techniker-Dokumentationsstandard

Stand: 2026-08-13
Status: **Konsolidierter Zielstandard aus Prototypen; konkrete Template-Technologie offen**

## 1. Zielgruppe

Die primäre Zielgruppe sind Techniker und Administratoren, die eine Kundenumgebung verstehen, betreiben, warten oder im Störungsfall analysieren müssen.

Die Dokumentation ist daher **keine maschinennahe Wiedergabe des Collector-JSON**.

Collector-Daten liefern die Faktenbasis. Die DocumentationEngine strukturiert diese Fakten in eine für Menschen nutzbare Betriebssicht.

---

## 2. Leitfrage

Die Dokumentation soll für einen Techniker zuerst beantworten:

1. Was ist diese Umgebung und wofür dient sie?
2. Wie ist sie grundsätzlich aufgebaut?
3. Welche Workloads/Dienste sind vorhanden?
4. Wie hängen die wesentlichen Komponenten zusammen?
5. Welche Infrastruktur ist für einen bestimmten Workload relevant?
6. Was ist geschützt, überwacht oder automatisiert?
7. Wo finde ich die Detaildaten, wenn ich tiefer analysieren muss?

Die Reihenfolge ist bewusst **Orientierung vor Inventar**.

---

## 3. Zielstruktur eines Kundendokuments

### Kapitel 1 – Kurzüberblick

Inhalt:

- Kunde / Umgebung / Erfassungsstand,
- Scope der Dokumentation,
- wichtigste Workloads,
- wichtigste Plattformkomponenten,
- zentrale externe/on-premises Anbindungen,
- bekannte Datenabdeckung.

Ziel:

Ein neuer Techniker versteht innerhalb kurzer Zeit, womit er es zu tun hat.

### Kapitel 2 – Architekturübersicht

Enthält die High-Level-Standard-View `Azure-Gesamtübersicht` bzw. eine äquivalente herstellerübergreifende Sicht.

Die Übersicht zeigt nur die für Orientierung relevanten Komponenten.

### Kapitel 3 – Netzwerk & Connectivity

Inhalt abhängig von verfügbaren Collector-Daten:

- VNets / Segmente / Subnets,
- Peerings,
- Gateways / Connections,
- On-Premises-Verbindungen,
- NSGs,
- Routing,
- relevante Public-/Private-Endpunkte,
- DNS-bezogene Infrastruktur, soweit erhoben.

Schwerpunkt:

> Wie erreicht ein System ein anderes System?

### Kapitel 4 – Workloads / Anwendungen

Workloads werden nach Betriebsfunktion gruppiert.

Beispiele:

- Azure Virtual Desktop,
- ERP/Sage,
- Domain Services,
- weitere Fachanwendungen.

Für jeden Workload sollen – soweit belegt – beschrieben werden:

- Zweck / Rolle,
- zentrale Ressourcen,
- Netzwerkabhängigkeiten,
- Compute-Abhängigkeiten,
- Storage-Abhängigkeiten,
- Protection/Backup,
- Operations/Monitoring.

Der Zweck eines Workloads darf nur dann als Fakt dargestellt werden, wenn er aus Kundendaten, Konfiguration oder einer anderen freigegebenen Quelle belegt ist. Ressourcennamen allein sind keine hinreichende Grundlage für frei erfundene Funktionsbeschreibungen.

### Kapitel 5 – Backup & Recovery

Die Darstellung startet bei den geschützten Systemen.

Technikerorientierte Reihenfolge:

```text
Ressource
 -> geschützt: ja/nein/soweit erhoben
 -> Policy
 -> Vault
 -> Recovery-Metadaten
```

Die reine Vault-/Policy-Inventarsicht wird nachgelagert.

### Kapitel 6 – Security & Governance

Nur soweit fachlich erhoben:

- RBAC,
- Locks,
- Policy-/Initiative-Assignments,
- Key Vault,
- Defender/Security-Konfiguration,
- weitere Governance-Fakten.

Nicht erhobene Bereiche werden als solche kenntlich gemacht und nicht aus Referenzarchitekturen ergänzt.

### Kapitel 7 – Monitoring & Automation

Nur soweit fachlich erhoben:

- Log Analytics,
- Diagnostic Settings,
- Alerts,
- Action Groups,
- Automation Accounts,
- Runbook-Metadaten,
- Schedules / Associations.

### Kapitel 8 – Betriebsrelevante Zusammenhänge

Ziel dieses Kapitels ist nicht, neue technische Fakten zu erfinden, sondern vorhandene Relationships in eine Technikersicht zu übersetzen.

Beispiele:

- Session Host ist durch eine bestimmte VM bereitgestellt,
- VM verwendet bestimmte NIC/Disks,
- Subnet ist durch NSG/Route Table beeinflusst,
- Protected Item schützt konkrete VM/Storage-Ressource,
- Application Group verwendet konkreten Host Pool.

### Kapitel 9 – Detailinventar / Anhang

Hierhin gehören die maschinennahen Vollständigkeitsdaten:

- Resource Groups,
- Ressourcenlisten,
- IDs,
- SKUs,
- Detailtabellen,
- Relationship-Tabellen,
- technische Konfigurationsfelder.

Das Inventar bleibt wichtig, soll aber die Einstiegskapitel nicht dominieren.

---

## 4. Informationsdichte

### Einstiegsebene

- wenige Kernaussagen,
- Diagramm statt langer Tabellen,
- keine unnötigen ARM IDs,
- keine vollständigen Properties.

### Betriebsebene

- konkrete technische Namen,
- relevante IPs/Netze/Policies/Abhängigkeiten,
- Zustände und Konfigurationen,
- klare Beziehungen.

### Anhang

- maximale belegte technische Detailtiefe,
- IDs und vollständige normalisierte Inventardaten, soweit freigegeben.

---

## 5. Sprachstandard

Die DocumentationEngine soll sachlich und technikerorientiert formulieren.

Bevorzugt:

- "Der Session Host wird durch die Azure-VM `...` bereitgestellt."
- "Die VM verwendet die Netzwerkschnittstelle `...`."
- "Die Ressource ist der Backup Policy `...` zugeordnet."

Zu vermeiden:

- Marketingformulierungen,
- ungesicherte Ursachen-/Risikobewertungen,
- Vermutungen aus Namenskonventionen,
- künstliche Management-Zusammenfassungen ohne Datenbasis.

---

## 6. Fakt, Ableitung und Hinweis

Die Engine sollte langfristig drei Aussageklassen unterstützen:

### Fakt

Direkt aus freigegebenem Input belegt.

### Deterministische Ableitung

Aus expliziten IDs/Relationships eindeutig ableitbar.

Beispiel:

`SessionHost -> BackedByVm -> VM`

### Hinweis / Coverage

Beschreibt Datenabdeckung oder fehlende Erhebung, ohne Infrastruktur zu erfinden.

Beispiel:

`RBAC-Detaildaten sind im aktuellen Input nicht enthalten.`

Freie KI-Inferenz darf nicht stillschweigend als Fakt erscheinen.

---

## 7. Diagramme im Dokument

Standardmäßig vorgesehene Sichten:

1. Azure-/Infrastruktur-Gesamtübersicht,
2. Netzwerk & Connectivity,
3. Workload & Deployment,
4. Backup & Recovery,
5. Security & Operations.

Die genaue Sichtauswahl kann abhängig von den tatsächlich vorhandenen Collector-Domänen erfolgen.

Ein leeres oder nicht erhobenes Fachgebiet darf nicht mit typischen Standardkomponenten aufgefüllt werden.

Details: `DIAGRAM_ENGINE_STANDARD.md`.

---

## 8. Verhalten bei unvollständigen Daten

Erlaubt:

- Abschnitt auslassen,
- Abschnitt als `nicht erhoben` kennzeichnen,
- Coverage-Hinweis,
- unbekannte Einzelwerte leer bzw. als nicht verfügbar behandeln.

Nicht erlaubt:

- Defaultwerte erfinden,
- nicht vorhandene Dienste ergänzen,
- Rollen allein aus Namen als gesichert behaupten,
- fehlende Beziehungen durch typische Azure-Architekturmuster ersetzen.

---

## 9. Outputstrategie

Bereits beschlossen:

- Markdown-first.

Später vorgesehen:

- DOCX,
- PDF,
- ggf. weitere Publishing-Ziele nach separater Architekturentscheidung.

Die fachliche Dokumentstruktur soll unabhängig vom finalen Ausgabeformat bleiben. Dafür ist langfristig ein internes Document View Model sinnvoll; dessen konkrete technische Ausgestaltung ist noch offen.
