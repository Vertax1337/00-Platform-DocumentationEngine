# DocumentationEngine – Diagram Engine Standard

Stand: 2026-09-01
Status: **Konsolidierter Prototypstandard; Actual-/Desired-Perspektiven ergänzt; Renderer- und Layouttechnologie weiterhin offen**

Dieses Dokument hält die fachlichen Regeln für die spätere Diagram Engine fest.

---

## 1. Ziel

Die Diagram Engine erzeugt aus validierten technischen Fakten technikergeeignete Diagramme.

Sie ist **kein Infrastruktur-Collector**, kein IaC-Deploymentwerkzeug und darf fehlende Ressourcen oder Beziehungen nicht durch typische Referenzarchitekturen ergänzen.

```text
Collector / Bicep / weitere Quellen
      |
      v
Source / Provider Adapter
      |
      v
Canonical Infrastructure Model
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

## 2. Oberste Faktentreue-Regeln

### DE-DIAG-001 – Nur belegte Ressourcen

Ein Infrastruktur- oder Serviceelement darf nur dargestellt werden, wenn es durch einen freigegebenen Input belegt ist.

```text
Resource / Relationship im freigegebenen Input belegt
        -> darf in der passenden Perspektive dargestellt werden

nicht belegt
        -> wird nicht dargestellt
```

Eine Microsoft-Referenzarchitektur darf nur Layout-/Kommunikationsreferenz sein.

### DE-DIAG-002 – Keine Namensheuristik als Faktenersatz

Technische Beziehungen werden nach Möglichkeit über stabile IDs bzw. explizite Relationships dargestellt.

Namen dürfen nicht verwendet werden, um externe Beziehungen zu erfinden.

### DE-DIAG-003 – Keine generative Bildausgabe als technische Source of Truth

Generative Bildmodelle dürfen für Stilfindung, visuelle Inspiration, Mockups und Layoutideen verwendet werden.

Sie dürfen nicht den finalen technischen Diagramminhalt erzeugen.

Produktive Diagramme werden deterministisch aus einem validierten Diagram View Model gerendert.

### DE-DIAG-004 – Perspektive ist Pflicht

Jedes technische Diagramm muss eindeutig ausweisen, welche Perspektive dargestellt wird.

Initiale Perspektiven:

```text
actual
desiredTemplate
desiredDeployment
reconciled   # erst nach explizitem Reconciliation-Contract
```

Regeln:

- `actual` basiert auf freigegebenem Iststandsinput, z. B. Collector-Snapshot.
- `desiredTemplate` basiert auf versioniertem IaC-Template und kann konditionale/unaufgelöste Ressourcen enthalten.
- `desiredDeployment` basiert auf IaC plus freigegebenem, sanitiztem Deploymentkontext.
- `reconciled` darf erst nach expliziter Actual-/Desired-Korrelation erzeugt werden.
- Desired State darf nicht als gemessener Kunden-Iststand beschriftet werden.
- Actual und Desired dürfen nicht durch Name-only-Matching in einem Diagramm zusammengeführt werden.

---

## 3. Microsoft-orientierte Darstellungsregeln

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

```text
Context / Gesamtübersicht
        -> Domänensicht
        -> Workload-/Komponentensicht
        -> Detailinventar
