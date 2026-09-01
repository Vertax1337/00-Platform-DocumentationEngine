# DocumentationEngine – Document View Model

Stand: 2026-09-01
Status: **BESCHLOSSENER fachlicher Contract; technische Implementierung offen**

## 1. Zweck

Das `Document View Model` (DVM) ist der kanonische, rendererunabhängige Dokumentvertrag der `DocumentationEngine`.

Es trennt fachliche Dokumentstruktur und technische Fakten von konkreten Ausgabeformaten.

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

Markdown ist damit **nicht** die Source of Truth des Dokuments. Markdown ist der erste produktive Renderer desselben strukturierten Document View Models.

DOCX und PDF werden später als weitere Renderer aus demselben DVM erzeugt. Eine dauerhafte Architektur `Markdown -> DOCX/PDF` ist ausdrücklich nicht vorgesehen.

---

## 2. Verbindliche Prinzipien

### DE-DOC-001 – Rendererunabhängige Source of Truth

Der fachliche Dokumentzustand wird im DVM abgebildet und darf keine Markdown-, DOCX-/OOXML- oder PDF-spezifischen Strukturen als kanonische Semantik voraussetzen.

Renderer dürfen das DVM formatgerecht darstellen, aber keine neuen technischen Fakten erzeugen.

### DE-DOC-002 – Markdown-first bedeutet erster Renderer

Markdown bleibt das erste produktive Ausgabeformat, weil es:

- leicht diffbar,
- Git-tauglich,
- deterministisch,
- gut für Golden-Master-Tests geeignet,
- technisch einfach zu inspizieren

ist.

Diese Priorisierung macht Markdown nicht zum internen Dokumentmodell.

### DE-DOC-003 – DOCX-/PDF-Fähigkeit wird im DVM berücksichtigt

Das DVM muss bereits vor Implementierung der DOCX-/PDF-Renderer ausreichend strukturiert sein, um typische professionelle Dokumentanforderungen ohne Rückwärtsanalyse von Markdown abzubilden.

Dazu gehören insbesondere:

- Titel-/Dokumentmetadaten,
- Kapitel-/Unterkapitelhierarchie,
- Absätze,
- Tabellen,
- Abbildungen/Diagramme,
- Bildunterschriften,
- Hinweise/Callouts,
- kontrollierte Seiten-/Abschnittsumbrüche,
- optionale Layout-Hinweise wie Querformat für breite Tabellen,
- Inhaltsverzeichnis-fähige Überschriftenstruktur,
- Actual-/Desired-/Reconciliation-Perspektive,
- Evidence-/Coverage-Verweise, soweit für die Darstellung erforderlich.

Layout-Hinweise bleiben semantisch und formatneutral. OOXML-IDs, Word-Relationship-IDs, CSS-Klassen oder PDF-spezifische Objekte gehören nicht in das DVM.

### DE-DOC-004 – Ein fachlicher Inhalt, mehrere Renderer

Bei identischem validierten DVM müssen die verschiedenen Renderer denselben fachlichen Inhalt wiedergeben.

```text
Document View Model
   |- TechnicalDocumentation.md
   |- TechnicalDocumentation.docx
   `- TechnicalDocumentation.pdf
```

Formatbedingte Unterschiede in Pagination, Typografie oder Tabellenumbruch sind zulässig. Unterschiede in Fakten, Perspektive, Tabellenzeilen, Diagrammzuordnung oder Coverage sind nicht zulässig, sofern das Zielformat die Information darstellen kann.

### DE-DOC-005 – Renderer sind Darstellungsschichten

Renderer sind verantwortlich für:

- formatspezifische Syntax/Serialisierung,
- Typografie und Styles,
- Tabellenlayout,
- Bild-/Diagrammeinbettung,
- Seiten-/Abschnittslayout,
- Inhaltsverzeichnis-Ausprägung,
- Header/Footer und Branding, soweit für das Zielformat vorgesehen.

Renderer sind nicht verantwortlich für:

- Infrastrukturinterpretation,
- Provider-Mapping,
- Erfindung fehlender Fakten,
- Actual-/Desired-Korrelation,
- semantische Workload-Erkennung.

---

## 3. Initiale DVM-Struktur

Die konkrete technische Repräsentation bleibt noch offen. Fachlich muss das Modell mindestens folgende Struktur abbilden können:

```text
Document
|- metadata
|- titlePage
|- sections[]
|  |- heading
|  |- blocks[]
|  |  |- Paragraph
|  |  |- Table
|  |  |- Figure
|  |  |- Callout
|  |  `- List
|  `- sections[]
`- renderHints
```

