# DocumentationEngine – konsolidierter Projektstand

Stand: 2026-08-13

Dieses Dokument konsolidiert den bisher im Gesamtprojekt beschlossenen und bekannten Stand der `DocumentationEngine`. Es trennt bewusst zwischen bereits beschlossenen, bereits implementierten, offenen und später möglichen Punkten.

## 1. Bereits beschlossen

### 1.1 Einordnung in die Plattform

Die `DocumentationEngine` ist als zentrales Repository im Azure-DevOps-Projekt `00-Platform` vorgesehen. Die Repository-Provisionierung gehört zum bestehenden DEVOPS-/Platform-Bootstrap.

Die Engine ist zentrale Plattformlogik und wird nicht pro Kunde oder pro Collector dupliziert.

Zielstruktur:

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

20-IaC
└── ...

CUST-<Debitor>-<Name>
├── CustomerConfiguration
├── Documentation
└── ggf. kundenspezifische RAW-Repositories
```

### 1.2 Verantwortungsgrenze Collector ↔ DocumentationEngine

Collector sind für die herstellerspezifische Erfassung, das Parsing und die Normalisierung von Daten zuständig.

```text
Herstellerspezifische Quelle
        |
        v
      Collect
        |
        v
      Parse
        |
        v
    Normalize
```

Die `DocumentationEngine` verarbeitet anschließend normalisierte Daten zu standardisierten technischen Dokumentationen und Visualisierungen.

```text
Normalisierte Daten
        |
        v
DocumentationEngine
   |      |      |
   |      |      +--> Architekturdiagramme
   |      +---------> Infrastrukturübersichten
   +----------------> Texte / Tabellen / Netzwerkdiagramme
```

Damit gilt insbesondere: Collector sollen nicht jeweils ihre eigene vollständige Kundendokumentationslogik implementieren.

### 1.3 Grundlegender Datenfluss

Der bisher festgelegte Zielpfad lautet:

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
Schema Validation
       |
       v
Security Validation
       |
       v
Quality Gates
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

Validierungs- und Sicherheitsfehler müssen den Prozess nach dem Fail-Closed-Prinzip stoppen.

### 1.4 Markdown-first

Initiales Ausgabeformat ist Markdown.

PDF und DOCX sind als spätere Ausgabeformate vorgesehen und dürfen die erste produktive Implementierung nicht blockieren.

### 1.5 Diagramme und Hersteller-Icons

Für Azure-Diagramme sollen die offiziellen **Microsoft Azure Architecture Icons** verwendet werden.

Offizielle Hersteller-Icons sollen nicht willkürlich verändert werden. Insbesondere sollen sie nicht:

- verzerrt,
- gespiegelt,
- gedreht,
- willkürlich umgefärbt

werden.

Ein vollständiger herstellerübergreifender Symbol-, Icon- und Layoutstandard ist noch nicht beschlossen.

### 1.6 Reproduzierbarkeit und Qualität

Die `DocumentationEngine` muss langfristig:

- reproduzierbare Builds liefern,
- automatisiert testbar sein,
- technische Quality Gates unterstützen,
- Dokumentationsstandards versionierbar und wartbar machen.

## 2. Bereits implementiert

Nach dem derzeit bekannten Stand sind vor allem Plattform- und Repository-Grundlagen vorhanden bzw. über den Bootstrap vorgesehen.

| Bereich | Stand |
|---|---|
| Zentrale Azure-DevOps-Projektstruktur | vorhanden / provisionierbar |
| `00-Platform` | vorgesehen / vorhanden im Plattformmodell |
| `DocumentationEngine` Repository-Provisionierung | im DEVOPS-/Platform-Bootstrap vorgesehen |
| `PipelineTemplates` Repository | strukturell vorgesehen bzw. angelegt |
| `SecurityValidation` Repository | strukturell vorgesehen bzw. angelegt |
| `SharedModules` Repository | strukturell vorgesehen bzw. angelegt |
| `AzureInfrastructureCollector` | als zentrales Collector-Repository vorgesehen bzw. angelegt |
| `OPNsenseDocumentation` | als zentrales Collector-/Dokumentationsvorstufen-Repository vorgesehen bzw. angelegt |
| `CUST-*` Provisionierungsmodell | im Bootstrap vorgesehen |
| Eigentliche fachliche `DocumentationEngine` | **noch nicht implementiert** |
| Produktiver End-to-End-Dokumentationsbuild | **noch nicht implementiert** |

Dieses GitHub-Repository enthält ab jetzt den konsolidierten Ausgangsstand und den kanonischen Umsetzungsplan. Der aktuelle GitHub-Repositoryname `10-DocumentationEngine` ändert die beschlossene logische Zielzuordnung `00-Platform / DocumentationEngine` nicht.

## 3. Anforderungen und Schnittstellen aus angrenzenden Teilprojekten

### 3.1 DEVOPS / PlatformBootstrap

Bereits vorgegeben:

- Die `DocumentationEngine` wird zentral provisioniert.
- Kunden werden als `CUST-<Debitor>-<Name>` onboardet.
- Kundenspezifische Repositories und bei Bedarf RAW-Repositories werden über den Bootstrap bereitgestellt.
- Technische Prozesse und HowTos dürfen voraussetzen bzw. prüfen, dass ein Kunde vor Nutzung der Dokumentationsintegration korrekt onboardet wurde.

Noch nicht Bestandteil dieses Unterprojekts ist die erneute Planung des Bootstrap-Verfahrens.

### 3.2 PipelineTemplates

Beschlossen ist eine zentrale Pipeline-Orchestrierung. Kundenspezifische Pipelines sollen die gemeinsame Logik nicht selbst duplizieren.

Zielbild:

```text
Collect
  |
  v
