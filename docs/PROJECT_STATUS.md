# DocumentationEngine – konsolidierter Projektstand

Stand: 2026-09-01

Dieses Dokument trennt zwischen bereits beschlossen, bereits implementiert, noch offen und später möglichen Erweiterungen.

## 1. Bereits beschlossen

### 1.1 Plattformzuordnung

Die `DocumentationEngine` ist zentrale Plattformlogik unter:

```text
00-Platform / DocumentationEngine
```

Collector verbleiben unter `10-Automation`. Die Engine wird nicht pro Kunde oder Collector dupliziert.

### 1.2 Zwei technische Perspektiven

Die DocumentationEngine verarbeitet von Beginn an zwei unterschiedliche technische Wahrheiten:

```text
Actual State
  -> Collector-Artefakte

Desired State
  -> Bicep / IaC-Artefakte
```

Für Azure gilt:

```text
AzureInfrastructureCollector
  -> belegter Iststand
  -> Canonical Graph [actual]

Bicep / IaC
  -> versionierter Sollstand
  -> Canonical Graph [desiredTemplate|desiredDeployment]
```

Bicep ersetzt den Collector nicht als Iststandsbeweis. Der Collector ersetzt Bicep nicht als IaC-Source-of-Truth.

Actual und Desired werden nicht stillschweigend zusammengeführt. Eine spätere Reconciliation ist eine explizite Transformation.

### 1.3 Collector-Grenze

Collector sind für quellspezifische Erfassung, Parsing und Normalisierung verantwortlich.

Die DocumentationEngine inventarisiert Azure nicht erneut live.

Für den produktiven Azure-Actual-State-Adapter bleibt das stabilisierte Collector-P9-Relationship-Schema Voraussetzung.

### 1.4 Bicep/IaC-Grenze

Bicep ist **keine spätere optionale Erweiterung mehr**, sondern als initiale First-Class-Desired-State-Quelle beschlossen.

Fachlich muss die Engine für IaC-Inputs mindestens verarbeiten bzw. nachweisen können:

- immutable Source-/Commit-Provenance,
- Root-Bicep und Module,
- Compiler-/Buildprovenance,
- Ressourcen/Scopes/Conditions,
- maschinenauflösbare Relationships,
- Parametervertrag,
- sichere Parameterkennzeichnung,
- sanitizten Deploymentkontext für konkrete `desiredDeployment`-Sichten,
- keine Secret-Werte in Engine-Artefakten.

Details: `IAC_BICEP_INTERFACE.md`.

### 1.5 Canonical Infrastructure Core

Der fachliche Core-Contract ist beschlossen:

```text
Canonical Graph Envelope
  |- perspective
  |- InfrastructureNode[]
  |- Relationship[]
  |- EvidenceReference[]
  `- CoverageRecord[]
```

Initiale Perspektiven:

```text
actual
desiredTemplate
desiredDeployment
```

Verbindlich sind:

- stabile technische IDs,
- Evidence-Pflicht für Nodes und Relationships,
- referentielle Integrität,
- deterministische Sortierung,
- explizite Coverage,
- keine Name-only-Relationships,
- keine Referenzarchitektur-Erfindungen,
- keine Renderer-/Layoutattribute im Core,
- keine implizite Actual-/Desired-Mischung.

### 1.6 Semantic View Layer

Beschlossen:

```text
Source Data
  -> Canonical Relationship Graph
  -> Semantic View Builder
  -> Document / Diagram View Model
  -> Renderer
```

Fünf Standard-Views bleiben:

1. Gesamtübersicht,
2. Netzwerk & Connectivity,
3. Workload & Deployment,
4. Backup & Recovery,
5. Security & Operations.

Views müssen später ihre Perspektive eindeutig ausweisen.

### 1.7 Faktentreue / No-Invention

Nur belegte Ressourcen und Relationships dürfen als technische Fakten erscheinen.

Referenzarchitekturen und generative Bilder dürfen Stil-/Layoutreferenz sein, aber keine technische Source of Truth.

### 1.8 Rendererunabhängiges Document View Model / Markdown-first

Der fachliche Dokumentzustand wird in einem strukturierten **Document View Model (DVM)** gehalten.

```text
Semantic View Builder
        |
        v
Document View Model
        |
        +--> Markdown Renderer
        +--> DOCX Renderer
        `--> PDF/HTML Renderer
```

Verbindlich gilt:

