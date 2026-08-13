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

## Noch ausdrücklich offen

Zu folgenden Punkten ist noch **keine Technologie- oder Architekturentscheidung** getroffen worden:

- konkreter Input-Contract und physische Paket-/Dateistruktur,
- internes Dokument-, Infrastruktur- und Netzwerkmodell,
- Template Engine,
- Renderer,
- Diagrammformat und Diagrammrenderer,
- Layoutverfahren,
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
| `AzureInfrastructureCollector` | Liefert normalisierte Azure-Infrastrukturdaten. |
| `OPNsenseDocumentation` | Liefert nach RAW/Sanitize/Validate/Normalize einen normalisierten Netzwerk-/Firewall-Datenstand. |
| `CUST-*` | Kundenspezifischer Consumer bzw. Zielort generierter Dokumentationsartefakte. |

## Dokumentation dieses Repositories

- [`docs/PROJECT_STATUS.md`](docs/PROJECT_STATUS.md) – konsolidierter aktueller Projektstand.
- [`docs/IMPLEMENTATION_PLAN.md`](docs/IMPLEMENTATION_PLAN.md) – kanonischer technischer Umsetzungsplan dieses Unterprojekts.

## Arbeitsprinzip

Der Umsetzungsplan ist künftig der **kanonische technische Stand** der `DocumentationEngine`. Neue Architekturentscheidungen, technische Standards, offene Punkte, Implementierungsschritte und erreichte Umsetzungsstände werden dort fortlaufend gepflegt.

Bereits gelöste Grundsatzfragen aus dem Gesamtprojekt werden nicht neu geöffnet, sofern keine ausdrückliche Neubewertung angefordert wird.
