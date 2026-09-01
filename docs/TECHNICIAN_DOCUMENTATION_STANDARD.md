# DocumentationEngine – Techniker-Dokumentationsstandard

Stand: 2026-09-01
Status: **Konsolidierter Zielstandard aus Prototypen; Actual-/Desired-Perspektivregel ergänzt; konkrete Template-Technologie offen**

## 1. Zielgruppe

Die primäre Zielgruppe sind Techniker und Administratoren, die eine Kundenumgebung verstehen, betreiben, warten oder im Störungsfall analysieren müssen.

Die Dokumentation ist daher **keine maschinennahe Wiedergabe von Collector-JSON oder IaC-Quelltext**.

Technische Quellen liefern die Faktenbasis. Die DocumentationEngine strukturiert diese Fakten in eine für Menschen nutzbare Betriebssicht.

Dabei werden zwei Perspektiven strikt unterschieden:

```text
Actual State
  -> Collector

Desired State
  -> Bicep / IaC
```

Ein Desired-State-Sollbild darf niemals ohne eindeutige Kennzeichnung als gemessener Kunden-Iststand dargestellt werden.

---

## 2. Leitfrage

Die Dokumentation soll für einen Techniker zuerst beantworten:

1. Was ist diese Umgebung und wofür dient sie?
2. Wie ist sie grundsätzlich aufgebaut?
3. Welche Workloads/Dienste sind vorhanden bzw. laut IaC vorgesehen?
4. Wie hängen die wesentlichen Komponenten zusammen?
5. Welche Infrastruktur ist für einen bestimmten Workload relevant?
6. Was ist geschützt, überwacht oder automatisiert?
7. Wo finde ich die Detaildaten, wenn ich tiefer analysieren muss?
8. Wenn Actual und Desired gemeinsam betrachtet werden: Wo stimmt der Iststand mit dem versionierten Sollstand überein bzw. wo ist ein Abgleich nicht möglich?

Die Reihenfolge ist bewusst **Orientierung vor Inventar**.

---

## 3. Zielstruktur eines Kundendokuments

### Kapitel 1 – Kurzüberblick

Inhalt:

- Kunde / Umgebung / Erfassungsstand,
- Scope der Dokumentation,
- verwendete technische Perspektive (`actual`, `desiredTemplate`, `desiredDeployment` oder später `reconciled`),
- wichtigste Workloads,
- wichtigste Plattformkomponenten,
- zentrale externe/on-premises Anbindungen,
- bekannte Datenabdeckung.

Ziel:

Ein neuer Techniker versteht innerhalb kurzer Zeit, womit er es zu tun hat und ob eine dargestellte Aussage gemessener Iststand oder versionierter IaC-Sollstand ist.

### Kapitel 2 – Architekturübersicht

Enthält die High-Level-Standard-View `Azure-Gesamtübersicht` bzw. eine äquivalente herstellerübergreifende Sicht.

Die Übersicht zeigt nur die für Orientierung relevanten Komponenten.

Für IaC-verwaltete Workloads kann zusätzlich eine eindeutig als **Desired State / IaC** gekennzeichnete Architekturübersicht erzeugt werden.

### Kapitel 3 – Netzwerk & Connectivity

Inhalt abhängig von verfügbaren Quellen:

- VNets / Segmente / Subnets,
- Peerings,
- Gateways / Connections,
- On-Premises-Verbindungen,
- NSGs,
- Routing,
- relevante Public-/Private-Endpunkte,
- DNS-bezogene Infrastruktur, soweit erhoben bzw. im Desired State eindeutig deklariert.

Schwerpunkt:

> Wie erreicht ein System ein anderes System?

Actual- und Desired-Netzbeziehungen dürfen nicht ungekennzeichnet vermischt werden.

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

Ein Bicep-Sollvertrag kann zeigen, dass Backup-/Protection-Ressourcen **vorgesehen** sind. Ob ein konkretes System tatsächlich aktuell geschützt ist und Recovery Points besitzt, bleibt eine Actual-State-Aussage und benötigt entsprechende Iststandsdaten.

### Kapitel 6 – Security & Governance

Nur soweit fachlich erhoben bzw. im jeweiligen Desired-State-Vertrag eindeutig deklariert:

- RBAC,
- Locks,
- Policy-/Initiative-Assignments,
- Key Vault,
- Defender/Security-Konfiguration,
- weitere Governance-Fakten.

Nicht erhobene Bereiche werden als solche kenntlich gemacht und nicht aus Referenzarchitekturen ergänzt.

### Kapitel 7 – Monitoring & Automation

Nur soweit fachlich belegt:

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

Für Desired-State-Aussagen ist die Herkunft aus Bicep/IaC explizit kenntlich zu machen.

### Kapitel 9 – Detailinventar / Anhang

Hierhin gehören die maschinennahen Vollständigkeitsdaten:

- Resource Groups,
- Ressourcenlisten,
- IDs,
- SKUs,
- Detailtabellen,
- Relationship-Tabellen,
- technische Konfigurationsfelder,
- IaC-Source-/Commit-/Build-Provenance soweit für Nachvollziehbarkeit relevant und frei von Secrets.