Sanitize / Security
  |
  v
Normalize
  |
  v
Schema Validation / Quality Gates
  |
  v
DocumentationEngine
  |
  v
Publish
```

Noch offen sind für die `DocumentationEngine`:

- konkrete Template-Namen,
- Parameter,
- Artefaktübergabe,
- Working Directories,
- Input-/Output-Pfade,
- Exitcodes,
- Fehlervertrag,
- Versionierung der Pipeline-Schnittstelle.

### 3.3 SecurityValidation

Bereits vorgegeben sind Sicherheitsprinzipien wie:

- Read-only Collection,
- Sanitize,
- Secret Detection,
- Schema Validation,
- Security Validation,
- Fail Closed.

Noch offen ist die genaue Verantwortungsgrenze zwischen zentraler `SecurityValidation` und den eigenen technischen Konsistenzprüfungen der `DocumentationEngine`.

### 3.4 SharedModules

`SharedModules` ist der vorgesehene Ort für wiederverwendbare technische Komponenten, die von mehreren Plattformkomponenten genutzt werden.

Noch offen ist, welche konkreten Funktionen der `DocumentationEngine` dort liegen sollen. Es wird aktuell nichts vorschnell ausgelagert.

### 3.5 AzureInfrastructureCollector

Der Collector liefert normalisierte strukturierte Azure-Infrastrukturdaten als Input für die Engine.

Als bisheriger Arbeitsbegriff wurde unter anderem `inventory.json` verwendet. Noch nicht abschließend entschieden ist, ob der technische Input später aus einer einzelnen Datei, mehreren modularen Dateien oder einem anderen definierten Artefaktpaket besteht.

Damit ist fest:

- der logische Input sind normalisierte Azure-Daten,
- der konkrete technische Input-Contract ist noch offen.

### 3.6 OPNsenseDocumentation

Für OPNsense ist ein vorgelagerter Verarbeitungspfad vorgesehen:

```text
RAW
 |
 v
Sanitize
 |
 v
Secret Scan
 |
 v
Validate
 |
 v
Normalize
 |
 v
Netzwerkmodell
 |
 v