- das DVM ist der kanonische Dokumentvertrag,
- Markdown ist der erste produktive Renderer und nicht die Dokument-Source-of-Truth,
- DOCX und PDF werden später aus demselben DVM gerendert,
- eine dauerhafte Produktionsarchitektur `Markdown -> DOCX/PDF` ist nicht vorgesehen,
- Sections, Tabellen, Figures und Callouts werden strukturiert modelliert und nicht als Markdown-Fragmente gespeichert,
- Actual-/Desired-/Reconciliation-Perspektive bleibt im DVM erhalten,
- DOCX-/PDF-Fähigkeit wird im DVM bereits berücksichtigt, ohne deren konkrete Renderer-Technologie vorzuziehen.

Details: `architecture/DOCUMENT_VIEW_MODEL.md`.

### 1.9 Diagramme

- Azure-Diagramme mit offiziellen Microsoft Azure Architecture Icons,
- deterministische Diagrammerzeugung,
- semantische Zonen und Progressive Disclosure,
- Diagram View Model bleibt vom Document View Model getrennt; das DVM referenziert Diagrammartefakte als strukturierte Figures.

---

## 2. Bereits implementiert

| Bereich | Stand |
|---|---|
| Kanonischer DocumentationEngine-Umsetzungsplan | implementiert |
| Konsolidierter Projektstatus | implementiert |
| Collector-/Engine-Verantwortungsdokument | implementiert |
| Techniker-Dokumentationsstandard | implementiert |
| Diagram Engine Standard | implementiert |
| Prototyp-Erkenntnis-/Fehlerdokument | implementiert |
| Canonical-Infrastructure-Core-Fachspezifikation | implementiert |
| Bicep/IaC-Desired-State-Fachspezifikation | implementiert |
| Document-View-Model-Fachspezifikation | implementiert (`docs/architecture/DOCUMENT_VIEW_MODEL.md`) |
| Actual-/Desired-Perspektivvertrag | fachlich dokumentiert |
| Technische Core-Modell-/Validator-Implementierung | **noch nicht implementiert** |
| Technische Document-View-Model-Implementierung | **noch nicht implementiert** |
| Bicep Desired-State Adapter | **noch nicht implementiert** |
| Azure Actual-State Adapter auf P9 | **noch nicht implementiert** |
| Desired/Actual Reconciliation | **noch nicht implementiert** |
| Semantic View Builder | **noch nicht implementiert** |
| Markdown Renderer | **noch nicht implementiert** |
| DOCX Renderer | **noch nicht implementiert** |
| PDF Renderer | **noch nicht implementiert** |
| Produktiver End-to-End-Dokumentationsbuild | **noch nicht implementiert** |

Wichtig: „Fachspezifikation implementiert“ bedeutet, dass der Contract im Repository verbindlich dokumentiert ist. Es bedeutet nicht, dass die fachliche Engine bereits technisch umgesetzt ist.

---

## 3. Angrenzende Komponenten

### 3.1 PlatformBootstrap

- provisioniert zentrale Plattform-/Kundenstrukturen,
- DocumentationEngine implementiert keine eigene Kundenprovisionierung.

### 3.2 PipelineTemplates

Zentrale Orchestrierung bleibt beschlossen. Die konkrete Schnittstelle muss künftig sowohl:

- Collector-Actual-State-Artefakte,
- Bicep/IaC-Desired-State-Artefakte

reproduzierbar an die Engine übergeben können.

### 3.3 SecurityValidation

Fail-Closed-Prinzip ist beschlossen. Die genaue Abgrenzung zwischen vorgelagerter SecurityValidation und Engine-eigener Contract-/Consistency-Validation bleibt offen.

### 3.4 AzureInfrastructureCollector

Bekannter modularer Actual-State-Output:

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

P9 vereinheitlicht die produktive Azure-Relationship-Schnittstelle.

### 3.5 IaC-/Bicep-Repositories

Für IaC-verwaltete Workloads werden versionierte Bicep-Quellen als Desired-State-Input eingeplant.

Der physische Artefaktvertrag ist noch offen, fachliche Mindestanforderungen sind in `IAC_BICEP_INTERFACE.md` beschlossen.

### 3.6 OPNsenseDocumentation

Vorgelagerter Pfad:

```text
RAW -> Sanitize -> Secret Scan -> Validate -> Normalize -> Netzwerkmodell -> DocumentationEngine
```

Technischer Contract noch offen.

### 3.7 CUST-* Projekte

Die Engine erzeugt kundenspezifische Dokumentationsartefakte für `CUST-<Debitor>-<Name>`. `CUST-00000` bleibt Testkunde.

---

## 4. Noch offen

### 4.1 Contracts