Das Inventar bleibt wichtig, soll aber die Einstiegskapitel nicht dominieren.

---

## 4. Informationsdichte

### Einstiegsebene

- wenige Kernaussagen,
- Diagramm statt langer Tabellen,
- keine unnötigen ARM IDs,
- keine vollständigen Properties,
- Perspektive klar sichtbar.

### Betriebsebene

- konkrete technische Namen,
- relevante IPs/Netze/Policies/Abhängigkeiten,
- Zustände und Konfigurationen,
- klare Beziehungen,
- Actual und Desired sauber getrennt.

### Anhang

- maximale belegte technische Detailtiefe,
- IDs und normalisierte Inventardaten,
- IaC-Provenance und Contract-Metadaten, soweit freigegeben.

---

## 5. Sprachstandard

Die DocumentationEngine soll sachlich und technikerorientiert formulieren.

Bevorzugt Actual State:

- "Der Session Host wird durch die Azure-VM `...` bereitgestellt."
- "Die VM verwendet die Netzwerkschnittstelle `...`."
- "Die Ressource ist der Backup Policy `...` zugeordnet."

Bevorzugt Desired State:

- "Der versionierte IaC-Sollstand sieht die Azure-VM `...` vor."
- "Im Bicep-Deploymentvertrag ist das Subnet `...` der Ressource `...` zugeordnet."

Zu vermeiden:

- Marketingformulierungen,
- ungesicherte Ursachen-/Risikobewertungen,
- Vermutungen aus Namenskonventionen,
- künstliche Management-Zusammenfassungen ohne Datenbasis,
- Formulierungen, die Desired State als gemessenen Iststand erscheinen lassen.

---

## 6. Fakt, Ableitung, Desired State und Coverage

Die Engine soll mindestens folgende Aussageklassen unterscheiden können:

### Fakt / Actual State

Direkt aus freigegebenem Iststandsinput belegt.

### Deterministische Ableitung

Aus expliziten IDs/Relationships eindeutig ableitbar.

Beispiel:

`SessionHost -> BackedByVm -> VM`

### Desired State

Aus einer versionierten IaC-Quelle eindeutig deklariert bzw. deterministisch daraus abgeleitet.

Desired-State-Aussagen tragen die IaC-/Commit-/Build-Provenance und werden nicht als Actual State ausgegeben.

### Hinweis / Coverage

Beschreibt Datenabdeckung oder fehlende Erhebung, ohne Infrastruktur zu erfinden.

Beispiel:

`RBAC-Detaildaten sind im aktuellen Actual-State-Input nicht enthalten.`

Freie KI-Inferenz darf nicht stillschweigend als Fakt erscheinen.

---

## 7. Diagramme im Dokument

Standardmäßig vorgesehene Sichten:

1. Azure-/Infrastruktur-Gesamtübersicht,
2. Netzwerk & Connectivity,
3. Workload & Deployment,
4. Backup & Recovery,
5. Security & Operations.

Diese Views können abhängig vom Vertrag aus unterschiedlichen Perspektiven erzeugt werden:

```text
Actual State          -> Collector
Desired Template      -> Bicep
Desired Deployment    -> Bicep + sanitizter Deploymentkontext
Reconciled            -> späterer expliziter Actual/Desired-Abgleich
```

Jedes Diagramm muss seine Perspektive eindeutig ausweisen. Ein Desired-State-Diagramm darf nicht als Kunden-Iststand beschriftet sein.

Die genaue Sichtauswahl kann abhängig von den tatsächlich vorhandenen Source-Domänen erfolgen.

Ein leeres oder nicht erhobenes Fachgebiet darf nicht mit typischen Standardkomponenten aufgefüllt werden.

Details: `DIAGRAM_ENGINE_STANDARD.md`, `IAC_BICEP_INTERFACE.md` und `architecture/CANONICAL_MODEL.md`.

---

## 8. Verhalten bei unvollständigen Daten

Erlaubt:

- Abschnitt auslassen,
- Abschnitt als `nicht erhoben` kennzeichnen,
- Coverage-Hinweis,
- unbekannte Einzelwerte als nicht verfügbar behandeln,
- nicht aufgelöste Bicep-Conditions als `unresolved` markieren.

Nicht erlaubt:

- Defaultwerte erfinden,
- nicht vorhandene Dienste ergänzen,
- Rollen allein aus Namen als gesichert behaupten,
- fehlende Beziehungen durch typische Azure-Architekturmuster ersetzen,
- nicht aufgelöste Bicep-Conditions als aktiviert annehmen.

---

## 9. Outputstrategie

Bereits beschlossen:

- Markdown-first,
- Actual-/Desired-Perspektive muss im Output nachvollziehbar bleiben.

Später vorgesehen:

- DOCX,
- PDF,
- ggf. weitere Publishing-Ziele nach separater Architekturentscheidung.

Die fachliche Dokumentstruktur soll unabhängig vom finalen Ausgabeformat bleiben. Dafür ist langfristig ein internes Document View Model vorgesehen; dessen konkrete technische Ausgestaltung ist noch offen.