DocumentationEngine
```

Frühere Arbeitsstände erwähnten teilweise ein Artefakt wie `network.mmd`. Dies ist **keine belastbare globale Entscheidung**, dass die DocumentationEngine generell Mermaid verwenden muss.

Das konkrete Diagrammformat und der Diagrammrenderer bleiben offen.

### 3.7 CUST-* Projekte

Das Kundenmodell folgt:

```text
CUST-<Debitor>-<Name>
```

Die interne Debitor-/Kundennummer ist die führende stabile ID.

Die `DocumentationEngine` erzeugt kundenspezifische Artefakte für diese Projekte. Die zentrale Engine selbst wird nicht in den Kundenprojekten implementiert.

Vorgesehene Bereiche sind insbesondere:

- `CustomerConfiguration`
- `Documentation`
- ggf. getrennte kundenspezifische RAW-Repositories

`CUST-00000` ist als Testkunde vorgesehen. Ein vollständiger End-to-End-Ablauf ist nach dem bekannten Stand noch nicht implementiert.

## 4. Noch offen

### 4.1 Input und Contracts

- konkreter Input-Contract,
- physische Datei-/Paketstruktur,
- Schema-Versionierung,
- Pflicht- und Optionalfelder,
- Umgang mit mehreren Collector-Quellen,
- Zusammenführung von Azure-, Firewall- und später weiteren Quellen.

### 4.2 Interne Modelle

- internes Dokumentmodell,
- internes Infrastrukturmodell,
- internes Netzwerkmodell,
- Verhältnis Collector-Modell ↔ Dokumentmodell,
- Versionierung der internen Modelle.

### 4.3 Template und Rendering

- Template Engine,
- Template-Struktur,
- Renderer,
- Markdown-Erzeugung,
- Conditional Sections,
- Verhalten bei optionalen oder fehlenden Daten,
- Tabellenstandard.

### 4.4 Diagramme

- Diagrammformat,
- Diagrammrenderer,
- internes Diagrammmodell,
- Layoutverfahren,
- herstellerübergreifender Symbolstandard,
- Positionierungs- und Abstandsregeln,
- Linien-/Kantenstandard,
- Port-/Interface-Darstellung,
- VLAN-Darstellung,
- Netzwerksegment-Darstellung,
- Legendenstandard,
- Architektur- und Infrastrukturdiagrammstandards,
- Diagrammvalidierung.

### 4.5 Integration

- CLI oder andere technische Aufrufschnittstelle,
- Pipeline-Aufruf,
- Input-/Output-Artefakte,
- Exitcodes,
- Logging,
- Fehlerformat,
- konkrete `PipelineTemplates`-Schnittstelle,
- konkrete `SecurityValidation`-Schnittstelle,
- konkrete `SharedModules`-Nutzung.

### 4.6 Tests und Quality Gates

- Unit Tests,
- Schema-/Contract-Tests,
- Snapshot-/Golden-Master-Tests,
- Diagrammtests,
- Dokumenttests,
- deterministische Ausgabe,
- reproduzierbarer Build,
- Quality-Gate-Regeln.

## 5. OPEN – Knowledge Base / Publishing

Noch nicht entschieden ist, wie zentral gepflegte technische HowTos und Runbooks für Techniker bereitgestellt werden.

Bereits als Anforderung festgehalten:

- HowTos und Runbooks sollen eine versionierte Source of Truth besitzen.
- Identische Inhalte sollen nicht manuell parallel in mehreren Zielsystemen gepflegt werden.
- Technische HowTos müssen Voraussetzungen und Abhängigkeiten aus der Plattformarchitektur abbilden können.

### Realweltbeispiel OPNsense

Ein Techniker richtet eine neue OPNsense-Firewall ein. Das zentrale Ersteinrichtungs-HowTo muss ihn darauf hinweisen, dass der Kunde zunächst über den DEVOPS-Bootstrap als `CUST-*` onboardet sein muss. Nur so sind die erforderlichen Kunden-/Firewall-Repositories vorhanden und die Backup-/Dokumentationsintegration kann korrekt eingerichtet werden.

Noch offen ist, wo Techniker die freigegebene Fassung konsumieren:

- Azure DevOps / Wiki,
- SharePoint,
- Teams,
- kombinierte Publishing-Lösung.

Ebenfalls offen sind:

- Ort und Format der Source of Truth,
- Authoring-Modell,
- Freigabe-/Release-Modell,
- Publishing-Automatisierung,
- Versionierung veröffentlichter Fassungen,
- genaue Abgrenzung zwischen generierter Kundendokumentation und zentraler Techniker-Knowledge-Base.

Diese offenen Punkte sind **keine** bereits getroffenen Technologieentscheidungen.

## 6. Mögliche spätere Erweiterungen

Bereits als spätere Ausbaustufen erkennbar:

- PDF-Ausgabe,
- DOCX-Ausgabe,
- weitere Collector-Typen,
- On-Premises-/Hyper-V-Datenquellen,
- Switch-/Layer-2-Collector-Daten für physische Topologien,
- weitere Hersteller-Iconsets,
- weitere Dokumentationstypen,
- weitere Architekturdiagrammtypen,
- ggf. automatisiertes Publishing in Knowledge-Base-Zielsysteme nach späterer Architekturentscheidung.

## 7. Arbeitsregel

Neue technische Entscheidungen dürfen offene Punkte nur dann in „beschlossen“ überführen, wenn die Entscheidung in diesem Unterprojekt bewusst getroffen und im kanonischen Umsetzungsplan dokumentiert wurde.

Nicht eindeutig rekonstruierbare frühere Entscheidungen werden nicht angenommen oder neu erfunden.