```

High-Level- und Detaildiagramme werden bewusst getrennt.

### 3.3 Legende und visuelle Semantik

Jedes Diagramm mit unterschiedlichen Linien-, Rahmen- oder Containersemantiken enthält eine kompakte Legende.

Die Bedeutung darf nicht ausschließlich über Farbe vermittelt werden.

Bei Desired-/Reconciled-Sichten muss zusätzlich die Perspektive visuell/textuell eindeutig erkennbar sein.

### 3.4 Diagrammquellen unter Versionskontrolle

Das Diagram View Model bzw. die deterministische Diagrammquelle wird versionierbar abgelegt bzw. reproduzierbar aus versionierten Inputs erzeugt.

PNG/PDF sind Ausgabeformate, nicht die einzige Source of Truth.

Für Bicep-basierte Views muss die Source-Provenance mindestens bis zum IaC-Commit/Build zurückführbar sein.

---

## 4. Referenzen

### 4.1 Microsoft Resource Visualizer

`azure-resource-visualizer` dient als fachliche Referenz für Discovery, Relationship Mapping, Diagram Generation und Resource Documentation Creation, nicht als festgelegte technische Dependency.

Microsoft-Referenz:
https://learn.microsoft.com/en-us/azure/developer/azure-skills/skills/azure-resource-visualizer

Die DocumentationEngine unterscheidet sich dadurch, dass sie freigegebene Collector- bzw. IaC-Artefakte konsumiert und nicht ungeplant selbst live inventarisiert.

### 4.2 Azure Landing Zone Diagramme

Relevante Kommunikationsmuster:

- semantische Bereiche statt freiem Ressourcengraph,
- Plattform-/Workload-Trennung,
- Connectivity als eigene Zone,
- externe/on-premises Systeme außerhalb der Azure-Grenze,
- Hierarchie und Position transportieren Bedeutung,
- High-Level-Sicht vor Detailansicht.

Microsoft-Referenz:
https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/

Für Kundenbilder gilt dennoch: Nur tatsächlich belegte Komponenten der jeweiligen Perspektive werden dargestellt.

---

## 5. Semantische Zonen statt freiem Graph-Autolayout

Freies Graph-Autolayout reicht für High-Level-Technikerdokumentation nicht aus.

Ziel ist ein kontrolliertes Layout mit semantischen Zonen.

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

Layoutgrundsätze:

- feste Raster und Alignment-Regeln,
- konsistente Abstände,
- stabile Containerpositionen,
- gleiche Ressourcenklassen mit gleicher Darstellungslogik,
- Kanten möglichst ohne unnötige Kreuzungen,
- Detailtiefe abhängig von der View,
- keine künstliche Flächenfüllung,
- gleiche Inputs und gleiche Regeln erzeugen reproduzierbare Positionierung soweit der gewählte Renderer dies technisch erlaubt.

Die konkrete Layoutbibliothek bzw. Rendertechnologie ist weiterhin **OFFEN**.

---

## 6. Fünf Standard-Views

### View 01 – Azure-/Infrastruktur-Gesamtübersicht

Technikerfragen:

- Tenant/Subscription-Scope,
- zentrale Connectivity,
- wesentliche Workloads,
- Plattform-/Shared Services,
- externe/on-premises Anbindung,
- grobe Protection-/Betriebsbeziehungen.

Nicht vorgesehen: vollständige Ressourcenliste, jede NIC/Disk/Policy, vollständige Properties.

Diese View kann sowohl als `actual` als auch als eindeutig gekennzeichnete Bicep-`desired*`-Architektursicht erzeugt werden.

### View 02 – Netzwerk & Connectivity

Primäre Fragen:

- Welche VNets/Subnets existieren bzw. sind vorgesehen?
- Welche Peerings/Gateways/Connections sind relevant?
- Welche NSGs/Routen wirken auf Segmente?
- Wie ist On-Premises angebunden?

Actual und Desired werden nicht ungekennzeichnet vermischt.

### View 03 – Workload & Deployment

Gruppierung primär nach Betriebsfunktion/Workload statt Ressourcentyp.

Für Bicep eignet sich diese View ausdrücklich zur Darstellung der versionierten Deploymentarchitektur.

### View 04 – Backup & Recovery

Technikersicht startet bei der geschützten Ressource:

```text
Geschützte Ressource
  -> Protection / Backup Status
  -> verwendete Policy
  -> zuständiger Vault
  -> Recovery-Metadaten
```

Wichtig: Bicep kann geplante Protection-Ressourcen darstellen. Tatsächlicher Schutzstatus und Recovery Points sind Actual-State-Fakten und dürfen nicht aus IaC erfunden werden.

### View 05 – Security & Operations

Wird nur mit tatsächlich fachlich verfügbaren Daten vollständig gerendert.

Bis ausreichende Inputs vorhanden sind, wird Coverage statt hypothetischer Topologie gezeigt.

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

---

## 7. Kanten- und Beziehungssemantik

Mindestens folgende Kategorien werden fachlich unterschieden:

- technische Topologie / Resource Relationship,
- Netzwerk-/Datenpfad,
- Management-/Konfigurationsbeziehung,
- Protection/Backup,
- externe Verbindung.

Jede View enthält nur Kanten, die für ihre Fragestellung relevant sind.

Desired-State-Kanten müssen aus Bicep/ARM-Strukturen maschinenauflösbar belegbar sein; reine Namensähnlichkeit ist unzulässig.

---

## 8. Fehlende und unaufgelöste Daten

Erlaubte Darstellungen:

- Bereich weglassen,
- `nicht erhoben` / `nicht verfügbar`,
- Coverage-Status,
- Bicep-Condition `unresolved`, wenn sie ohne freigegebenen Deploymentkontext nicht sicher auflösbar ist.

Nicht erlaubt:

- typische Standardressource ergänzen,
- aus Namensmustern externe Beziehungen erfinden,
- fehlendes Feld als bekannten Standardwert darstellen,
- `unresolved` als `enabled` ausgeben.

---

## 9. Desired/Actual Reconciliation

Die Diagram Engine plant eine spätere Reconciliation-View ein, erzeugt sie aber erst auf Basis eines expliziten Reconciliation-Contracts.

Zielzustände können beispielsweise sein:

```text
matched
desiredMissing
actualUnmanaged
unresolved
propertyDrift   # nur bei belastbarer Vergleichssemantik
```

Der Renderer entscheidet keine Korrelation selbst. Er erhält ein bereits validiertes Reconciliation-/Diagram View Model.

---

## 10. Offene technische Entscheidungen

Weiterhin **OFFEN**:

- konkretes Diagram View Model,
- Layout-Engine,
- SVG-Renderer,
- Mermaid/Graphviz/ELK/PlantUML/eigene SVG-Pipeline,
- automatische Text-/Labelplatzierung,
- Kantenrouting,
- Icon-Katalog-Updateverfahren,
- herstellerübergreifende Icon-Standards,
- Diagramm-Snapshot-/Golden-Master-Testverfahren,
- Barrierefreiheitsprüfung,
- Export nach DOCX/PDF.

Bicep/IaC als Desired-State-Quelle ist dagegen **nicht mehr offen**; offen ist nur die konkrete technische Adapter-/Parserimplementierung.
