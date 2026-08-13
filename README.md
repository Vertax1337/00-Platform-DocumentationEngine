# DocumentationEngine

Zentrale Engine zur Generierung standardisierter technischer Kundendokumentationen aus normalisierten Collector-Daten.

> **Projektstatus:** Initialisierung / Architekturrahmen konsolidiert. Die eigentliche fachliche Engine ist noch nicht implementiert.
>
> **Logische Einordnung im Gesamtprojekt:** `00-Platform / DocumentationEngine`
>
> **Aktuelles GitHub-Arbeitsrepository:** `Vertax1337/10-DocumentationEngine`

## Zweck

Die `DocumentationEngine` verarbeitet normalisierte und validierte Daten aus zentralen Collector-Repositories und erzeugt daraus reproduzierbare technische Dokumentationsartefakte. Dazu gehören insbesondere:

- strukturierte technische Texte,
- Tabellen,
- Infrastrukturübersichten,
- Netzwerkdiagramme,
- Architekturdiagramme.

Initiales Ausgabeformat ist **Markdown**. **PDF** und **DOCX** sind als spätere Erweiterungen vorgesehen.

## Rolle in der Gesamtarchitektur

```text
Read-only Quelle
       |
       v
    Collector
       |
       v
Normalisierung
       |
       v
Schema-/Security-Validation
       |
       v
DocumentationEngine
       |
       v
Dokumentationsartefakte
       |
       v
CUST-<Debitor>-<Name>
```

Die `DocumentationEngine` ist zentrale Plattformlogik. Sie gehört nicht in einzelne Collector-Repositories und nicht in kundenspezifische `CUST-*`-Projekte.

Collector sind für Erfassung, Parsing und Normalisierung herstellerspezifischer Daten zuständig. Die `DocumentationEngine` übernimmt die herstellerübergreifende Verarbeitung in standardisierte Dokumentation und Visualisierungen.

**Verbindliche Trennlinie:** Die DocumentationEngine soll Azure nicht selbst erneut inventarisieren. Für Azure konsumiert sie freigegebene, normalisierte Collector-Artefakte. Details: [`docs/COLLECTOR_INTERFACE.md`](docs/COLLECTOR_INTERFACE.md).

## Bereits beschlossene Randbedingungen

- Zentrale Einordnung unter `00-Platform`.
- Provisionierung des Repositories erfolgt über den bestehenden DEVOPS-/Platform-Bootstrap.
- Normalisierte Collector-Daten bilden den Input der Engine.
- Vorgelagerte Schema-, Sicherheits- und Quality-Gates arbeiten nach dem Fail-Closed-Prinzip.
- Zentrale Pipeline-Integration erfolgt über `PipelineTemplates`.
- Wiederverwendbare gemeinsame technische Komponenten sollen bei Bedarf aus `SharedModules` genutzt werden.
- Initialer Dokumentationsoutput ist Markdown.
- PDF- und DOCX-Ausgabe folgen später.
- Azure-Diagramme sollen offizielle **Microsoft Azure Architecture Icons** verwenden.
- Offizielle Hersteller-Icons dürfen nicht willkürlich verzerrt, gespiegelt, gedreht oder umgefärbt werden.
- Dokumentationsbuilds müssen reproduzierbar und automatisiert testbar werden.
- Fehlende Infrastruktur-Fakten dürfen nicht durch typische Referenzarchitekturen ergänzt werden.
- Generative Bildausgaben dürfen Stil-/Mockup-Zwecken dienen, aber nicht die technische Source of Truth eines Kundendiagramms bilden.

## Konsolidierter Prototypstand

Aus den ersten mit realen Azure-Collector-Daten erzeugten Dokumentations- und Diagrammprototypen wurden zusätzliche Arbeitsstandards abgeleitet:

- **Technikerorientierung vor Inventar:** Einstieg über Architektur, Workloads und Betriebszusammenhänge; vollständige Ressourcenlisten nachgelagert.
- **Semantic View Builder:** Zwischen Collector-Daten und Renderer wird eine semantische Sicht benötigt; direkte JSON-zu-Diagramm-Abbildung reicht nicht aus.
- **Progressive Disclosure:** High-Level-Übersicht und fokussierte Detailviews statt eines überladenen Gesamtbilds.
- **Semantische Layoutzonen:** Positionen sollen Bedeutung transportieren; freies Graph-Autolayout ist für die High-Level-Technikersicht nicht ausreichend.
- **Fünf Standard-Views als Prototyprahmen:** Gesamtübersicht, Netzwerk & Connectivity, Workload & Deployment, Backup & Recovery, Security & Operations.
- **Backup aus Sicht der geschützten Ressource:** primäre Frage ist „Was ist geschützt und wodurch?“.
- **Coverage statt Erfindung:** Noch nicht erhobene Security-/Operations-Domänen werden als nicht erhoben gekennzeichnet und nicht mit typischen Azure-Diensten aufgefüllt.

