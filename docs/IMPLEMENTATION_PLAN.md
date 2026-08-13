# DocumentationEngine – kanonischer Umsetzungsplan

Stand: 2026-08-13

Dieses Dokument ist der kanonische technische Umsetzungsplan für `00-Platform / DocumentationEngine`.

Neue Architekturentscheidungen, technische Standards, offene Punkte, Implementierungsschritte und erreichte Umsetzungsstände werden hier fortlaufend gepflegt.

Bereits im Gesamtprojekt getroffene Entscheidungen werden übernommen und nicht ohne ausdrückliche Neubewertung erneut geöffnet.

---

## 1. Statusmodell

Für alle Themen dieses Unterprojekts werden folgende Zustände verwendet:

- **BESCHLOSSEN** – Architektur-/Technikentscheidung ist getroffen.
- **IMPLEMENTIERT** – Entscheidung ist technisch umgesetzt und im Repository vorhanden.
- **OFFEN** – Entscheidung oder konkrete Umsetzung ist noch nicht festgelegt.
- **SPÄTER** – bekannte mögliche Erweiterung, nicht Bestandteil der ersten produktiven Version.

---

## 2. Ziel des Unterprojekts

Die `DocumentationEngine` verarbeitet normalisierte und validierte Daten zentraler Collector und erzeugt daraus standardisierte technische Kundendokumentationen.

Zielartefakte umfassen insbesondere:

- strukturierte technische Texte,
- Tabellen,
- Infrastrukturübersichten,
- Netzwerkdiagramme,
- Architekturdiagramme.

Initiales Ausgabeformat ist Markdown.

---

## 3. Bereits beschlossene Architekturgrenzen

### DE-DEC-001 – Verbindliche organisatorische Zuordnung

**Status:** BESCHLOSSEN

