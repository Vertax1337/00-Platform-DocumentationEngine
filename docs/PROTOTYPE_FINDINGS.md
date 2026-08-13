# DocumentationEngine – Erkenntnisse aus den bisherigen Prototypen

Stand: 2026-08-13

Dieses Dokument hält die konkreten Erkenntnisse aus den ersten Dokumentations- und Diagrammprototypen fest, die auf real validierten Daten des `AzureInfrastructureCollector` aufgebaut wurden.

Es dokumentiert ausdrücklich auch Fehler, damit sie nicht später erneut als Architekturannahmen in die Engine einfließen.

---

## 1. Ausgangslage

Mit dem aktuellen AzureInfrastructureCollector standen bereits normalisierte Daten für folgende Domänen zur Verfügung:

- Core / Resource Groups / Ressourcen,
- Netzwerk,
- Compute,
- Azure Virtual Desktop,
- Storage,
- Backup,
- Key Vault.

Damit wurde testweise eine erste Kundendokumentation sowie mehrere Diagrammvarianten erzeugt.

Der Test zeigte: Die Datenbasis reicht bereits für belastbare technische Dokumentation, aber eine direkte Darstellung des normalisierten Inventars ist für Techniker zu maschinennah.

---

## 2. Prototyp 1 – maschinennahe Dokumentation

### Positiv

- technische Fakten konnten reproduzierbar aus Collector-Daten übernommen werden,
- Resource-ID-basierte Relationships ermöglichten belastbare Verknüpfungen,
- Netzwerk-/Compute-/AVD-/Backup-Zusammenhänge konnten ohne freie Namensheuristik dargestellt werden,
- Detailinventar und Tabellen waren bereits gut automatisierbar.

### Negativ

Die Dokumentation war zu stark geprägt von:

- Ressourcenlisten,
- technischen Feldern,
- JSON-/Inventarstruktur,
- maschinennaher Vollständigkeit.

Für einen Techniker fehlten zuerst:

- Orientierung,
- betriebliche Gruppierung,
- Service-/Workload-Sicht,
- High-Level-Architektur,
- schnelle Antwort auf Abhängigkeits- und Störungsfragen.

### Konsequenz

Das vollständige Inventar bleibt Bestandteil der Dokumentation, wird aber in Detailkapitel bzw. den Anhang verschoben.

Die Einstiegskapitel werden technikerorientiert aufgebaut.

---

## 3. Prototyp 2 – Technikerstruktur und High-Level-Übersicht

### Positiv

Die Umstellung auf:

- Kurzüberblick,
- Architekturübersicht,
- Netzwerk,
- Workloads,
- Backup,
- Security/Operations,
- Detailanhang

war deutlich näher an der gewünschten Betriebsdokumentation.

Die Verwendung offizieller Azure Architecture Icons ist für die Zielqualität erforderlich.

### Erkenntnis

Die DocumentationEngine benötigt eine eigene semantische Zwischenschicht.

Nicht ausreichend:

```text
Collector JSON -> Diagramm
```

Ziel:

```text
Collector JSON
 -> Relationship Graph
 -> Semantic View Builder
 -> Diagram View Model
 -> Renderer
```

---

## 4. Prototyp 3 – fünf Standard-Views

Es wurden folgende fünf Sichten prototypisch bewertet:

1. Azure-Gesamtübersicht,
2. Netzwerk & Connectivity,
3. Workload & Deployment,
4. Backup & Recovery,
5. Security & Operations.

### Bewertung

#### Gesamtübersicht

Grundsätzlicher Schnitt und Informationsniveau wurden als passend bewertet.

Verbesserungsbedarf:

- stabileres Raster,
- bessere Ausrichtung,
- ausschließlich real vorhandene Services.

#### Netzwerk & Connectivity

Grundsätzlich geeignete eigene Fachsicht.

Wichtig:

- keine typische Hub-/Landing-Zone-Komponente ergänzen, die beim Kunden nicht vorhanden ist,
- tatsächliche VNets, Subnets, Peerings, Gateways, NSGs und Routingobjekte aus dem Input verwenden.

#### Workload & Deployment

Grundsätzlich geeignet.

Die Gruppierung soll stärker nach betrieblicher Funktion bzw. Anwendung erfolgen und nicht primär nach Ressourcentyp.

#### Backup & Recovery

Die erste Darstellung war zu stark aus Sicht von Vaults und Policies aufgebaut.

