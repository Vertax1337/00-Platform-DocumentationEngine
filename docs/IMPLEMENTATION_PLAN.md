# DocumentationEngine – kanonischer Umsetzungsplan

Stand: 2026-09-01

Dieses Dokument ist der kanonische technische Umsetzungsplan für `00-Platform / DocumentationEngine`.

Neue Architekturentscheidungen, technische Standards, offene Punkte, Implementierungsschritte und erreichte Umsetzungsstände werden hier fortlaufend gepflegt.

Bereits im Gesamtprojekt getroffene Entscheidungen werden übernommen und nicht ohne ausdrückliche Neubewertung erneut geöffnet.

---

## 1. Statusmodell

- **BESCHLOSSEN** – Architektur-/Technikentscheidung ist getroffen.
- **IMPLEMENTIERT** – Entscheidung ist technisch umgesetzt und im Repository vorhanden.
- **OFFEN** – Entscheidung oder konkrete Umsetzung ist noch nicht festgelegt.
- **SPÄTER** – bekannte mögliche Erweiterung, nicht Bestandteil der ersten produktiven Version.

Eine Phase gilt erst dann als abgeschlossen, wenn sowohl Fachdokumentation als auch kanonischer Umsetzungsplan denselben bestätigten Status enthalten.

---

## 2. Ziel des Unterprojekts

Die `DocumentationEngine` verarbeitet normalisierte und validierte technische Quellen zu standardisierten technischen Kundendokumentationen und reproduzierbaren Diagrammen.

Initiale Quellen umfassen zwei unterschiedliche Perspektiven:

```text
Actual State
  -> Collector-Artefakte

Desired State
  -> Bicep / IaC-Artefakte
```

Zielartefakte umfassen insbesondere:

- strukturierte technische Texte,
- Tabellen,
- Infrastrukturübersichten,
- Netzwerkdiagramme,
- Architekturdiagramme,
- später Desired-vs-Actual-/Drift-Sichten.

Der kanonische Dokumentvertrag ist ein rendererunabhängiges `Document View Model`. Markdown ist der erste produktive Renderer. DOCX und PDF folgen später als weitere Renderer desselben Document View Models.

---

## 3. Bereits beschlossene Architekturgrenzen

### DE-DEC-001 – Verbindliche organisatorische Zuordnung

**Status:** BESCHLOSSEN

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

Die `DocumentationEngine` ist zentrale Plattformlogik im Azure-DevOps-Projekt `00-Platform` und wird nicht pro Kunde oder Collector dupliziert.

Der aktuelle GitHub-Arbeitsrepositoryname `Vertax1337/10-DocumentationEngine` ändert diese Zielzuordnung nicht.

### DE-DEC-002 – Repository-Provisionierung

**Status:** BESCHLOSSEN

Die Provisionierung erfolgt über den bestehenden DEVOPS-/Platform-Bootstrap. Diese Bootstrap-Architektur wird hier nicht neu entworfen.

### DE-DEC-003 – Trennung Collector und Dokumentationslogik

**Status:** BESCHLOSSEN

Collector übernehmen quellspezifische Erfassung, Parsing und Normalisierung.

Die DocumentationEngine inventarisiert Azure nicht parallel neu. Für Azure-Actual-State konsumiert sie freigegebene normalisierte Collector-Artefakte.

Diese Trennung schließt IaC nicht aus: Bicep ist eine separate Desired-State-Quelle und kein Ersatz für den Collector-Iststand.

### DE-DEC-004 – Validierte Eingaben / Fail Closed

**Status:** BESCHLOSSEN

Relevante Schema-, Security-, Contract- und Quality-Fehler stoppen den Build nach dem Fail-Closed-Prinzip.

Die genaue Verantwortungsabgrenzung zu `SecurityValidation` bleibt noch festzulegen.

### DE-DEC-005 – Zentrale Pipeline-Integration

**Status:** BESCHLOSSEN

Die Einbindung erfolgt über zentrale `PipelineTemplates`; Kundenprojekte duplizieren die Engine-Orchestrierung nicht.

### DE-DEC-006 – SharedModules

**Status:** BESCHLOSSEN

Gemeinsame Komponenten werden erst bei tatsächlich belegter Wiederverwendung nach `SharedModules` ausgelagert.

### DE-DEC-007 – Markdown-first

**Status:** BESCHLOSSEN

Markdown ist das erste produktive Dokumentausgabeformat bzw. der erste produktive Dokumentrenderer. Markdown ist nicht die kanonische Dokument-Source-of-Truth. PDF und DOCX folgen später als weitere Renderer desselben Document View Models.