### 3.1 Document Metadata

Mindestens konzeptionell vorgesehen:

- Dokumenttyp,
- Kunde/Scope,
- Erfassungs-/Buildstand,
- Source-Perspektive (`actual`, `desiredTemplate`, `desiredDeployment`, später `reconciled`),
- Engine-/Schema-Version,
- relevante Snapshot-/Commit-/Build-Provenance.

### 3.2 Sections und Headings

Überschriften sind strukturierte Objekte und nicht bereits gerenderte Markdown-Strings.

Sie müssen eine stabile Hierarchie und eine deterministische Reihenfolge besitzen.

Damit können Markdown, Word und PDF dieselbe Kapitelstruktur und dasselbe Inhaltsverzeichnis ableiten.

### 3.3 Paragraph

Ein Absatz enthält fachlichen Text und bei Bedarf maschinenlesbare Metadaten zur Aussageklasse, beispielsweise:

- Fakt,
- deterministische Ableitung,
- Desired-State-Aussage,
- Coverage-/Hinweistext.

Das DVM darf dafür keine Markdown-Markupzeichen oder Word-Run-XML voraussetzen.

### 3.4 Table

Tabellen werden strukturiert repräsentiert und nicht als Markdown-Text gespeichert.

Fachlich mindestens vorgesehen:

```text
Table
|- id
|- caption
|- columns[]
|- rows[]
|- layoutHints
`- evidence/coverage context (falls erforderlich)
```

Mögliche formatneutrale Layout-Hinweise:

- breite Tabelle,
- bevorzugtes Querformat,
- Headerzeile wiederholen,
- Spaltenprioritäten,
- optionaler Seitenumbruch vor der Tabelle.

Die exakte Word-Spaltenbreite oder Markdown-Pipe-Syntax gehört in den jeweiligen Renderer.

### 3.5 Figure

Diagramme und andere Abbildungen werden als strukturierte Referenzen eingebunden:

```text
Figure
|- artifactId
|- title
|- caption
|- altText
|- viewType
|- perspective
`- placementHints
```

Die Diagramm-Source-of-Truth bleibt das validierte Diagram View Model bzw. das daraus erzeugte Artefakt. Das DVM referenziert das Diagramm und erfindet keine Diagramminhalte.

### 3.6 Callout

Hinweise wie Coverage, Warnungen oder nicht aufgelöste Desired-State-Conditions können als strukturierte Callouts modelliert werden.

Beispiele für fachliche Kategorien:

- `information`,
- `coverage`,
- `warning`,
- `unresolved`.

Die visuelle Ausprägung entscheidet der Renderer.

### 3.7 Render Hints

Formatneutrale Render-Hinweise sind zulässig, wenn sie eine dokumentfachliche Absicht ausdrücken.

Beispiele:

- `pageBreakBefore`,
- `keepWithNext`,
- `preferredOrientation = landscape`,
- `avoidSplit`,
- `preferredWidth = full`.

Nicht zulässig sind rendererinterne IDs oder Formatcode.

---

## 4. Beziehung zum Diagram View Model

Document View Model und Diagram View Model bleiben getrennte Verträge.

```text
Canonical Graph
      |
      +--> Semantic Document View
      |        -> Document View Model
      |
      `--> Semantic Diagram View
               -> Diagram View Model
