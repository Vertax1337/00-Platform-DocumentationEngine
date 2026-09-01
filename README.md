# DocumentationEngine

Zentrale Engine zur Generierung standardisierter technischer Kundendokumentationen aus validierten Collector- und IaC-Daten.

> **Projektstatus:** Initialisierung / Architekturrahmen konsolidiert. Der providerunabhängige Canonical Infrastructure Core ist fachlich beschlossen; Bicep/IaC ist als initiale Desired-State-Quelle eingeplant; ein rendererunabhängiges Document View Model ist als kanonischer Dokumentvertrag beschlossen; die eigentliche fachliche Engine ist noch nicht implementiert.
>
> **Logische Einordnung im Gesamtprojekt:** `00-Platform / DocumentationEngine`
>
> **Aktuelles GitHub-Arbeitsrepository:** `Vertax1337/10-DocumentationEngine`

## Zweck

Die `DocumentationEngine` verarbeitet normalisierte und validierte technische Quellen und erzeugt daraus reproduzierbare technische Dokumentationsartefakte. Dazu gehören insbesondere:

- strukturierte technische Texte,
- Tabellen,
- Infrastrukturübersichten,
- Netzwerkdiagramme,
- Architekturdiagramme,
- später Desired-vs-Actual-/Drift-Sichten.

Der kanonische Dokumentvertrag ist ein **rendererunabhängiges Document View Model**. **Markdown** ist der erste produktive Renderer. **DOCX** und **PDF** werden später als weitere Renderer desselben Document View Models umgesetzt.

## Zwei technische Perspektiven

Die Engine unterscheidet von Beginn an zwischen tatsächlichem Iststand und versioniertem IaC-Sollstand:

```text
Actual State
  AzureInfrastructureCollector / weitere Collector
        |
        v
Actual-State Adapter
        |
        v
Canonical Graph [actual]

Desired State
  Bicep / IaC
        |
        v
Bicep Desired-State Adapter
        |
        v
Canonical Graph [desiredTemplate|desiredDeployment]
```

Bicep ersetzt den Collector nicht als Iststandsbeweis. Der Collector ersetzt Bicep nicht als IaC-Source-of-Truth.

Actual und Desired werden nicht stillschweigend zusammengeführt. Ein späterer Abgleich erfolgt über einen expliziten Reconciliation-Contract.

## Rolle in der Gesamtarchitektur

```text
Collector-/IaC-Quelle
       |
       v
Normalisierung / Build-Artefakt
       |
       v
Schema-/Security-/Contract-Validation
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
       +--> Diagram View Model --> Diagram Renderer
       |
       `--> Document View Model
                 |
                 +--> Markdown Renderer
                 +--> DOCX Renderer
                 `--> PDF/HTML Renderer
       |
       v
Dokumentationsartefakte
       |
       v
CUST-<Debitor>-<Name>
```

Die `DocumentationEngine` ist zentrale Plattformlogik. Sie gehört nicht in einzelne Collector-Repositories und nicht in kundenspezifische `CUST-*`-Projekte.

**Verbindliche Trennlinie Actual State:** Die DocumentationEngine inventarisiert Azure nicht selbst erneut. Für Azure konsumiert sie freigegebene, normalisierte Collector-Artefakte. Details: [`docs/COLLECTOR_INTERFACE.md`](docs/COLLECTOR_INTERFACE.md).

**Verbindliche Trennlinie Desired State:** Für IaC-verwaltete Workloads wird Bicep von Beginn an als First-Class-Desired-State-Quelle eingeplant. Details: [`docs/IAC_BICEP_INTERFACE.md`](docs/IAC_BICEP_INTERFACE.md).

**Verbindliche Trennlinie Dokumentoutput:** Markdown, DOCX und PDF sind Renderer desselben strukturierten Document View Models. Markdown ist nicht die kanonische Dokument-Source-of-Truth. Details: [`docs/architecture/DOCUMENT_VIEW_MODEL.md`](docs/architecture/DOCUMENT_VIEW_MODEL.md).

## Bereits beschlossene Randbedingungen