Diese Erkenntnisse legen **noch keine konkrete Rendertechnologie** fest.

## Noch ausdrücklich offen

Zu folgenden Punkten ist noch **keine finale Technologie- oder Architekturentscheidung** getroffen worden:

- konkreter Input-Contract und physische Paket-/Dateistruktur,
- internes Dokument-, Infrastruktur- und Netzwerkmodell,
- Template Engine,
- konkrete Renderer-Technologie,
- konkretes Diagrammformat,
- konkretes Layoutverfahren / Layoutbibliothek,
- CLI/API der Engine,
- konkrete Pipeline-Parameter und Artefaktübergabe,
- konkrete Abgrenzung eigener Validierungen gegenüber `SecurityValidation`,
- konkrete Nutzung bzw. Auslagerung in `SharedModules`,
- PDF-/DOCX-Exportwerkzeuge,
- Knowledge-Base-/Publishing-Technologie.

Diese Punkte werden erst nach Konsolidierung der bestehenden Schnittstellen bewertet und anschließend als explizite Architekturentscheidungen dokumentiert.

## Angrenzende Repositories / Systeme

| Bereich | Beziehung zur DocumentationEngine |
|---|---|
| `DEVOPS / PlatformBootstrap` | Provisioniert zentrale Plattform- und Kundenrepositories. |
| `PipelineTemplates` | Zukünftige zentrale Pipeline-Orchestrierung und standardisierter Aufruf der Engine. |
| `SecurityValidation` | Vorgelagerte Sicherheits- und Validierungsgrenze; genaue Verantwortungsabgrenzung noch offen. |
| `SharedModules` | Quelle für künftig gemeinsam nutzbare technische Komponenten. |
| `AzureInfrastructureCollector` | Liefert normalisierte Azure-Infrastrukturdaten; Azure wird durch die DocumentationEngine nicht erneut inventarisiert. |
| `OPNsenseDocumentation` | Liefert nach RAW/Sanitize/Validate/Normalize einen normalisierten Netzwerk-/Firewall-Datenstand. |
| `CUST-*` | Kundenspezifischer Consumer bzw. Zielort generierter Dokumentationsartefakte. |

## Dokumentation dieses Repositories

- [`docs/PROJECT_STATUS.md`](docs/PROJECT_STATUS.md) – konsolidierter aktueller Projektstand.
- [`docs/IMPLEMENTATION_PLAN.md`](docs/IMPLEMENTATION_PLAN.md) – kanonischer technischer Umsetzungsplan dieses Unterprojekts.
- [`docs/COLLECTOR_INTERFACE.md`](docs/COLLECTOR_INTERFACE.md) – Verantwortungsgrenze und bekannte Collector-Schnittstelle.
- [`docs/TECHNICIAN_DOCUMENTATION_STANDARD.md`](docs/TECHNICIAN_DOCUMENTATION_STANDARD.md) – Zielstruktur für technikerorientierte Kundendokumentation.
- [`docs/DIAGRAM_ENGINE_STANDARD.md`](docs/DIAGRAM_ENGINE_STANDARD.md) – konsolidierte Diagrammregeln, Microsoft-Referenzen und fünf Standard-Views.
- [`docs/PROTOTYPE_FINDINGS.md`](docs/PROTOTYPE_FINDINGS.md) – positive und negative Erkenntnisse aus den bisherigen Prototypen.

## Externe Referenzen für Diagramme

- Azure Architecture Icons: https://learn.microsoft.com/en-us/azure/architecture/icons/
- Azure Well-Architected Framework – Architecture design diagrams: https://learn.microsoft.com/en-us/azure/well-architected/architect-role/design-diagrams
- Azure Resource Visualizer Skill: https://learn.microsoft.com/en-us/azure/developer/azure-skills/skills/azure-resource-visualizer
- Azure Landing Zone conceptual architecture: https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/

Microsofts Referenzarchitekturen dienen als Kommunikations-/Layoutreferenzen und nicht als Quelle für im Kunden-Iststand fehlende Ressourcen.

## Arbeitsprinzip

Der Umsetzungsplan ist künftig der **kanonische technische Stand** der `DocumentationEngine`. Neue Architekturentscheidungen, technische Standards, offene Punkte, Implementierungsschritte und erreichte Umsetzungsstände werden dort fortlaufend gepflegt.

Bereits gelöste Grundsatzfragen aus dem Gesamtprojekt werden nicht neu geöffnet, sofern keine ausdrückliche Neubewertung angefordert wird.