Für Techniker ist die bessere Einstiegsfrage:

> Welche Systeme sind geschützt und wodurch?

Daher künftig:

```text
Protected Resource
 -> Protection/Status
 -> Policy
 -> Vault
 -> Recovery-Metadaten
```

#### Security & Operations

Die erste Darstellung war fachlich noch zu schwach, weil relevante Detaildomänen zum Zeitpunkt des Prototyps noch nicht vollständig durch Collector-Daten abgedeckt waren.

Konsequenz:

- keine hypothetische Security-/Operations-Topologie erzeugen,
- bis zur ausreichenden Datenabdeckung lieber Coverage anzeigen,
- vollständige Sicht erst aus entsprechenden Security-/Governance-/Monitoring-/Automation-Inputs erzeugen.

---

## 5. Kritischer Prototypfehler – erfundene Azure-Dienste

In einer visuell erzeugten Referenzdarstellung erschienen unter anderem typische Azure-Komponenten wie Firewall/Bastion, obwohl diese im tatsächlichen Kunden-Iststand nicht vorhanden waren.

### Bewertung

Für eine technische Kundendokumentation ist das ein **kritischer Fehler**.

Die Darstellung war optisch plausibel, aber sachlich falsch.

### Verbindliche Konsequenz

Generative Bildmodelle dürfen nicht den finalen technischen Diagramminhalt bestimmen.

Sie können für Stil-/Mockup-Arbeit eingesetzt werden, aber das produktive Diagramm muss deterministisch aus dem validierten Input erzeugt werden.

Microsoft-Referenzarchitekturen dürfen nur als Kommunikations-/Layoutreferenz genutzt werden, niemals als fehlende Kundenarchitektur.

---

## 6. Layout-Erkenntnisse

Freies Graph-Autolayout ist für High-Level-Technikerdokumentation nicht ausreichend.

Probleme:

- unruhige Positionierung,
- ungleichmäßige Abstände,
- fachlich schwache Gruppierung,
- zu viele Kantenkreuzungen,
- Elemente werden optisch nach Platz statt nach Bedeutung angeordnet.

### Ziel

Semantische Zonen mit kontrolliertem Raster.

Beispiel:

```text
Tenant / Subscription

Identity     Connectivity     Workloads

Shared Services / Protection / Operations

On-Prem / Extern außerhalb der Azure-Grenze
```

Die Position soll Bedeutung transportieren.

---

## 7. Microsoft-Referenzen als Designinput

Aus Microsofts aktuellen Leitlinien wurden folgende Punkte als relevant bestätigt:

- offizielle Azure-Icons und Servicenamen,
- Icons nicht verzerren/rotieren/umfärben,
- Legenden bei Linien-/Rahmensemantik,
- progressive Offenlegung statt eines überladenen Gesamtbilds,
- Diagrammquellen versionieren,
- High-Level- und Detaildiagramme bewusst trennen.

Relevante Referenzen:

- https://learn.microsoft.com/en-us/azure/architecture/icons/
- https://learn.microsoft.com/en-us/azure/well-architected/architect-role/design-diagrams
- https://learn.microsoft.com/en-us/azure/developer/azure-skills/skills/azure-resource-visualizer
- https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/

---

## 8. Was aus dem Prototyp ausdrücklich NICHT beschlossen ist

Die Prototypen entscheiden noch nicht:

- konkrete Renderbibliothek,
- Mermaid vs. Graphviz vs. PlantUML vs. eigener SVG-Renderer,
- konkrete Programmiersprache der Engine,
- finales internes Diagrammschema,
- finale Farbpalette,
- exakte Pixel-/Gridmaße,
- DOCX-/PDF-Technologie,
- Publishing-Ziel.

Diese Punkte bleiben im kanonischen Umsetzungsplan offen, bis sie bewusst bewertet werden.

---

## 9. Aktueller sinnvoller Qualitätsmaßstab

Ein Diagramm gilt nicht deshalb als gut, weil es visuell professionell wirkt.

Es muss mindestens:

1. faktisch korrekt sein,
2. nur belegte Ressourcen enthalten,
3. nur belegte Beziehungen enthalten,
4. die jeweilige Technikerfrage beantworten,
5. die passende Detailtiefe besitzen,
6. konsistent und reproduzierbar gerendert werden,
7. fehlende Daten sichtbar behandeln,
8. eine deterministische, testbare Diagrammquelle besitzen.