Die organisatorische Zielzuordnung ist bereits im zentralen [`DEV_Ops_Bootstrap`-Umsetzungsplan](https://github.com/Vertax1337/DEV_Ops_Bootstrap/blob/main/docs/Umsetzungsplan.md) beschlossen und wird in diesem Unterprojekt nicht erneut als offene Architekturentscheidung geführt.

```text
00-Platform
├── PlatformBootstrap
├── PipelineTemplates
├── DocumentationEngine
├── SecurityValidation
└── SharedModules

10-Automation
├── AzureInfrastructureCollector
└── OPNsenseDocumentation
```

Die `DocumentationEngine` ist zentrale Plattformlogik im Azure-DevOps-Projekt `00-Platform`. Sie verarbeitet normalisierte Collector-Daten providerübergreifend und erzeugt daraus die Kundendokumentation.

`AzureInfrastructureCollector` und `OPNsenseDocumentation` sind quellspezifische Automationskomponenten im Azure-DevOps-Projekt `10-Automation`. Sie erfassen und normalisieren ihre jeweiligen Quelldaten, enthalten aber keine eigene vollständige Kundendokumentationsengine.

Die Bezeichnung des aktuellen GitHub-Arbeitsrepositories `Vertax1337/10-DocumentationEngine` ändert diese beschlossene Azure-DevOps-Zuordnung nicht und öffnet sie nicht erneut.

Die `DocumentationEngine` wird nicht pro Kunde und nicht pro Collector dupliziert.

### DE-DEC-002 – Repository-Provisionierung

**Status:** BESCHLOSSEN

Die Provisionierung des Repositories erfolgt über den bestehenden DEVOPS-/Platform-Bootstrap.

Die Bootstrap-Architektur wird in diesem Unterprojekt nicht neu entworfen.

### DE-DEC-003 – Trennung Collector und Dokumentationslogik

**Status:** BESCHLOSSEN

Collector übernehmen herstellerspezifische Erfassung, Parsing und Normalisierung.

Die `DocumentationEngine` verarbeitet die normalisierten Daten anschließend herstellerübergreifend in standardisierte Dokumentationsartefakte.

Collector sollen keine jeweils eigene vollständige Kundendokumentationsengine enthalten.

Umgekehrt soll die `DocumentationEngine` die jeweilige Infrastruktur nicht erneut live inventarisieren. Für Azure konsumiert sie die freigegebenen, normalisierten Artefakte des `AzureInfrastructureCollector` und implementiert keinen parallelen Azure-Collector.

### DE-DEC-004 – Validierte Eingaben / Fail Closed

**Status:** BESCHLOSSEN

Die Engine ist Teil eines Datenflusses mit vorgelagerten Schema-, Security- und Quality-Gates.

Fehlgeschlagene relevante Validierungen stoppen den Build nach dem Fail-Closed-Prinzip.

Die genaue Verantwortungsabgrenzung zwischen `SecurityValidation` und eigenen technischen Engine-Validierungen bleibt offen.

### DE-DEC-005 – Zentrale Pipeline-Integration

**Status:** BESCHLOSSEN

Die Einbindung erfolgt über zentrale `PipelineTemplates`.

Kundenprojekte sollen die Engine-Orchestrierung nicht jeweils selbst duplizieren.

### DE-DEC-006 – SharedModules

**Status:** BESCHLOSSEN

Wiederverwendbare technische Komponenten können aus `SharedModules` genutzt werden.

Welche konkreten Komponenten dies sind, wird erst bei der Implementierung entschieden.

### DE-DEC-007 – Markdown-first

**Status:** BESCHLOSSEN

Markdown ist das initiale Dokumentationsoutputformat.

PDF und DOCX sind spätere Ausbaustufen.

### DE-DEC-008 – Offizielle Azure Architecture Icons

**Status:** BESCHLOSSEN

Azure-Diagramme verwenden offizielle Microsoft Azure Architecture Icons.

Offizielle Hersteller-Icons werden nicht willkürlich verzerrt, gespiegelt, gedreht oder umgefärbt.

Ein vollständiger herstellerübergreifender Symbolstandard und die konkrete Renderer-Implementierung bleiben offen.

### DE-DEC-009 – Reproduzierbare Builds und automatisierte Tests

**Status:** BESCHLOSSEN

Die Engine muss reproduzierbare Pipeline-Builds und automatisierte Tests ermöglichen.

Konkrete Testarten und Quality Gates werden im Zuge der Implementierung festgelegt.

### DE-DEC-010 – Faktentreue vor Referenzarchitektur

**Status:** BESCHLOSSEN

Ein Infrastruktur- oder Serviceelement darf nur dann als Bestandteil des Kunden-Iststands dargestellt werden, wenn es durch einen freigegebenen Input belegt ist.

Microsoft- und Hersteller-Referenzarchitekturen dienen ausschließlich als Kommunikations-, Layout- und Stilreferenz. Sie dürfen nicht verwendet werden, um typische, aber beim Kunden nicht vorhandene Dienste zu ergänzen.

Fehlende Daten werden ausgelassen oder explizit als `nicht erhoben` / `nicht verfügbar` gekennzeichnet.

### DE-DEC-011 – Technikerorientierung vor Inventar

**Status:** BESCHLOSSEN

Primäre Zielgruppe der generierten Kundendokumentation sind Techniker und Administratoren.

Die Einstiegssicht priorisiert daher:

- Orientierung,
- Architektur,
- Workloads,
- betriebliche Zusammenhänge,
- Abhängigkeiten,
- Protection und Operations.

Vollständige Ressourceninventare und maschinennahe Detailtabellen werden nachgelagert bzw. im Anhang dargestellt.

### DE-DEC-012 – Semantische View-Schicht

**Status:** BESCHLOSSEN

Zwischen Collector-Daten und Renderer wird eine semantische Sichtebene vorgesehen.

Zielmodell:

```text
Collector Data
      -> Relationship Graph
      -> Semantic View Builder
      -> Document / Diagram View Model
      -> Renderer
      -> Output
```

Die konkrete technische Implementierung des internen Modells ist weiterhin offen. Beschlossen ist die Verantwortungs- und Abstraktionsschicht.

### DE-DEC-013 – Generative Bilder sind keine technische Diagramm-Source-of-Truth

**Status:** BESCHLOSSEN

Generative Bildmodelle dürfen für Stilfindung, Mockups und Layoutideen genutzt werden.

Sie dürfen nicht den finalen technischen Diagramminhalt bestimmen.

Produktive Kundendiagramme müssen deterministisch aus validierten Daten und einem testbaren Diagram View Model erzeugt werden.

Auslöser dieser Entscheidung war ein Prototyp, der optisch plausible, aber im Kunden-Iststand nicht vorhandene Azure-Dienste wie Firewall/Bastion ergänzte.

---

## 4. Externe Abhängigkeiten und Schnittstellen

### 4.1 DEVOPS / PlatformBootstrap

**Status:** BESCHLOSSENER RAHMEN / externe Implementierung

Erwartungen an die `DocumentationEngine`:

- keine eigene Kundenprovisionierung,
- Nutzung bereits provisionierter `CUST-*`-Strukturen,
- keine erneute Implementierung der Bootstrap-Logik.

Offen für spätere Integration:

- genaue Pfade/Repository-IDs,
- Übergabe kundenspezifischer Metadaten,
- technische Voraussetzungen für End-to-End-Testkunden.

### 4.2 PipelineTemplates

**Status:** BESCHLOSSENER RAHMEN / konkrete Schnittstelle OFFEN

Zu definieren:

- Template-Aufruf,
- Parameter,
- Input-Artefakte,
- Output-Artefakte,
- Working Directory,
- Exitcodes,
- Logging-/Fehlervertrag,
- Versionierung des Engine-Aufrufs.

### 4.3 SecurityValidation

**Status:** BESCHLOSSENER RAHMEN / Verantwortungsgrenze OFFEN

Zu definieren:

- welche Prüfungen zwingend vor der Engine stattfinden,
- welche Konsistenzprüfungen die Engine selbst ausführt,
- welche Fehlerarten Security-, Contract-, Data-Quality- oder Rendering-Fehler darstellen,
- welche Fehler jeweils den Build stoppen.

### 4.4 SharedModules

**Status:** BESCHLOSSENER RAHMEN / konkrete Nutzung OFFEN

Regel:

Gemeinsame Funktionalität wird nur dann ausgelagert, wenn tatsächliche Wiederverwendung erkennbar ist. Es wird zu Beginn keine künstliche Shared-Library-Struktur erzeugt.

### 4.5 AzureInfrastructureCollector

**Status:** LOGISCHE SCHNITTSTELLE BESCHLOSSEN / aktueller Output gesichtet / finaler technischer Contract OFFEN

Erwarteter Input:

- normalisierte und validierte Azure-Infrastrukturdaten,
- stabile technische IDs,
- explizite Relationships,
- schema-stabile Collections,
- Snapshot-/Manifest-/Validation-Metadaten.

Der aktuell real validierte Collector arbeitet modular und erzeugt unter anderem:

```text
Inventory/
|- resourceGroups.json
|- resources.json
|- network.json
|- compute.json
|- avd.json
|- storage.json
|- backup.json
`- keyVault.json

summary.json
manifest.json
readOnlyVerification.json
```

Diese Struktur ist als aktueller Iststand dokumentiert, aber noch nicht automatisch der finale Engine-Contract.

Noch offen:

- direkte Übernahme dieses Artefaktpakets vs. eigenes Exchange-Schema,
- Schema-Versionierung,
- Pflicht-/Optionalfelder,
- Provider-/Collector-Kennung,
- Umgang mit mehreren Collector-Artefakten,
- Kompatibilitätsregeln zwischen Collector- und Engine-Versionen.

Details: `COLLECTOR_INTERFACE.md`.

### 4.6 OPNsenseDocumentation

**Status:** LOGISCHE SCHNITTSTELLE BESCHLOSSEN / technischer Contract OFFEN

Vorgelagerter Pfad:

```text
RAW -> Sanitize -> Secret Scan -> Validate -> Normalize -> Netzwerkmodell -> DocumentationEngine
```

Hinweis:

Eine frühere Erwähnung von `network.mmd` ist keine globale Festlegung auf Mermaid.

### 4.7 CUST-* Projekte

**Status:** BESCHLOSSENER RAHMEN

Die Engine erzeugt kundenspezifische Artefakte für `CUST-<Debitor>-<Name>`.

Vorgesehene Bereiche:

- `CustomerConfiguration`
- `Documentation`
- ggf. separate RAW-Repositories

`CUST-00000` ist als Testkunde vorgesehen.

---

## 5. Offene Architekturentscheidungen

Die folgenden Punkte dürfen erst nach Sichtung der jeweils notwendigen Schnittstellen entschieden werden.

### DE-OPEN-001 – Input-Contract

Zu entscheiden:

- technische Verpackung des Inputs,
- Schemaformat,
- Versionierung,
- mehrere Collector-Artefakte vs. aggregierter Input,
- Umgang mit optionalen Datenquellen.

### DE-OPEN-002 – Internes Modell

Zu entscheiden:

- konkretes Document View Model,
- konkretes Infrastructure/Relationship Model,
- konkretes Diagram View Model,
- Referenzmodell zwischen Objekten,
- Modellversionierung.

Die Notwendigkeit einer semantischen View-Schicht ist mit DE-DEC-012 bereits beschlossen; offen ist deren konkrete technische Ausgestaltung.

### DE-OPEN-003 – Template Engine

Zu entscheiden:

- Template-Konzept,
- Template-Syntax,
- Template-Versionierung,
- Conditional Sections,
- Wiederverwendung gemeinsamer Sections.

### DE-OPEN-004 – Markdown-Renderer

Zu entscheiden:

- konkrete Rendering-Implementierung,
- Tabellenregeln,
- Escape-/Encoding-Verhalten,
- deterministische Sortierung,
- Behandlung fehlender Werte.

### DE-OPEN-005 – Diagrammformat und Renderer

Zu entscheiden:

- Diagrammformat,
- Renderer,
- internes technisches Diagrammschema,
- Erzeugung und Einbettung der Diagramme in Markdown.

Es besteht aktuell keine globale Festlegung auf Mermaid, PlantUML, Graphviz, eine eigene SVG-Pipeline oder ein anderes Format.

### DE-OPEN-006 – Renderer-spezifischer Symbol-, Icon- und Layoutstandard

Der fachliche Prototypstandard ist in `DIAGRAM_ENGINE_STANDARD.md` dokumentiert.

Bereits fest:

- offizielle Azure Architecture Icons,
- keine willkürliche Icon-Manipulation,
- nur belegte Ressourcen/Relationships,
- Progressive Disclosure,
- Legenden für visuelle Semantik,
- semantische Zonen statt rein freiem Graph-Autolayout,
- fünf Standard-Views als aktueller Prototyprahmen,
- Backup-Sicht aus Perspektive geschützter Ressourcen,
- Coverage statt Erfindung für nicht erhobene Domänen.

Noch zu definieren:

- konkreter Icon-Katalog und Updateprozess,
- Größen,
- exakte Positionierung,
- Grid-/Abstandssystem,
- Containergeometrie,
- Linien-/Kantenrouting,
- Richtungspfeile,
- Port-/Interface-Darstellung,
- VLAN-/Subnetz-/Segment-Darstellung,
- herstellerübergreifende Symbolregeln,
- konkrete Barrierefreiheitsregeln.

### DE-OPEN-007 – Diagrammvalidierung

Zu definieren:

- technische Validierungsregeln,
- fehlende Referenzen,
- doppelte IDs,
- ungültige Kanten,
- nicht auflösbare Icons,
- unbelegte Nodes/Edges als harter Fehler,
- Layout-/Rendering-Fehler,
- Snapshot-/Golden-Master-Vergleich,
- Quality-Gate-Schwellen.

### DE-OPEN-008 – Engine-Aufrufschnittstelle

Zu entscheiden:

- CLI/API oder vergleichbare technische Aufrufschnittstelle,
- Argumente,
- Konfiguration,
- Exitcodes,
- Logging,
- Fehlerobjekte.

### DE-OPEN-009 – Teststrategie

Zu definieren:

- Unit Tests,
- Contract-/Schema-Tests,
- Golden-Master-/Snapshot-Tests,
- Diagrammtests,
- Markdown-Tests,
- Faktentreue-/Halluzinationsregressionstests,
- End-to-End-Tests,
- Testdaten und Fixtures.

### DE-OPEN-010 – Knowledge Base / Publishing

Noch nicht entschieden ist, wie zentral gepflegte technische HowTos und Runbooks versioniert, freigegeben und für Techniker veröffentlicht werden.

Anforderungen:

- versionierte Source of Truth,
- keine parallele manuelle Pflege identischer Inhalte,
- technische HowTos müssen Plattformvoraussetzungen abbilden können.

Realweltbeispiel:

Ein OPNsense-Ersteinrichtungs-HowTo muss vor der Einrichtung darauf hinweisen, dass der Kunde über den DEVOPS-Bootstrap als `CUST-*` onboardet sein muss, damit Kunden-/Firewall-Repositories und Backup-/Dokumentationsintegration korrekt vorhanden sind.

Offene Konsum-/Publishing-Ziele:

- Azure DevOps / Wiki,
- SharePoint,
- Teams,
- kombinierte Publishing-Lösung.

Die Knowledge Base wird aktuell **nicht automatisch der DocumentationEngine zugerechnet**. Die genaue Abgrenzung bleibt Bestandteil dieser offenen Architekturentscheidung.

---

## 6. Umsetzungsreihenfolge

Die Reihenfolge ist so gewählt, dass keine Technologieentscheidung getroffen wird, bevor die dafür notwendigen Contracts und Verantwortungsgrenzen geklärt sind.

### Phase 0 – Projektbasis

**Ziel:** Kanonischen Ausgangsstand im Repository festhalten.

- [x] README mit Projektzweck und Architekturgrenzen anlegen.
- [x] konsolidierten Projektstand dokumentieren.
- [x] kanonischen Umsetzungsplan anlegen.
- [x] offene Technologieentscheidungen explizit markieren.
- [x] Erkenntnisse der ersten Azure-Dokumentations-/Diagrammprototypen konsolidieren.
- [x] Collector-/DocumentationEngine-Verantwortungsgrenze separat dokumentieren.
- [x] Techniker-Dokumentationsstandard als Prototyprahmen dokumentieren.
- [x] Diagram Engine Standard als Prototyprahmen dokumentieren.
- [x] verbindliche organisatorische Zielzuordnung aus `DEV_Ops_Bootstrap` übernommen: `DocumentationEngine` unter `00-Platform`; `AzureInfrastructureCollector` und `OPNsenseDocumentation` unter `10-Automation`. Der Name des aktuellen GitHub-Arbeitsrepositories ändert diese Zuordnung nicht.

### Phase 1 – Contracts und Verantwortungsgrenzen

**Ziel:** Stabile Eingangs- und Integrationsschnittstellen definieren.

- [x] aktuellen Output des `AzureInfrastructureCollector` fachlich/strukturell gegen den geplanten Engine-Input abgeglichen; finaler Contract bleibt offen.
- [ ] aktuellen Output von `OPNsenseDocumentation` gegen den geplanten Engine-Input abgleichen.
- [ ] gemeinsames Input-Contract festlegen.
- [ ] Schema-Versionierungsstrategie festlegen.
- [ ] Verantwortungsgrenze zu `SecurityValidation` festlegen.
- [ ] Aufruf-/Artefaktvertrag mit `PipelineTemplates` festlegen.
- [ ] erforderliche Kundenmetadaten aus `CUST-*` festlegen.

**Decision Gate:** Erst danach konkretes internes Modell und Rendering-Technologien festlegen.

### Phase 2 – Internes Modell und Core Engine

**Ziel:** Normalisierte Eingaben deterministisch in ein internes Dokumentationsmodell überführen.

- [ ] konkretes Infrastructure-/Relationship-Modell definieren.
- [ ] Document View Model definieren.
- [ ] Semantic View Builder definieren und implementieren.
- [ ] Diagram View Model definieren.
- [ ] Modellvalidierung implementieren.
- [ ] Collector-Inputs in internes Modell transformieren.
- [ ] deterministische Sortierung und Referenzauflösung implementieren.
- [ ] Coverage-/fehlende-Daten-Modell implementieren.
- [ ] Fehler- und Loggingmodell implementieren.

### Phase 3 – Markdown-Rendering

**Ziel:** Erste produktive standardisierte Kundendokumentation erzeugen.

- [ ] Template-Architektur auswählen und dokumentieren.
- [ ] Markdown-Renderer implementieren.
- [x] technikerorientierte Dokumentstruktur und Standardsektionen als Prototypstandard definiert (`TECHNICIAN_DOCUMENTATION_STANDARD.md`).
- [ ] Tabellenstandard implementieren.
- [ ] Regeln für optionale/fehlende Daten technisch implementieren.
- [ ] Fakt/Ableitung/Coverage-Kennzeichnung technisch festlegen.
- [ ] erste Golden-Master-Dokumente aufbauen.

### Phase 4 – Diagrammstandard und Diagrammgenerierung

**Ziel:** Reproduzierbare Netzwerk-, Infrastruktur- und Architekturdiagramme erzeugen.

- [x] Microsoft-/Prototyperkenntnisse als `DIAGRAM_ENGINE_STANDARD.md` konsolidiert.
- [x] fünf Standard-Views als Prototyprahmen definiert.
- [x] Faktentreue-/No-Invention-Regel definiert.
- [x] semantische Zonen und Progressive Disclosure als Layoutprinzip definiert.
- [ ] Diagrammformat und Renderer auswählen.
- [ ] konkretes Diagram View Model definieren.
- [ ] renderer-spezifischen Symbol-/Icon-/Layoutstandard finalisieren.
- [ ] Microsoft Azure Architecture Icons technisch integrieren und versionieren.
- [ ] Gesamtübersicht generieren.
- [ ] Netzwerk-&-Connectivity-View generieren.
- [ ] Workload-&-Deployment-View generieren.
- [ ] Backup-&-Recovery-View generieren.
- [ ] Security-&-Operations-/Coverage-View generieren.
- [ ] Diagrammvalidierung implementieren.
- [ ] Regressionstest gegen unbelegte/erfundene Ressourcen implementieren.

### Phase 5 – Pipeline-Integration und Quality Gates

**Ziel:** Vollständig reproduzierbarer zentraler Build.

- [ ] Engine-Aufrufschnittstelle finalisieren.
- [ ] Integration in `PipelineTemplates` umsetzen.
- [ ] Integration mit `SecurityValidation` umsetzen.
- [ ] tatsächlich gemeinsame Funktionen mit `SharedModules` integrieren.
- [ ] Build-Artefakte standardisieren.
- [ ] Fehlercodes/Logs standardisieren.
- [ ] Quality Gates aktivieren.

### Phase 6 – Automatisierte Tests und End-to-End-Test

**Ziel:** Verlässliche Regressionserkennung und Pilotbetrieb.

- [ ] Unit Tests.
- [ ] Schema-/Contract-Tests.
- [ ] Golden-Master-/Snapshot-Tests.
- [ ] Diagrammtests.
- [ ] Faktentreue-/No-Invention-Regressionstests.
- [ ] End-to-End-Test mit `CUST-00000`.
- [ ] erster realer Pilotkunde nach bestätigtem Gesamtprojektstand.

### Phase 7 – Spätere Ausgabeformate

**Status:** SPÄTER

- [ ] PDF-Ausgabe.
- [ ] DOCX-Ausgabe.
- [ ] dafür notwendigen Export-/Rendering-Stack auswählen.

### Phase 8 – Weitere Datenquellen und Publishing

**Status:** SPÄTER / teilweise OPEN

Mögliche Erweiterungen:

- [ ] On-Premises-/Hyper-V-Collector integrieren.
- [ ] Switch-/Layer-2-Collector integrieren.
- [ ] weitere Hersteller-Iconsets.
- [ ] weitere Dokumentationstypen.
- [ ] Knowledge-Base-/Publishing-Architektur nach eigener Architekturentscheidung integrieren.

---

## 7. Definition of Done für die erste produktive Version

Die erste produktive Version der `DocumentationEngine` gilt erst dann als abgeschlossen, wenn mindestens folgende Punkte erfüllt sind:

- definierter und versionierter Input-Contract,
- Verarbeitung mindestens der vorgesehenen initialen Collector-Daten,
- internes validiertes Modell,
- Semantic View Builder / dokumentierte View-Transformation,
- standardisierte technikerorientierte Markdown-Ausgabe,
- mindestens die initial erforderlichen Infrastruktur-/Netzwerkdiagramme,
- verbindlicher Diagramm- und Iconstandard,
- deterministische Diagrammerzeugung ohne unbelegte Ressourcen,
- Integration in zentrale `PipelineTemplates`,
- abgestimmte Grenze zu `SecurityValidation`,
- automatisierte Tests,
- reproduzierbarer Build,
- erfolgreicher End-to-End-Test mit einem provisionierten `CUST-*`-Testkunden.

PDF, DOCX und die endgültige Knowledge-Base-/Publishing-Lösung sind keine Voraussetzung für diese erste produktive Version.

---

## 8. Entscheidungsprotokoll

| ID | Thema | Status | Entscheidung |
|---|---|---|---|
| DE-DEC-001 | Organisatorische Zuordnung | BESCHLOSSEN | `DocumentationEngine` unter `00-Platform`; Collector unter `10-Automation` |
| DE-DEC-002 | Provisionierung | BESCHLOSSEN | DEVOPS-/Platform-Bootstrap |
| DE-DEC-003 | Collector-Trennung | BESCHLOSSEN | Collector normalisieren; Engine dokumentiert und inventarisiert Quelle nicht erneut |
| DE-DEC-004 | Validation | BESCHLOSSEN | Fail Closed; Detailgrenze noch offen |
| DE-DEC-005 | Pipeline | BESCHLOSSEN | zentrale `PipelineTemplates` |
| DE-DEC-006 | Shared Code | BESCHLOSSEN | `SharedModules` bei echter Wiederverwendung |
| DE-DEC-007 | Initialer Output | BESCHLOSSEN | Markdown |
| DE-DEC-008 | Azure Icons | BESCHLOSSEN | offizielle Microsoft Azure Architecture Icons |
| DE-DEC-009 | Qualität | BESCHLOSSEN | reproduzierbare Builds + automatisierte Tests |
| DE-DEC-010 | Faktentreue | BESCHLOSSEN | nur belegte Kundenressourcen/-beziehungen; Referenzarchitektur nur als Layoutreferenz |
| DE-DEC-011 | Zielgruppe | BESCHLOSSEN | Technikerorientierung vor maschinennahem Inventar |
| DE-DEC-012 | View-Schicht | BESCHLOSSEN | Relationship Graph -> Semantic View Builder -> View Models -> Renderer |
| DE-DEC-013 | Generative Bilder | BESCHLOSSEN | keine technische Diagramm-Source-of-Truth |
| DE-OPEN-001 | Input-Contract | OFFEN | – |
| DE-OPEN-002 | Konkretes internes Modell | OFFEN | – |
| DE-OPEN-003 | Template Engine | OFFEN | – |
| DE-OPEN-004 | Markdown-Renderer | OFFEN | – |
| DE-OPEN-005 | Diagrammformat/Renderer | OFFEN | – |
| DE-OPEN-006 | Renderer-spezifischer Symbol-/Layoutstandard | OFFEN | fachlicher Prototypstandard vorhanden |
| DE-OPEN-007 | Diagrammvalidierung | OFFEN | – |
| DE-OPEN-008 | Engine-Aufruf | OFFEN | – |
| DE-OPEN-009 | Teststrategie | OFFEN | – |
| DE-OPEN-010 | Knowledge Base / Publishing | OFFEN | – |

---

## 9. Änderungsprotokoll

### 2026-08-13 – Organisatorische Zuordnung klargestellt

- die bereits in `DEV_Ops_Bootstrap` beschlossene Projektzuordnung ausdrücklich übernommen,
- `DocumentationEngine` verbindlich unter `00-Platform` dokumentiert,
- `AzureInfrastructureCollector` und `OPNsenseDocumentation` verbindlich unter `10-Automation` dokumentiert,
- den widersprüchlichen offenen Prüfpunkt zur Zielzuordnung in Phase 0 geschlossen,
- klargestellt, dass der aktuelle GitHub-Repositoryname die Azure-DevOps-Zuordnung nicht ändert oder erneut öffnet.

### 2026-08-13 – Diagramm-/Technikerprototypen konsolidiert

- Verantwortungsgrenze zum `AzureInfrastructureCollector` geschärft: keine erneute Azure-Inventarisierung durch die DocumentationEngine,
- aktuellen modularen Azure-Collector-Output als bekannten Iststand dokumentiert,
- technikerorientierte Dokumentstruktur aus den ersten Real-Daten-Prototypen konsolidiert,
- Microsoft Architecture Icons / Well-Architected Diagram Guidance / Resource Visualizer / Landing Zone als Referenzquellen dokumentiert,
- Semantic View Builder als notwendige Abstraktionsschicht beschlossen,
- fünf Standard-Views als Prototyprahmen festgehalten,
- Backup-&-Recovery-Sicht auf die geschützte Ressource als Einstieg ausgerichtet,
- Security-&-Operations-Coverage-Regel bei noch fehlenden Fachinputs festgehalten,
- kritischen Prototypfehler mit erfundenen Firewall-/Bastion-Komponenten in eine harte No-Invention-Regel überführt,
- generative Bildmodelle als technische Diagramm-Source-of-Truth ausgeschlossen,
- konkrete Renderer-/Layouttechnologie weiterhin ausdrücklich offen gelassen.

### 2026-08-13 – Initialisierung

- initialen kanonischen Projektstand angelegt,
- bereits beschlossene Gesamtprojektentscheidungen übernommen,
- offene Technologieentscheidungen ausdrücklich nicht vorweggenommen,
- Abhängigkeiten zu Bootstrap, `PipelineTemplates`, `SecurityValidation`, `SharedModules`, `AzureInfrastructureCollector`, `OPNsenseDocumentation` und `CUST-*` konsolidiert,
- Knowledge-Base-/Publishing-Frage als offenen Plattformpunkt aufgenommen.
