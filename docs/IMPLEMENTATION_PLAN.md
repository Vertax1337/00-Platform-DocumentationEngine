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

### DE-DEC-001 – Zentrale Plattformkomponente

**Status:** BESCHLOSSEN

Die `DocumentationEngine` ist zentrale Plattformlogik im Azure-DevOps-Projekt `00-Platform`.

Sie wird nicht pro Kunde und nicht pro Collector dupliziert.

### DE-DEC-002 – Repository-Provisionierung

**Status:** BESCHLOSSEN

Die Provisionierung des Repositories erfolgt über den bestehenden DEVOPS-/Platform-Bootstrap.

Die Bootstrap-Architektur wird in diesem Unterprojekt nicht neu entworfen.

### DE-DEC-003 – Trennung Collector und Dokumentationslogik

**Status:** BESCHLOSSEN

Collector übernehmen herstellerspezifische Erfassung, Parsing und Normalisierung.

Die `DocumentationEngine` verarbeitet die normalisierten Daten anschließend herstellerübergreifend in standardisierte Dokumentationsartefakte.

Collector sollen keine jeweils eigene vollständige Kundendokumentationsengine enthalten.

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

Ein vollständiger herstellerübergreifender Symbol- und Layoutstandard ist noch offen.

### DE-DEC-009 – Reproduzierbare Builds und automatisierte Tests

**Status:** BESCHLOSSEN

Die Engine muss reproduzierbare Pipeline-Builds und automatisierte Tests ermöglichen.

Konkrete Testarten und Quality Gates werden im Zuge der Implementierung festgelegt.

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

**Status:** LOGISCHE SCHNITTSTELLE BESCHLOSSEN / technischer Contract OFFEN

Erwarteter Input:

- normalisierte Azure-Infrastrukturdaten.

Bisheriger Arbeitsbegriff:

- `inventory.json`

Noch offen:

- eine Datei vs. modulares Artefaktpaket,
- Schema,
- Schema-Versionierung,
- Pflicht-/Optionalfelder,
- Identifikatoren und Referenzen zwischen Ressourcen.

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

- internes Dokumentmodell,
- internes Infrastrukturmodell,
- internes Netzwerkmodell,
- Referenzmodell zwischen Objekten,
- Modellversionierung.

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
- internes Diagrammmodell,
- Erzeugung und Einbettung der Diagramme in Markdown.

Es besteht aktuell keine globale Festlegung auf Mermaid, PlantUML, Graphviz oder ein anderes Format.

### DE-OPEN-006 – Symbol-, Icon- und Layoutstandard

Zu definieren:

- herstellerübergreifende Symbolregeln,
- Icon-Kataloge,
- Größen,
- Positionierung,
- Abstände,
- Container,
- Linien/Kanten,
- Richtungspfeile,
- Port-/Interface-Darstellung,
- VLAN-/Subnetz-/Segment-Darstellung,
- Legenden,
- Namenskonventionen.

Azure Architecture Icons bilden dabei eine bereits beschlossene Teilvorgabe.

### DE-OPEN-007 – Diagrammvalidierung

Zu definieren:

- technische Validierungsregeln,
- fehlende Referenzen,
- doppelte IDs,
- ungültige Kanten,
- nicht auflösbare Icons,
- Layout-/Rendering-Fehler,
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
- [ ] Repository-Namens-/Zielzuordnung beim späteren Azure-DevOps-Transfer prüfen, da das aktuelle GitHub-Repository `10-DocumentationEngine` heißt, die beschlossene logische Zielposition jedoch `00-Platform / DocumentationEngine` ist.

### Phase 1 – Contracts und Verantwortungsgrenzen

**Ziel:** Stabile Eingangs- und Integrationsschnittstellen definieren.

- [ ] aktuellen Output des `AzureInfrastructureCollector` gegen den geplanten Engine-Input abgleichen.
- [ ] aktuellen Output von `OPNsenseDocumentation` gegen den geplanten Engine-Input abgleichen.
- [ ] gemeinsames Input-Contract festlegen.
- [ ] Schema-Versionierungsstrategie festlegen.
- [ ] Verantwortungsgrenze zu `SecurityValidation` festlegen.
- [ ] Aufruf-/Artefaktvertrag mit `PipelineTemplates` festlegen.
- [ ] erforderliche Kundenmetadaten aus `CUST-*` festlegen.