```

Ein `Figure`-Block im DVM referenziert ein versioniertes/validiertes Diagrammartefakt oder Diagram View Model.

Dadurch kann dasselbe Diagramm in Markdown, DOCX und PDF wiederverwendet werden.

---

## 5. Actual / Desired / Reconciliation

Die Source-Perspektive muss im DVM erhalten bleiben.

Ein Dokument oder Abschnitt darf Desired State nicht als gemessenen Actual State darstellen.

Mindestens zu berücksichtigen:

```text
actual
  -> gemessener/erhobener Iststand

desiredTemplate
  -> deklarierter Bicep-/IaC-Sollstand ohne vollständig aufgelösten Deploymentkontext

desiredDeployment
  -> IaC-Sollstand mit sanitiztem, aufgelöstem Deploymentkontext

reconciled
  -> späterer expliziter Desired-/Actual-Abgleich
```

Die konkrete Reconciliation-Semantik bleibt im DE-WC-04-Contract.

---

## 6. Determinismus und Tests

Bei identischem DVM-Input muss jeder Renderer reproduzierbare fachliche Ausgabe erzeugen.

Für den DVM-Contract sind mindestens folgende Tests vorzusehen:

- stabile Kapitel-/Blockreihenfolge,
- strukturierte Tabellen statt gerenderter Markdown-Strings,
- Figure-Referenzen auf existierende Diagrammartefakte,
- Perspektivkennzeichnung bleibt erhalten,
- fehlende optionale Daten werden deterministisch behandelt,
- keine renderer-spezifischen Strukturen im DVM,
- Golden-Master-Test des Markdown-Renderers,
- später Cross-Renderer-Contract-Tests für fachliche Parität Markdown/DOCX/PDF.

Binäre Byte-Gleichheit von DOCX/PDF ist nicht automatisch erforderlich, sofern formatspezifische Metadaten dies verhindern. Der fachliche Renderer-Contract und relevante deterministische Artefaktbestandteile müssen jedoch prüfbar bleiben.

---

## 7. Renderer-Reihenfolge

Verbindliche Implementierungsreihenfolge:

```text
1. Document View Model fachlich definieren
2. technische DVM-Repräsentation + Validierung
3. Markdown Renderer
4. Golden-Master-/Contract-Tests
5. DOCX Renderer
6. PDF Renderer bzw. kontrollierter HTML/CSS->PDF-Renderer
```

Die genaue Technologie für DOCX und PDF bleibt offen und wird erst anhand des stabilen DVM bewertet.

Ein früher Prototyp darf Markdown als Zwischenformat für einen PDF-Test verwenden. Dies begründet jedoch keinen kanonischen Produktionsvertrag `Markdown -> PDF/DOCX`.

---

## 8. DE-WC-02.1 – Document View Model Contract

**Status:** BESCHLOSSEN / technische Implementierung offen

### Fachliches Gate

Der Workchunk ist fachlich spezifiziert, wenn:

- DVM als kanonischer Dokumentvertrag dokumentiert ist,
- Markdown eindeutig als erster Renderer und nicht als Source of Truth festgelegt ist,
- DOCX/PDF als spätere Renderer desselben DVM eingeordnet sind,
- Tabellen/Figures/Sections nicht als vorgerenderte Markdown-Fragmente modelliert werden,
- Actual-/Desired-Perspektiven im Dokumentvertrag erhalten bleiben,
- DVM und Diagram View Model sauber getrennt sind,
- Fachdokumentation und kanonischer Umsetzungsplan denselben Status enthalten.

### Technisches Gate

`IMPLEMENTIERT` darf erst gesetzt werden, wenn:

- technische DVM-Repräsentation versioniert ist,
- DVM-Validierung vorhanden ist,
- deterministische Sortierung/Struktur nachgewiesen ist,
- mindestens ein vollständiges Dokument-Fixture existiert,
- der Markdown-Renderer ausschließlich aus dem DVM rendert,
- Golden-Master-/Contract-Tests grün sind.

Bis dahin ist der DVM-Contract **BESCHLOSSEN**, aber noch nicht technisch **IMPLEMENTIERT**.