### DE-DEC-008 – Offizielle Azure Architecture Icons

**Status:** BESCHLOSSEN

Azure-Diagramme verwenden offizielle Microsoft Azure Architecture Icons. Hersteller-Icons werden nicht willkürlich verzerrt, gespiegelt, gedreht oder umgefärbt.

### DE-DEC-009 – Reproduzierbare Builds und automatisierte Tests

**Status:** BESCHLOSSEN

Die Engine muss reproduzierbar bauen und automatisiert testbar sein.

### DE-DEC-010 – Faktentreue vor Referenzarchitektur

**Status:** BESCHLOSSEN

Ein Infrastruktur- oder Serviceelement darf nur dargestellt werden, wenn es durch eine freigegebene Quelle belegt ist.

Referenzarchitekturen dienen nur als Kommunikations-/Layoutreferenz. Fehlende Kundenkomponenten werden niemals ergänzt.

### DE-DEC-011 – Technikerorientierung vor Inventar

**Status:** BESCHLOSSEN

Einstieg über Orientierung, Architektur, Workloads, Abhängigkeiten, Protection und Operations; maschinennahe Inventardetails nachgelagert.

### DE-DEC-012 – Semantische View-Schicht

**Status:** BESCHLOSSEN

```text
Source Data
  -> Canonical Relationship Graph
  -> Semantic View Builder
  -> Document / Diagram View Model
  -> Renderer
  -> Output
```

### DE-DEC-013 – Generative Bilder sind keine technische Source of Truth

**Status:** BESCHLOSSEN

Generative Bildmodelle dürfen Stil-/Mockup-Zwecken dienen, aber nicht den finalen technischen Diagramminhalt bestimmen.

### DE-DEC-014 – Providergrenze des Azure Relationship Contracts

**Status:** BESCHLOSSEN

Collector-P9 vereinheitlicht den Azure-seitigen Actual-State-Relationship-Input.

Das globale providerübergreifende DocumentationEngine-Modell wird unabhängig davon zentral definiert.

Der produktive Azure-Actual-State-Adapter wird erst auf Basis des stabilisierten und versionierten P9-Schemas finalisiert.

### DE-DEC-015 – Providerunabhängiger Canonical Infrastructure Core

**Status:** BESCHLOSSEN / technische Implementierung offen

Der Core besteht aus:

- `InfrastructureNode`,
- `Relationship`,
- `EvidenceReference`,
- `CoverageRecord`.

Zusätzlich besitzt jeder Canonical Graph einen validierten Graph-Envelope mit einer eindeutigen Source-Perspektive.

Details: [`docs/architecture/CANONICAL_MODEL.md`](architecture/CANONICAL_MODEL.md).

### DE-DEC-016 – Bicep/IaC ist initiale First-Class-Desired-State-Quelle

**Status:** BESCHLOSSEN / technische Implementierung offen

Bicep wird nicht als spätere Sondererweiterung behandelt, sondern von Beginn an in Core-, Adapter-, View- und Testarchitektur eingeplant.

Verbindlich gilt:

```text
Azure Actual State
  AzureInfrastructureCollector
      -> Azure Actual-State Adapter
      -> Canonical Graph [actual]

Azure Desired State
  Bicep / IaC
      -> Bicep Desired-State Adapter
      -> Canonical Graph [desiredTemplate|desiredDeployment]
```

Bicep ersetzt den Collector nicht als Iststandsbeweis. Der Collector ersetzt Bicep nicht als versionierte IaC-Sollquelle.

Actual und Desired werden nicht stillschweigend gemischt. Eine spätere Reconciliation erfolgt über einen expliziten Contract mit stabilen technischen Identitäten.

Verbindliche Bicep-/IaC-Schnittstelle: [`docs/IAC_BICEP_INTERFACE.md`](IAC_BICEP_INTERFACE.md).

### DE-DEC-017 – Rendererunabhängiges Document View Model als kanonischer Dokumentvertrag

**Status:** BESCHLOSSEN / technische Implementierung offen

Der fachliche Dokumentzustand der DocumentationEngine wird in einem strukturierten, rendererunabhängigen `Document View Model` (DVM) gehalten.

Verbindlich gilt:

```text
Canonical Infrastructure Model
        |
        v
Semantic View Builder
        |
        v
Document View Model
        |
        +--> Markdown Renderer
        +--> DOCX Renderer
        `--> PDF/HTML Renderer
