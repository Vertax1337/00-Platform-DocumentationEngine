# DocumentationEngine – Diagram Engine Standard

Stand: 2026-08-13
Status: **Konsolidierter Prototypstandard; Renderer- und Layouttechnologie weiterhin offen**

Dieses Dokument hält die aus Microsoft-Referenzen und den bisherigen realen Collector-Prototypen gewonnenen Regeln für die spätere Diagram Engine fest.

---

## 1. Ziel

Die Diagram Engine erzeugt aus normalisierten, validierten Infrastruktur-Fakten technikergeeignete Diagramme.

Sie ist **kein Infrastruktur-Collector** und darf fehlende Ressourcen oder Beziehungen nicht durch typische Referenzarchitekturen ergänzen.

```text
Collector JSON
      |
      v
Relationship Graph
      |
      v
Semantic View Builder
      |
      v
Diagram View Model
      |
      v
Layout Engine
      |
      v
Icon Renderer
      |
      v
SVG / PNG / Markdown / später DOCX/PDF
```

Die Datenbasis und die visuelle Darstellung sind bewusst getrennt.

---

## 2. Oberste Faktentreue-Regel

### DE-DIAG-001 – Nur belegte Ressourcen

Ein Infrastruktur- oder Serviceelement darf nur dargestellt werden, wenn es durch einen freigegebenen Input belegt ist.

```text
Resource / Relationship im Collector belegt
        -> darf dargestellt werden

nicht belegt
        -> wird nicht dargestellt
```

Eine Microsoft-Referenzarchitektur darf **nur** als Layout-/Kommunikationsreferenz dienen.

Sie darf niemals verwendet werden, um im Kunden-Iststand fehlende Komponenten wie beispielsweise Azure Firewall, Bastion, DDoS, ExpressRoute oder andere typische Plattformdienste zu ergänzen.

### DE-DIAG-002 – Keine Namensheuristik als Faktenersatz

Technische Beziehungen werden nach Möglichkeit über stabile IDs bzw. explizite Relationships dargestellt.

Namen dürfen nicht verwendet werden, um eine externe Beziehung zu erfinden, die im Input nicht belegt ist.

### DE-DIAG-003 – Keine generative Bildausgabe als technische Source of Truth

Generative Bildmodelle dürfen für:

- Stilfindung,
- visuelle Inspiration,
- Mockups,
- Layoutideen

verwendet werden.

Sie dürfen **nicht** den finalen technischen Diagramminhalt erzeugen.

Grund: Im Prototyp wurde eine optisch plausible Azure-Übersicht erzeugt, die jedoch nicht vorhandene Dienste wie Azure Firewall/Bastion enthielt. Ein solches Verhalten ist für technische Kundendokumentation nicht akzeptabel.

Produktive Diagramme müssen daher deterministisch aus dem Diagram View Model gerendert werden.

---

## 3. Microsoft-orientierte Darstellungsregeln

Microsofts aktuelle Architekturleitlinien werden als Kommunikationsstandard übernommen, soweit sie mit dem kundenspezifischen Faktenmodell vereinbar sind.

### 3.1 Offizielle Icons

Für Azure-Komponenten werden offizielle Microsoft Azure Architecture Icons verwendet.

Regeln:

- Icon nicht verzerren,
- nicht spiegeln,
- nicht drehen,
- nicht willkürlich umfärben,
- offiziellen Servicenamen in unmittelbarer Nähe darstellen,
- Icon-Katalog versionieren.

Microsoft-Referenz:
https://learn.microsoft.com/en-us/azure/architecture/icons/

### 3.2 Progressive Disclosure

Nicht alle Details gehören in ein Diagramm.

Die Diagrammdokumentation folgt dem Prinzip:

```text
Context / Gesamtübersicht
        -> Domänensicht
        -> Workload-/Komponentensicht
        -> Detailinventar
```

Damit wird die von Microsoft im Well-Architected Framework beschriebene Regel **"Layer, don't overload"** auf die DocumentationEngine übertragen.

Microsoft-Referenz:
https://learn.microsoft.com/en-us/azure/well-architected/architect-role/design-diagrams

### 3.3 Legende und visuelle Semantik

Jedes Diagramm mit unterschiedlichen Linien-, Rahmen- oder Containersemantiken enthält eine kompakte Legende.