- Zentrale Einordnung unter `00-Platform`.
- Provisionierung über den bestehenden DEVOPS-/Platform-Bootstrap.
- Collector liefern den belegten Actual State.
- Bicep/IaC liefert den versionierten Desired State für IaC-verwaltete Workloads.
- Actual und Desired tragen eine explizite Graph-Perspektive.
- Vorgelagerte Schema-, Sicherheits-, Contract- und Quality-Gates arbeiten fail-closed.
- Zentrale Pipeline-Integration erfolgt über `PipelineTemplates`.
- Gemeinsame technische Komponenten werden erst bei belegter Wiederverwendung nach `SharedModules` ausgelagert.
- Das rendererunabhängige Document View Model ist der kanonische Dokumentvertrag.
- Markdown ist der erste produktive Renderer.
- PDF und DOCX folgen als weitere Renderer desselben Document View Models.
- Eine dauerhafte Produktionsarchitektur `Markdown -> DOCX/PDF` ist nicht vorgesehen.
- Azure-Diagramme verwenden offizielle Microsoft Azure Architecture Icons.
- Offizielle Hersteller-Icons dürfen nicht willkürlich verzerrt, gespiegelt, gedreht oder umgefärbt werden.
- Dokumentationsbuilds müssen reproduzierbar und automatisiert testbar werden.
- Fehlende Infrastruktur-Fakten dürfen nicht durch typische Referenzarchitekturen ergänzt werden.
- Generative Bildausgaben dürfen Stil-/Mockup-Zwecken dienen, aber nicht die technische Source of Truth eines Kundendiagramms bilden.
- Der Canonical Infrastructure Core besteht aus `InfrastructureNode`, `Relationship`, `EvidenceReference` und `CoverageRecord` plus Graph-Envelope/Perspektive.
- Jeder kanonische Node und jede kanonische Relationship benötigt Evidence.
- Renderer-/Layoutattribute gehören nicht in den Canonical Infrastructure Core.
- Desired-vs-Actual-Korrelation darf nicht allein über Namen erfolgen.

## Konsolidierter Prototypstand

Aus den ersten realen Azure-Collector-Prototypen wurden folgende Arbeitsstandards abgeleitet:

- **Technikerorientierung vor Inventar**.
- **Semantic View Builder** zwischen technischen Inputs und Renderer.
- **Progressive Disclosure** statt überladenem Gesamtbild.
- **Semantische Layoutzonen** statt freiem Graph-Autolayout.
- **Fünf Standard-Views:** Gesamtübersicht, Netzwerk & Connectivity, Workload & Deployment, Backup & Recovery, Security & Operations.
- **Backup aus Sicht der geschützten Ressource**.
- **Coverage statt Erfindung** bei nicht erhobenen Domänen.

Diese Erkenntnisse legen noch keine konkrete Rendertechnologie fest.

## Aktueller Core-Contract

```text
Collector / Bicep / weitere Quellen
        |
        v
Source / Provider Adapter
        |
        v
Canonical Infrastructure Model
  |- Graph Perspective
  |- InfrastructureNode
  |- Relationship
  |- EvidenceReference
  `- CoverageRecord
        |
        v
Semantic View Builder
        |
        +--> Diagram View Model
        |
        `--> Document View Model
               |
               +--> Markdown
               +--> DOCX
               `--> PDF
```

Details und DE-WC-01-Gate: [`docs/architecture/CANONICAL_MODEL.md`](docs/architecture/CANONICAL_MODEL.md).

Der Dokumentvertrag ist in [`docs/architecture/DOCUMENT_VIEW_MODEL.md`](docs/architecture/DOCUMENT_VIEW_MODEL.md) definiert.

Für Azure bleibt nur der **produktive Actual-State-Adapter** vom stabilisierten P9-Relationship-Schema des `AzureInfrastructureCollector` abhängig. Der Bicep Desired-State Adapter ist kein P9-Blocker.

## Bicep / IaC

Bicep wird ausdrücklich nicht erst später „dazugeklebt“.

Geplanter Mindestpfad:

```text
versioniertes main.bicep / Module
        |
        +--> immutable Git-/Build-Provenance
        +--> deterministische Compiler-/Artefakt-Provenance
        +--> sanitizter Deploymentkontext, falls benötigt
        |
        v
Bicep Desired-State Adapter
        |
        v