```

Markdown ist damit der erste produktive Renderer, aber nicht die Dokument-Source-of-Truth.

DOCX und PDF werden später als weitere Renderer aus demselben DVM erzeugt. Eine dauerhafte kanonische Produktionsarchitektur `Markdown -> DOCX/PDF` ist nicht vorgesehen.

Das DVM wird von Beginn an ausreichend strukturiert ausgelegt, um professionelle Dokumentelemente wie Sections, Tabellen, Figures, Callouts, Inhaltsverzeichnis-fähige Überschriften und formatneutrale Layout-Hinweise abzubilden. Renderer-spezifische Details wie OOXML-IDs, Markdown-Syntax oder PDF-Objekte gehören nicht in das DVM.

Verbindliche Fachspezifikation: [`docs/architecture/DOCUMENT_VIEW_MODEL.md`](architecture/DOCUMENT_VIEW_MODEL.md).

---

## 4. Externe Abhängigkeiten und Schnittstellen

### 4.1 DEVOPS / PlatformBootstrap

**Status:** BESCHLOSSENER RAHMEN / externe Implementierung

- keine eigene Kundenprovisionierung,
- Nutzung provisionierter `CUST-*`-Strukturen,
- keine Duplizierung der Bootstrap-Logik.

Offen: konkrete Repository-/Pfad-/Metadatenübergabe.

### 4.2 PipelineTemplates

**Status:** BESCHLOSSENER RAHMEN / konkrete Schnittstelle OFFEN

Zu definieren:

- Template-Aufruf,
- Parameter,
- Input-/Output-Artefakte,
- Working Directory,
- Exitcodes,
- Logging-/Fehlervertrag,
- Versionierung.

Dabei müssen sowohl Collector-Actual-State-Artefakte als auch IaC/Bicep-Desired-State-Artefakte über einen reproduzierbaren Buildvertrag übergeben werden können.

### 4.3 SecurityValidation

**Status:** BESCHLOSSENER RAHMEN / Verantwortungsgrenze OFFEN

Zu definieren:

- vorgelagerte Pflichtprüfungen,
- Engine-eigene Konsistenzprüfungen,
- Fehlerklassen,
- Fail-Closed-Grenzen.

IaC-Evidence und sanitizte Deploymentkontexte dürfen keine Secret-Werte in Engine-Artefakte übernehmen.

### 4.4 SharedModules

**Status:** BESCHLOSSENER RAHMEN / konkrete Nutzung OFFEN

Nur tatsächliche Wiederverwendung rechtfertigt Auslagerung.

### 4.5 AzureInfrastructureCollector – Actual State

**Status:** LOGISCHE SCHNITTSTELLE BESCHLOSSEN / aktueller Output gesichtet / produktiver Relationship-Contract abhängig von P9

Bekannter Output:

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

Produktive Azure-Actual-State-Relationships werden erst nach P9 finalisiert.

Details: `COLLECTOR_INTERFACE.md`.

### 4.6 Bicep / IaC – Desired State

**Status:** ZIELRAHMEN BESCHLOSSEN / technische Schnittstelle noch zu implementieren

Bicep ist für IaC-verwaltete Workloads initiale Desired-State-Quelle.

Der Adapter muss mindestens nachvollziehen können:

- immutable Source-Version/Commit,
- Root-Bicep-Datei und Module,
- Compiler-/Buildprovenance,
- Ressourcen/Scopes/Conditions,
- maschinenauflösbare Relationships,
- Parametervertrag,
- sichere Parameterkennzeichnung,
- sanitizten Deploymentkontext für `desiredDeployment`,
- keine Secret-Werte.

Details: `IAC_BICEP_INTERFACE.md`.

### 4.7 OPNsenseDocumentation

**Status:** LOGISCHE SCHNITTSTELLE BESCHLOSSEN / technischer Contract OFFEN

```text
RAW -> Sanitize -> Secret Scan -> Validate -> Normalize -> Netzwerkmodell -> DocumentationEngine
```

Keine globale Mermaid-Festlegung.

### 4.8 CUST-* Projekte

**Status:** BESCHLOSSENER RAHMEN

Die Engine erzeugt kundenspezifische Artefakte für `CUST-<Debitor>-<Name>`.

Vorgesehene Bereiche:

- `CustomerConfiguration`,
- `Documentation`,
- ggf. getrennte RAW-Repositories.

`CUST-00000` bleibt Testkunde.

---

## 5. Offene Architekturentscheidungen

### DE-OPEN-001 – Input-Contract

**Status:** OFFEN

Zu entscheiden:

- physische Paketstruktur,
- Schemaformat,
- Versionierung,
- mehrere Source-Artefakte vs. aggregierter Input,
- optionale Datenquellen,
- gemeinsamer Envelope für Collector- und IaC-Inputs.

Die fachlichen Bicep-Mindestanforderungen sind bereits in `IAC_BICEP_INTERFACE.md` festgelegt.

### DE-OPEN-002 – Internes Modell

**Status:** TEILWEISE BESCHLOSSEN / Rest OFFEN

Bereits beschlossen:

- Canonical Infrastructure Core,
- Evidence/Provenance,
- Coverage,
- Graph-Perspektiven `actual`, `desiredTemplate`, `desiredDeployment`,
- keine implizite Actual-/Desired-Mischung,
- Trennung Core ↔ Semantic Views ↔ Renderer,
- rendererunabhängiges Document View Model als kanonischer Dokumentvertrag,
- Markdown/DOCX/PDF als getrennte Renderer desselben DVM.

Weiter offen:

- technische Repräsentation/Serialisierung des Canonical Core,
- Modellversionierung/Kompatibilität,
- technische Repräsentation/Serialisierung und Versionierung des Document View Models,
- Diagram View Model,
- Reconciliation View/Result Model,
- konkrete Semantic-View-Implementierung.

### DE-OPEN-003 – Template Engine

**Status:** OFFEN

Template-Konzept, Syntax, Versionierung, Conditional Sections, Wiederverwendung.

### DE-OPEN-004 – Markdown-Renderer

**Status:** OFFEN

Implementierung aus dem Document View Model, Tabellenregeln, Encoding/Escaping, deterministische Sortierung, fehlende Werte.

### DE-OPEN-005 – Diagrammformat und Renderer

**Status:** OFFEN

Keine globale Festlegung auf Mermaid, PlantUML, Graphviz, ELK oder eigene SVG-Pipeline.

### DE-OPEN-006 – Renderer-spezifischer Symbol-/Layoutstandard

**Status:** OFFEN / fachlicher Prototypstandard vorhanden

Bereits fest:

- offizielle Azure Icons,
- Faktentreue,
- Progressive Disclosure,
- semantische Zonen,
- Legenden,
- fünf Standard-Views,
- Coverage statt Erfindung.

Noch offen: konkreter Icon-Katalog, Grid, Abstände, Containergeometrie, Kantenrouting, Barrierefreiheit.

### DE-OPEN-007 – Diagrammvalidierung

**Status:** OFFEN

Fehlende Referenzen, doppelte IDs, ungültige Kanten, Iconauflösung, No-Invention, Layout-/Renderingfehler, Golden Masters, Quality Gates.

### DE-OPEN-008 – Engine-Aufrufschnittstelle

**Status:** OFFEN

CLI/API, Argumente, Konfiguration, Exitcodes, Logging, Fehlerobjekte.

### DE-OPEN-009 – Teststrategie

**Status:** OFFEN

Unit-, Contract-, Snapshot-/Golden-Master-, Diagramm-, DVM-, Markdown-, Cross-Renderer-, No-Invention- und E2E-Tests.

### DE-OPEN-010 – Knowledge Base / Publishing

**Status:** OFFEN

Azure DevOps/Wiki, SharePoint, Teams oder kombinierte Publishing-Lösung. Keine parallele manuelle Pflege identischer Inhalte.

---

## 6. Umsetzungsreihenfolge

### Phase 0 – Projektbasis

**Ziel:** Kanonischen Ausgangsstand festhalten.

- [x] README / Projektzweck.
- [x] konsolidierter Projektstand.
- [x] kanonischer Umsetzungsplan.
- [x] offene Technologieentscheidungen markiert.
- [x] Prototyperkenntnisse konsolidiert.
- [x] Collector-/Engine-Verantwortungsgrenze dokumentiert.
- [x] Techniker-Dokumentationsstandard dokumentiert.
- [x] Diagram Engine Standard dokumentiert.
- [x] organisatorische Zielzuordnung übernommen.

### Phase 1 – Contracts und Verantwortungsgrenzen

**Ziel:** Stabile Eingangs- und Integrationsschnittstellen definieren.

- [x] aktuellen AzureInfrastructureCollector-Output fachlich gesichtet.
- [x] P9-Contract-Grenze dokumentiert.
- [x] providerunabhängigen Diagramm-/Relationship-Bedarf spezifiziert.
- [x] Bicep/IaC als initiale Desired-State-Quelle architektonisch eingeplant.
- [x] rendererunabhängiges Document View Model als kanonischen Dokumentvertrag fachlich beschlossen.
- [ ] Bicep/IaC-Input-Paket fachlich/technisch finalisieren.
- [ ] nach Collector-P9 vereinheitlichtes Azure-Relationship-Schema prüfen/versionieren.
- [ ] produktiven Azure-Actual-State-Adapter finalisieren.
- [ ] OPNsense-Output abgleichen.
- [ ] gemeinsames Input-Contract festlegen.
- [ ] Schema-/Modellversionierungsstrategie festlegen.
- [ ] Grenze zu `SecurityValidation` festlegen.
- [ ] Aufruf-/Artefaktvertrag mit `PipelineTemplates` festlegen.
- [ ] erforderliche `CUST-*`-Metadaten festlegen.

**Decision Gate:** Bicep-Desired-State-Adapter und globales Core-/View-Modell sind kein P9-Blocker. Nur der produktive Azure-Actual-State-Adapter bleibt P9-gegated.

### Phase 2 – Internes Modell und Core Engine

**Ziel:** Actual- und Desired-State-Inputs deterministisch in perspektivgetrennte interne Modelle überführen.

- [x] providerunabhängiges Infrastructure-/Relationship-Core-Modell fachlich definiert.
- [x] Graph-Perspektiven fachlich definiert.
- [x] rendererunabhängigen Document-View-Model-Fachcontract definiert.
- [ ] DE-WC-01 technisch implementieren.
- [ ] Document View Model technisch repräsentieren und validieren.
- [ ] Diagram View Model definieren.
- [ ] Semantic View Builder definieren/implementieren.
- [ ] Modellvalidierung implementieren.
- [ ] Bicep Desired-State Adapter implementieren.
- [ ] Azure Actual-State Fixture-/Prototype-Adapter implementieren.
- [ ] produktiven Azure Actual-State Adapter nach P9 finalisieren.
- [ ] deterministische Sortierung/Referenzauflösung implementieren.
- [ ] Coverage implementieren.
- [ ] Fehler-/Loggingmodell implementieren.
- [ ] Desired/Actual Reconciliation Contract implementieren.

#### DE-WC-01 – Canonical Infrastructure Core technisch implementieren

**Status:** BESCHLOSSEN / IMPLEMENTIERUNG OFFEN

Verbindlicher Umfang:

- technische Repräsentation von `InfrastructureNode`, `Relationship`, `EvidenceReference`, `CoverageRecord`,
- Graph-Envelope mit `perspective`,
- zentrale Registries,
- referentielle Integrität,
- Evidence-Pflicht,
- fail-closed Validierung,
- deterministische Sortierung,
- Coverage-Validierung,
- positive/negative Fixtures,
- No-Invention-Regression,
- Provider-/Source-Unabhängigkeitsnachweis,
- mindestens ein Bicep-/Desired-State-Fixture im selben Core ohne Sondermodell.

Nicht Bestandteil:

- vollständiger Bicep-Parser,
- produktiver Azure-P9-Adapter,
- Renderer,
- Markdown/PDF/DOCX,
- PipelineTemplates-Integration.

**Hartes Gate:** Core, Fachdokumentation und kanonischer Plan müssen denselben Status tragen; Contract-/Fail-Closed-/Determinismus-/No-Invention-/Perspektivtests müssen grün sein.

#### DE-WC-02 – Semantic View Contracts

**Status:** GEPLANT

- fünf Standard-Views formal definieren,
- akzeptierte Perspektiven pro View definieren,
- Actual-/Desired-Kennzeichnung verpflichtend machen,
- keine Präsentationsgruppen in den Canonical Core zurückschreiben.

#### DE-WC-02.1 – Document View Model Contract

**Status:** BESCHLOSSEN / TECHNISCHE IMPLEMENTIERUNG OFFEN

Fachlicher Contract: [`docs/architecture/DOCUMENT_VIEW_MODEL.md`](architecture/DOCUMENT_VIEW_MODEL.md).

Verbindlich:

- DVM ist kanonischer Dokumentvertrag,
- Sections/Paragraphs/Tables/Figures/Callouts werden strukturiert modelliert,
- Actual-/Desired-/Reconciliation-Perspektive bleibt erhalten,
- DVM und Diagram View Model bleiben getrennte Verträge,
- Markdown ist erster Renderer,
- DOCX/PDF sind spätere Renderer desselben DVM,
- keine kanonische `Markdown -> DOCX/PDF`-Kette.

**Technisches Gate:** DVM darf erst als `IMPLEMENTIERT` gelten, wenn technische Repräsentation/Versionierung, Validierung, deterministische Struktur, vollständiges Fixture sowie ein ausschließlich aus dem DVM rendernder Markdown-Renderer mit Golden-Master-/Contract-Tests vorhanden sind.

#### DE-WC-03 – Initiale Source-/Provider-Adapter

**Status:** GEPLANT

1. **Bicep Desired-State Adapter** als initialer produktnaher Adapter.
2. **Azure Actual-State Adapter Prototype** gegen vorhandene Collector-Fixtures.
3. Produktive Azure-Actual-State-Finalisierung erst nach P9.
4. OPNsense-Vertrag nach Sichtung.

Bicep ist hierbei ausdrücklich kein späterer Optional-Workchunk mehr.

#### DE-WC-04 – Desired/Actual Reconciliation Contract

**Status:** GEPLANT

- stabile Korrelationsschlüssel,
- `matched`, `desiredMissing`, `actualUnmanaged`, `unresolved`,
- Property Drift nur bei eindeutig vergleichbarer Semantik,
- Evidence auf beiden Seiten,
- keine Name-only-Korrelation,
- Reconciliation-Output als eigenes Modell/View, nicht als stillschweigend gemergter Canonical Graph.

### Phase 3 – Document Rendering: Markdown first

**Ziel:** Erste produktive standardisierte Kundendokumentation aus dem rendererunabhängigen Document View Model erzeugen.

- [x] fachlichen Document-View-Model-Contract definiert.
- [ ] technische DVM-Repräsentation/Validierung implementieren.
- [ ] Template-Architektur auswählen.
- [ ] Markdown-Renderer ausschließlich aus dem DVM implementieren.
- [x] technikerorientierte Dokumentstruktur definiert.
- [ ] Tabellenstandard im DVM/Markdown-Renderer umsetzen.
- [ ] optionale/fehlende Daten technisch behandeln.
- [ ] Fakt/Ableitung/Coverage technisch kennzeichnen.
- [ ] Perspektive `actual` vs. `desired` in Ausgaben sichtbar machen.
- [ ] Golden-Master-Dokumente.
- [ ] sicherstellen, dass kein Markdown-spezifisches Markup in den kanonischen DVM-Contract zurückfließt.

### Phase 4 – Diagrammstandard und Diagrammgenerierung

**Ziel:** Reproduzierbare Infrastruktur-/Netzwerk-/Architekturdiagramme.

- [x] Diagram Engine Standard konsolidiert.
- [x] fünf Standard-Views definiert.
- [x] No-Invention-Regel definiert.
- [x] semantische Zonen / Progressive Disclosure definiert.
- [ ] Diagrammformat/Renderer auswählen.
- [ ] Diagram View Model implementieren.
- [ ] Icon-/Layoutstandard finalisieren.
- [ ] Azure Architecture Icons integrieren/versionieren.
- [ ] Actual-State-Gesamtübersicht generieren.
- [ ] Bicep-Desired-State-Architekturübersicht generieren.
- [ ] Netzwerk-&-Connectivity-View.
- [ ] Workload-&-Deployment-View.
- [ ] Backup-&-Recovery-View.
- [ ] Security-&-Operations-/Coverage-View.
- [ ] Reconciliation-/Drift-View sobald DE-WC-04 belastbar ist.
- [ ] Diagrammvalidierung.
- [ ] No-Invention-Regression.

### Phase 5 – Pipeline-Integration und Quality Gates

**Ziel:** Reproduzierbarer zentraler Build.

- [ ] Engine-Aufrufschnittstelle finalisieren.
- [ ] `PipelineTemplates` integrieren.
- [ ] `SecurityValidation` integrieren.
- [ ] notwendige SharedModules integrieren.
- [ ] Collector-Actual-State-Artefakte standardisieren.
- [ ] IaC/Bicep-Desired-State-Artefakte standardisieren.
- [ ] Document-View-Model-/Renderer-Artefakte standardisieren.
- [ ] Build-Artefakte standardisieren.
- [ ] Fehlercodes/Logs standardisieren.
- [ ] Quality Gates aktivieren.

### Phase 6 – Automatisierte Tests und End-to-End-Test

**Ziel:** Regressionserkennung und Pilotbetrieb.

- [ ] Unit Tests.
- [ ] Schema-/Contract-Tests.
- [ ] Bicep Adapter Contract Tests.
- [ ] Actual/Desired Perspective Tests.
- [ ] DVM Contract-/Validation Tests.
- [ ] Markdown Golden-Master Tests.
- [ ] Reconciliation Tests.
- [ ] Diagrammtests.
- [ ] No-Invention-Tests.
- [ ] E2E mit `CUST-00000` einschließlich mindestens eines IaC-verwalteten Workloads.
- [ ] erster realer Pilotkunde.

### Phase 7 – Weitere Document Renderer

**Status:** SPÄTER / ARCHITEKTURRAHMEN BESCHLOSSEN

DOCX und PDF werden nicht aus einem separat gepflegten Dokumentzustand erzeugt, sondern als weitere Renderer aus demselben Document View Model.

- [ ] DOCX-Renderer-Technologie auswählen.
- [ ] DOCX Renderer aus dem DVM implementieren.
- [ ] PDF-Renderer bzw. kontrollierten HTML/CSS->PDF-Pfad auswählen.
- [ ] PDF Renderer aus dem DVM implementieren.
- [ ] fachliche Cross-Renderer-Parität Markdown/DOCX/PDF testen.

Ein Prototyp darf Markdown als temporäres Zwischenformat verwenden; dies ist kein kanonischer Produktionsvertrag.

### Phase 8 – Weitere Datenquellen und Publishing

**Status:** SPÄTER / teilweise OPEN

- [ ] Hyper-V-/On-Premises-Collector.
- [ ] Switch-/Layer-2-Collector.
- [ ] weitere Hersteller-Iconsets.
- [ ] weitere Dokumentationstypen.
- [ ] Knowledge-Base-/Publishing-Architektur integrieren.

---

## 7. Definition of Done für die erste produktive Version

Die erste produktive Version gilt erst dann als abgeschlossen, wenn mindestens:

- definierter/versionierter Input-Contract,
- validierter Canonical Core,
- Actual-/Desired-Perspektivvertrag,
- mindestens ein produktnaher Bicep Desired-State Adapter,
- Azure Actual-State Adapter auf stabilisiertem P9-Contract,
- Semantic View Builder,
- versioniertes/validiertes rendererunabhängiges Document View Model,
- standardisierte technikerorientierte Markdown-Ausgabe, die ausschließlich aus dem DVM gerendert wird,
- initial erforderliche Infrastruktur-/Netzwerkdiagramme,
- mindestens eine Bicep-basierte Desired-State-Architektursicht,
- verbindlicher Diagramm-/Iconstandard,
- deterministische Diagrammerzeugung ohne unbelegte Ressourcen,
- zentrale Pipeline-Integration,
- abgestimmte SecurityValidation-Grenze,
- automatisierte Tests,
- reproduzierbarer Build,
- erfolgreicher E2E-Test mit `CUST-00000`.

Eine vollständige Property-Level-Drift-Engine, produktive DOCX-/PDF-Renderer und finale Knowledge-Base-/Publishing-Lösung sind keine Voraussetzung der ersten produktiven Version. Ihre spätere Ableitung aus demselben DVM ist jedoch architektonisch bereits festgelegt.

---

## 8. Entscheidungsprotokoll

| ID | Thema | Status | Entscheidung |
|---|---|---|---|
| DE-DEC-001 | Organisatorische Zuordnung | BESCHLOSSEN | `DocumentationEngine` unter `00-Platform`; Collector unter `10-Automation` |
| DE-DEC-002 | Provisionierung | BESCHLOSSEN | DEVOPS-/Platform-Bootstrap |
| DE-DEC-003 | Collector-Trennung | BESCHLOSSEN | Collector liefert Actual State; Engine inventarisiert nicht erneut |
| DE-DEC-004 | Validation | BESCHLOSSEN | Fail Closed |
| DE-DEC-005 | Pipeline | BESCHLOSSEN | zentrale `PipelineTemplates` |
| DE-DEC-006 | Shared Code | BESCHLOSSEN | `SharedModules` bei echter Wiederverwendung |
| DE-DEC-007 | Initialer Output | BESCHLOSSEN | Markdown als erster Renderer, nicht als Dokument-Source-of-Truth |
| DE-DEC-008 | Azure Icons | BESCHLOSSEN | offizielle Microsoft Azure Architecture Icons |
| DE-DEC-009 | Qualität | BESCHLOSSEN | reproduzierbare Builds + Tests |
| DE-DEC-010 | Faktentreue | BESCHLOSSEN | nur belegte Ressourcen/Relationships |
| DE-DEC-011 | Zielgruppe | BESCHLOSSEN | Technikerorientierung vor Inventar |
| DE-DEC-012 | View-Schicht | BESCHLOSSEN | Canonical Graph -> Semantic Views -> View Models -> Renderer |
| DE-DEC-013 | Generative Bilder | BESCHLOSSEN | keine technische Diagramm-Source-of-Truth |
| DE-DEC-014 | Azure Relationship Contract | BESCHLOSSEN | produktiver Actual-State-Adapter erst nach P9 |
| DE-DEC-015 | Canonical Infrastructure Core | BESCHLOSSEN | Nodes, Relationships, Evidence, Coverage + Graph-Perspektive |
| DE-DEC-016 | Bicep Desired State | BESCHLOSSEN | Bicep/IaC von Anfang an First-Class-Desired-State-Quelle |
| DE-DEC-017 | Document View Model | BESCHLOSSEN | rendererunabhängiges DVM ist kanonischer Dokumentvertrag; Markdown/DOCX/PDF sind Renderer |
| DE-OPEN-001 | Input-Contract | OFFEN | – |
| DE-OPEN-002 | Internes Modell Rest | TEILWEISE OFFEN | technische Repräsentation/Versionierung von Core/DVM, Diagram-/Reconciliation-Modelle offen |
| DE-OPEN-003 | Template Engine | OFFEN | – |
| DE-OPEN-004 | Markdown Renderer | OFFEN | aus DVM |
| DE-OPEN-005 | Diagrammformat/Renderer | OFFEN | – |
| DE-OPEN-006 | Renderer-spezifischer Layoutstandard | OFFEN | fachlicher Standard vorhanden |
| DE-OPEN-007 | Diagrammvalidierung | OFFEN | – |
| DE-OPEN-008 | Engine-Aufruf | OFFEN | – |
| DE-OPEN-009 | Teststrategie | OFFEN | – |
| DE-OPEN-010 | Knowledge Base / Publishing | OFFEN | – |

---

## 9. Änderungsprotokoll

### 2026-09-01 – Rendererunabhängiges Document View Model als kanonischer Dokumentvertrag beschlossen

- Markdown-first präzisiert: Markdown ist der erste produktive Renderer, nicht die Dokument-Source-of-Truth,
- `Document View Model` als kanonischen, rendererunabhängigen Dokumentvertrag beschlossen,
- DOCX und PDF als spätere Renderer desselben DVM eingeordnet,
- dauerhafte Produktionskette `Markdown -> DOCX/PDF` ausgeschlossen,
- Sections, Tabellen, Figures und Callouts als strukturierte DVM-Elemente vorgesehen,
- DOCX-/PDF-Fähigkeit bereits im DVM-Design berücksichtigt, ohne Renderer-Technologie vorzuziehen,
- `docs/architecture/DOCUMENT_VIEW_MODEL.md` als verbindliche Fachspezifikation aufgenommen,
- DE-WC-02.1 und die Renderer-/Testreihenfolge entsprechend ergänzt.

### 2026-09-01 – Bicep/IaC als initiale Desired-State-Quelle eingeplant

- frühere Einordnung von Bicep als nur mögliche spätere Adaptererweiterung korrigiert,
- Bicep als First-Class-Desired-State-Quelle beschlossen,
- Graph-Perspektiven `actual`, `desiredTemplate`, `desiredDeployment` eingeführt,
- Canonical Core so abgegrenzt, dass Collector- und IaC-Quellen denselben Kern nutzen,
- Bicep Desired-State Adapter in DE-WC-03 aufgenommen,
- Desired/Actual Reconciliation als DE-WC-04 eingeplant,
- klargestellt, dass P9 nur den produktiven Azure-Actual-State-Adapter blockiert, nicht Bicep,
- `IAC_BICEP_INTERFACE.md` als verbindlichen fachlichen Rahmen aufgenommen,
- Bicep-basierte Desired-State-Architekturansicht in die erste produktive Version aufgenommen.

### 2026-09-01 – DE-WC-01 / Canonical Infrastructure Core beschlossen

- `InfrastructureNode`, `Relationship`, `EvidenceReference`, `CoverageRecord` beschlossen,
- Evidence-/No-Invention-Invarianten festgelegt,
- providerunabhängige Node-/Relationship-Semantik beschrieben,
- Coverage und referentielle Integrität festgelegt,
- technische Scopes von View-/Diagrammzonen getrennt,
- Renderer-/Layoutattribute aus dem Core ausgeschlossen,
- Azure-P9-Gate beibehalten.

### 2026-08-13 – Azure-P9-Contract-Grenze abgegrenzt

- fachmodulspezifische Azure-Relationships als vorläufigen Input eingeordnet,
- P9 als Quelle des späteren kanonischen Azure-Actual-State-Relationship-Schemas festgehalten,
- globales Engine-Modell von P9 getrennt.

### 2026-08-13 – Organisatorische Zuordnung klargestellt

- `DocumentationEngine` verbindlich unter `00-Platform`,
- Collector unter `10-Automation`.

### 2026-08-13 – Diagramm-/Technikerprototypen konsolidiert

- Technikerorientierung,
- Semantic View Builder,
- fünf Standard-Views,
- Coverage-Regel,
- No-Invention-Regel,
- generative Bildmodelle als technische Source of Truth ausgeschlossen.

### 2026-08-13 – Initialisierung

- kanonischen Projektstand und Abhängigkeiten konsolidiert,
- offene Technologieentscheidungen ausdrücklich nicht vorweggenommen.