Die Bedeutung darf nicht ausschließlich über Farbe vermittelt werden.

### 3.4 Diagrammquellen unter Versionskontrolle

Das Diagram View Model bzw. die deterministische Diagrammquelle wird versionierbar abgelegt bzw. reproduzierbar aus versionierten Inputs erzeugt.

PNG/PDF sind Ausgabeformate, nicht die einzige Source of Truth.

---

## 4. Erkenntnisse aus Microsoft Resource Visualizer

Microsofts `azure-resource-visualizer` dient als fachliche Referenz für den Analyseablauf, nicht als festgelegte technische Dependency.

Relevante Prinzipien:

- Resource Discovery,
- Deep Resource Analysis,
- Relationship Mapping,
- Diagram Generation,
- Resource Documentation Creation.

Der Microsoft Skill erzeugt Mermaid-Diagramme aus Azure-Ressourcengruppen. Für dieses Projekt bleibt Mermaid jedoch nur eine mögliche Rendertechnologie; es besteht weiterhin keine globale Festlegung darauf.

Microsoft-Referenz:
https://learn.microsoft.com/en-us/azure/developer/azure-skills/skills/azure-resource-visualizer

Die DocumentationEngine unterscheidet sich bewusst dadurch, dass sie nicht erneut live Azure abfragt, sondern freigegebene Collector-Artefakte konsumiert.

---

## 5. Erkenntnisse aus Azure Landing Zone Diagrammen

Azure-Landing-Zone-Referenzdiagramme zeigen ein für die DocumentationEngine relevantes Kommunikationsmuster:

- semantische Bereiche statt freiem Ressourcengraph,
- klare Plattform-/Workload-Trennung,
- Connectivity als eigene Zone,
- externe/on-premises Systeme außerhalb der Azure-Grenze,
- Hierarchie und Position transportieren Bedeutung,
- High-Level-Sicht vor Detailansicht.

Microsoft stellt diese Referenzarchitekturen als anpassbare Ausgangspunkte und unter anderem als Visio/PDF bereit.

Microsoft-Referenz:
https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/

Für Kundenbilder gilt dennoch: Nur tatsächlich vorhandene Komponenten werden dargestellt.

---

## 6. Semantische Zonen statt freiem Graph-Autolayout

Der erste Prototyp hat gezeigt, dass reines automatisches Graph-Verteilen für Technikerdiagramme nicht ausreicht.

Das Ziel ist ein kontrolliertes Layout mit semantischen Zonen.

Beispiel:

```text
+-------------------------------------------------------------+
| Tenant / Subscription / Scope                               |
+-------------------+----------------------+------------------+
| Identity /        | Connectivity         | Workloads        |
| Management        |                      |                  |
+-------------------+----------------------+------------------+
| Shared Services / Protection / Operations                   |
+-------------------------------------------------------------+

On-Prem / Extern liegt außerhalb der Azure-/Subscription-Zone.
```

Die Position eines Elements trägt damit bewusst Information.

### Layoutgrundsätze

- feste Raster und Alignment-Regeln,
- konsistente Abstände,
- stabile Containerpositionen,
- gleiche Ressourcenklassen mit gleicher Darstellungslogik,
- Kanten möglichst ohne unnötige Kreuzungen,
- Detailtiefe abhängig von der jeweiligen View,
- keine künstliche Flächenfüllung durch nicht vorhandene Dienste.

Die konkrete Layoutbibliothek bzw. Rendertechnologie ist weiterhin **OFFEN**.

---

## 7. Fünf Standard-Views

Die bisherigen Prototypen bestätigen fünf sinnvolle Informationssichten als Arbeitsstandard.

### View 01 – Azure-Gesamtübersicht

Zweck:

Ein Techniker soll innerhalb weniger Sekunden erkennen:

- Tenant/Subscription-Scope,
- zentrale Connectivity,
- wesentliche Workloads,
- zentrale Plattform-/Shared Services,
- externe/on-premises Anbindung,
- grobe Schutz-/Betriebsbeziehungen.

Nicht vorgesehen:

- vollständige Ressourcenliste,
- jede NIC/Disk/Policy,
- Detailkonfigurationen.

### View 02 – Netzwerk & Connectivity

Primäre Fragen:

- Wie sind Netze miteinander verbunden?
- Welche VNets/Subnets existieren?
- Welche Peerings/Gateways/Connections sind relevant?
- Welche NSGs/Routen wirken auf welche Segmente?
- Wie ist On-Premises angebunden?

Nur vorhandene Netzwerkkomponenten werden dargestellt.

### View 03 – Workload & Deployment

Die Gruppierung erfolgt primär nach **Betriebsfunktion/Workload**, nicht nach Azure-Ressourcentyp.

Beispiele:

- AVD als betriebliche Domäne,
- Sage/ERP als betriebliche Domäne,
- Domain Services als betriebliche Domäne.

Detailressourcen wie NICs und Disks werden nur eingeblendet, wenn sie für die jeweilige Sicht relevant sind.

### View 04 – Backup & Recovery

Der erste Prototyp war zu Vault-/Policy-zentriert und damit zu maschinennah.

Die Technikersicht startet künftig bei der geschützten Ressource:

```text
Geschützte Ressource
  -> Protection / Backup Status
  -> verwendete Policy
  -> zuständiger Vault
  -> letzter verfügbarer Recovery Point / relevante Recovery-Metadaten
```

Erst sekundär wird die gemeinsame Vault-/Policy-Struktur dargestellt.

Primäre Technikerfrage:

> Was ist geschützt und wodurch?

### View 05 – Security & Operations

Die Sicht wird nur mit tatsächlich fachlich erhobenen Security-/Governance-/Monitoring-/Automation-Daten vollständig gerendert.

Bis entsprechende Collector-Domänen vorhanden sind, darf die Engine eine **Coverage-/Verfügbarkeitsansicht** darstellen, statt eine hypothetische Topologie zu erfinden.

Beispiel:

```text
Security & Operations

[x] Key Vault / Protection        erhoben
[ ] RBAC                          noch nicht erhoben
[ ] Policies / Locks              noch nicht erhoben
[ ] Diagnostic Settings           noch nicht erhoben
[ ] Alerting                      noch nicht erhoben
[ ] Automation Details            noch nicht erhoben
```

Später vorgesehene Bereiche:

```text
Security & Governance
|- RBAC
|- Resource Locks
|- Policy Assignments
|- Key Vault
`- Defender / Security Configuration

Operations
|- Log Analytics
|- Diagnostic Settings
|- Alerts
|- Action Groups
|- Automation Accounts
|- Runbooks
`- Schedules
```

---

## 8. Kanten- und Beziehungssemantik – Arbeitsstandard

Die exakten Stile werden mit dem Renderer finalisiert. Fachlich sollen mindestens folgende Kategorien unterschieden werden:

- **technische Topologie / Resource Relationship** – explizit belegte Ressourcenbeziehung,
- **Netzwerk-/Datenpfad** – belegte Konnektivität,
- **Management-/Konfigurationsbeziehung** – z. B. Zuordnung/Verwendung,
- **Protection/Backup** – Schutzbeziehung,
- **externe Verbindung** – Azure zu On-Prem/extern.

Jede View enthält nur Kanten, die für ihre Fragestellung relevant sind.

---

## 9. Fehlende Daten

Fehlende oder noch nicht erhobene Daten werden nicht implizit ergänzt.

Erlaubte Darstellungen:

- Bereich weglassen, wenn er für die View nicht erforderlich ist,
- explizit `nicht erhoben` / `nicht verfügbar` kennzeichnen,
- Coverage-Status anzeigen.

Nicht erlaubt:

- typische Standardressource ergänzen,
- aus Namensmustern externe Beziehungen erfinden,
- ein fehlendes Feld als bekannten Standardwert darstellen.

---

## 10. Offene technische Entscheidungen

Weiterhin **OFFEN**:

- konkretes Diagram View Model,
- Layout-Engine,
- SVG-Renderer,
- Mermaid/Graphviz/PlantUML oder eigene SVG-Pipeline,
- automatische Text-/Labelplatzierung,
- Kantenrouting,
- Icon-Katalog-Updateverfahren,
- herstellerübergreifende Icon-Standards,
- Diagramm-Snapshot-/Golden-Master-Testverfahren,
- Barrierefreiheitsprüfung,
- Export nach DOCX/PDF.

Diese Punkte werden nicht durch die Prototypen stillschweigend entschieden.