- physischer gemeinsamer Input-Contract,
- Schema-/Modellversionierung,
- Bicep-Paket-/Toolchain-Contract finalisieren,
- Azure-P9-Contract finalisieren,
- OPNsense-Contract,
- SecurityValidation-Grenze,
- PipelineTemplates-Artefaktvertrag,
- CUST-Metadatenvertrag.

### 4.2 Core und interne Modelle

- technische Repräsentation/Serialisierung des Canonical Core,
- Core Validator,
- Semantic View Builder,
- technische Repräsentation/Serialisierung und Validierung des bereits fachlich beschlossenen Document View Models,
- Diagram View Model,
- Reconciliation Result Model,
- Fehler-/Loggingmodell.

### 4.3 Adapter

- Bicep Desired-State Adapter,
- Azure Actual-State Adapter Prototype,
- produktiver Azure Actual-State Adapter nach P9,
- OPNsense Adapter.

### 4.4 Rendering

- Template Engine,
- Markdown Renderer aus dem DVM,
- DOCX Renderer aus dem DVM,
- PDF-/HTML Renderer aus dem DVM,
- Diagrammformat,
- Layoutbibliothek,
- SVG-/sonstiger Diagrammrenderer,
- Icon-Katalog und Updateprozess,
- Diagrammvalidierung,
- Cross-Renderer-Contract-Tests für fachliche Parität.

### 4.5 Tests / Quality Gates

- Unit-/Contract-Tests,
- Bicep-Adapter-Tests,
- Actual-/Desired-Perspektivtests,
- DVM-Validierungs-/Golden-Master-Tests,
- Reconciliation-Tests,
- Diagrammtests,
- Cross-Renderer-Paritätstests für spätere DOCX/PDF-Renderer,
- No-Invention-Regression,
- E2E mit `CUST-00000`.

### 4.6 Knowledge Base / Publishing

Weiterhin offen: Azure DevOps/Wiki, SharePoint, Teams oder kombinierte Publishing-Lösung. Keine parallele manuelle Pflege identischer Inhalte.

---

## 5. Geplante nächste Workchunks

### DE-WC-01 – Canonical Infrastructure Core technisch implementieren

- vier Core-Objekte technisch repräsentieren,
- Graph-Perspektive implementieren,
- Validator/Registry/Determinismus/Evidence/Coverage,
- mindestens ein Actual- und ein Bicep-Desired-Fixture,
- keine P9-Abhängigkeit im globalen Core.

### DE-WC-02 – Semantic View Contracts

- fünf Views formal definieren,
- Perspektivvertrag pro View,
- keine View-/Rendererlogik im Core.

### DE-WC-02.1 – Document View Model Contract

**Fachlich beschlossen / technische Implementierung offen.**

- rendererunabhängige Dokumentstruktur,
- Sections/Paragraphs/Tables/Figures/Callouts als strukturierte Elemente,
- Actual-/Desired-Perspektive erhalten,
- DVM und Diagram View Model getrennt,
- Markdown als erster Renderer,
- DOCX/PDF als spätere Renderer desselben DVM,
- keine kanonische `Markdown -> DOCX/PDF`-Kette.

### DE-WC-03 – Initiale Source-/Provider-Adapter

- **Bicep Desired-State Adapter** als initialer produktnaher Adapter,
- Azure Actual-State Prototype gegen vorhandene Fixtures,
- produktiver Azure-Actual-State-Adapter weiterhin P9-gegated.

### DE-WC-04 – Desired/Actual Reconciliation Contract

- stabile technische Korrelation,
- matched / desiredMissing / actualUnmanaged / unresolved,
- Property Drift nur bei belastbarer Vergleichssemantik,
- keine Name-only-Korrelation.

---

## 6. Spätere Ausbaustufen

Geplant:

- DOCX Renderer aus dem Document View Model,
- PDF Renderer bzw. kontrollierter HTML/CSS->PDF-Pfad aus dem Document View Model.

Weitere mögliche Erweiterungen:

- Hyper-V/On-Premises,
- Switch/Layer-2,
- weitere Provider-/Iconsets,
- weitere Dokumentationstypen,
- Publishing nach späterer Architekturentscheidung.

Eine vollständige Property-Level-Drift-Engine kann schrittweise erweitert werden; der Reconciliation-Grundvertrag wird jedoch bereits in der initialen Architektur eingeplant.

---

## 7. Arbeitsregel

Neue technische Entscheidungen dürfen offene Punkte nur bewusst in „beschlossen“ überführen und müssen im kanonischen `IMPLEMENTATION_PLAN.md` und der Fachdokumentation konsistent gepflegt werden.

Nicht eindeutig rekonstruierbare frühere Entscheidungen werden nicht angenommen oder neu erfunden.