**Decision Gate:** Erst danach internes Modell und Rendering-Technologien festlegen.

### Phase 2 – Internes Modell und Core Engine

**Ziel:** Normalisierte Eingaben deterministisch in ein internes Dokumentationsmodell überführen.

- [ ] internes Infrastruktur-/Netzwerk-/Dokumentmodell definieren.
- [ ] Modellvalidierung implementieren.
- [ ] Collector-Inputs in internes Modell transformieren.
- [ ] deterministische Sortierung und Referenzauflösung implementieren.
- [ ] Fehler- und Loggingmodell implementieren.

### Phase 3 – Markdown-Rendering

**Ziel:** Erste produktive standardisierte Kundendokumentation erzeugen.

- [ ] Template-Architektur auswählen und dokumentieren.
- [ ] Markdown-Renderer implementieren.
- [ ] Dokumentstruktur und Standardsektionen definieren.
- [ ] Tabellenstandard implementieren.
- [ ] Regeln für optionale/fehlende Daten definieren.
- [ ] erste Golden-Master-Dokumente aufbauen.

### Phase 4 – Diagrammstandard und Diagrammgenerierung

**Ziel:** Reproduzierbare Netzwerk-, Infrastruktur- und Architekturdiagramme erzeugen.

- [ ] Diagrammformat und Renderer auswählen.
- [ ] internes Diagrammmodell definieren.
- [ ] verbindlichen Symbol-/Icon-/Layoutstandard definieren.
- [ ] Microsoft Azure Architecture Icons integrieren.
- [ ] Netzwerkdiagramme generieren.
- [ ] Infrastrukturübersichten generieren.
- [ ] Architekturdiagramme generieren.
- [ ] Diagrammvalidierung implementieren.

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
- standardisierte Markdown-Ausgabe,
- mindestens die initial erforderlichen Infrastruktur-/Netzwerkdiagramme,
- verbindlicher Diagramm- und Iconstandard,
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
| DE-DEC-001 | Zentrale Plattformkomponente | BESCHLOSSEN | `00-Platform / DocumentationEngine` |
| DE-DEC-002 | Provisionierung | BESCHLOSSEN | DEVOPS-/Platform-Bootstrap |
| DE-DEC-003 | Collector-Trennung | BESCHLOSSEN | Collector normalisieren; Engine dokumentiert |
| DE-DEC-004 | Validation | BESCHLOSSEN | Fail Closed; Detailgrenze noch offen |
| DE-DEC-005 | Pipeline | BESCHLOSSEN | zentrale `PipelineTemplates` |
| DE-DEC-006 | Shared Code | BESCHLOSSEN | `SharedModules` bei echter Wiederverwendung |
| DE-DEC-007 | Initialer Output | BESCHLOSSEN | Markdown |
| DE-DEC-008 | Azure Icons | BESCHLOSSEN | offizielle Microsoft Azure Architecture Icons |
| DE-DEC-009 | Qualität | BESCHLOSSEN | reproduzierbare Builds + automatisierte Tests |
| DE-OPEN-001 | Input-Contract | OFFEN | – |
| DE-OPEN-002 | Internes Modell | OFFEN | – |
| DE-OPEN-003 | Template Engine | OFFEN | – |
| DE-OPEN-004 | Markdown-Renderer | OFFEN | – |
| DE-OPEN-005 | Diagrammformat/Renderer | OFFEN | – |
| DE-OPEN-006 | Symbol-/Layoutstandard | OFFEN | – |
| DE-OPEN-007 | Diagrammvalidierung | OFFEN | – |
| DE-OPEN-008 | Engine-Aufruf | OFFEN | – |
| DE-OPEN-009 | Teststrategie | OFFEN | – |
| DE-OPEN-010 | Knowledge Base / Publishing | OFFEN | – |

---

## 9. Änderungsprotokoll

### 2026-08-13

- initialen kanonischen Projektstand angelegt,
- bereits beschlossene Gesamtprojektentscheidungen übernommen,
- offene Technologieentscheidungen ausdrücklich nicht vorweggenommen,
- Abhängigkeiten zu Bootstrap, `PipelineTemplates`, `SecurityValidation`, `SharedModules`, `AzureInfrastructureCollector`, `OPNsenseDocumentation` und `CUST-*` konsolidiert,
- Knowledge-Base-/Publishing-Frage als offenen Plattformpunkt aufgenommen.