Canonical Graph [desiredTemplate|desiredDeployment]
```

Secret-Werte dürfen nicht in Canonical Graph, Evidence, Logs oder Dokumentartefakte übernommen werden.

## Noch ausdrücklich offen

- konkreter gemeinsamer Input-Contract und physische Paketstruktur,
- technische Repräsentation/Serialisierung und Versionierung des Canonical Core,
- konkrete technische Bicep-Parserstrategie,
- technische Repräsentation/Serialisierung und Versionierung des Document View Models,
- internes Diagram View Model,
- Reconciliation-/Diff-Result-Model,
- Template Engine,
- konkrete Markdown-Renderer-Implementierung,
- konkrete DOCX-Renderer-Technologie,
- konkrete PDF-/HTML-Renderer-Technologie,
- konkretes Diagrammformat,
- Layoutverfahren/Layoutbibliothek,
- CLI/API der Engine,
- konkrete Pipeline-Parameter und Artefaktübergabe,
- Abgrenzung eigener Validierungen gegenüber `SecurityValidation`,
- konkrete SharedModules-Nutzung,
- Knowledge-Base-/Publishing-Technologie.

## Angrenzende Repositories / Systeme

| Bereich | Beziehung zur DocumentationEngine |
|---|---|
| `DEVOPS / PlatformBootstrap` | Provisioniert zentrale Plattform- und Kundenrepositories. |
| `PipelineTemplates` | Zukünftige zentrale Orchestrierung für Collector- und IaC-Artefakte. |
| `SecurityValidation` | Vorgelagerte Sicherheits- und Validierungsgrenze. |
| `SharedModules` | Quelle für tatsächlich gemeinsam genutzte technische Komponenten. |
| `AzureInfrastructureCollector` | Liefert normalisierten Azure-Actual-State; Azure wird nicht erneut inventarisiert. |
| IaC-/Bicep-Repositories | Liefern versionierten Desired State für IaC-verwaltete Workloads. |
| `OPNsenseDocumentation` | Liefert normalisierten Netzwerk-/Firewall-Datenstand. |
| `CUST-*` | Kundenspezifischer Consumer/Zielort generierter Dokumentationsartefakte. |

## Dokumentation dieses Repositories

- [`docs/PROJECT_STATUS.md`](docs/PROJECT_STATUS.md) – konsolidierter Projektstand.
- [`docs/IMPLEMENTATION_PLAN.md`](docs/IMPLEMENTATION_PLAN.md) – kanonischer technischer Umsetzungsplan.
- [`docs/COLLECTOR_INTERFACE.md`](docs/COLLECTOR_INTERFACE.md) – Actual-State-Collector-Grenze.
- [`docs/IAC_BICEP_INTERFACE.md`](docs/IAC_BICEP_INTERFACE.md) – Bicep/IaC-Desired-State-Grenze.
- [`docs/architecture/CANONICAL_MODEL.md`](docs/architecture/CANONICAL_MODEL.md) – providerunabhängiger perspektivfähiger Core-Contract.
- [`docs/architecture/DOCUMENT_VIEW_MODEL.md`](docs/architecture/DOCUMENT_VIEW_MODEL.md) – rendererunabhängiger kanonischer Dokumentvertrag und Renderer-Grenze.
- [`docs/TECHNICIAN_DOCUMENTATION_STANDARD.md`](docs/TECHNICIAN_DOCUMENTATION_STANDARD.md) – technikerorientierter Dokumentationsstandard.
- [`docs/DIAGRAM_ENGINE_STANDARD.md`](docs/DIAGRAM_ENGINE_STANDARD.md) – Diagrammstandard und fünf Standard-Views.
- [`docs/PROTOTYPE_FINDINGS.md`](docs/PROTOTYPE_FINDINGS.md) – Prototyperkenntnisse und Fehlerregressionen.

## Externe Referenzen für Diagramme

- Azure Architecture Icons: https://learn.microsoft.com/en-us/azure/architecture/icons/
- Azure Well-Architected Framework – Architecture design diagrams: https://learn.microsoft.com/en-us/azure/well-architected/architect-role/design-diagrams
- Azure Resource Visualizer Skill: https://learn.microsoft.com/en-us/azure/developer/azure-skills/skills/azure-resource-visualizer
- Azure Landing Zone conceptual architecture: https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/

Microsofts Referenzarchitekturen dienen als Kommunikations-/Layoutreferenzen und nicht als Quelle für im Kunden-Iststand fehlende Ressourcen.

## Arbeitsprinzip

Der Umsetzungsplan ist der kanonische technische Stand der `DocumentationEngine`. Bereits gelöste Grundsatzfragen werden nicht neu geöffnet, sofern keine ausdrückliche Neubewertung angefordert wird.
