# DocumentationEngine – IaC/Bicep Desired-State Interface

Stand: 2026-09-01
Status: **BESCHLOSSENER Zielrahmen; technische Implementierung offen**

## 1. Zweck

Die `DocumentationEngine` soll Infrastructure-as-Code nicht erst in einer späteren Ausbaustufe berücksichtigen. Für IaC-verwaltete Workloads ist Bicep von Beginn an als **First-Class-Desired-State-Quelle** einzuplanen.

Die Dokumentationsplattform unterscheidet deshalb bewusst zwischen zwei technischen Perspektiven:

```text
Actual State
  AzureInfrastructureCollector / weitere Collector

Desired State
  Bicep / IaC-Artefakte
```

Beide Perspektiven werden auf denselben providerunabhängigen Canonical Infrastructure Core abgebildet, dürfen aber nicht stillschweigend miteinander vermischt werden.

Bicep ersetzt den AzureInfrastructureCollector **nicht** als Quelle für den real vorhandenen Azure-Iststand. Umgekehrt ist der Collector keine Quelle dafür, welcher Zustand laut IaC gewünscht bzw. versioniert vorgesehen ist.

---

## 2. Verbindliches Zielbild

```text
                    +---------------------------+
                    | DocumentationEngine       |
                    +---------------------------+
                              ^         ^
                              |         |
                 Actual State|         |Desired State
                              |         |
              +---------------+         +----------------+
              |                                          |
AzureInfrastructureCollector                        Bicep / IaC
              |                                          |
              v                                          v
       Azure Provider Adapter                    Bicep Desired-State Adapter
              |                                          |
              +-------------------+  +-------------------+
                                  v  v
                         Canonical Infrastructure Model
                         - InfrastructureNode
                         - Relationship
                         - EvidenceReference
                         - CoverageRecord
                         - Graph Perspective
                                  |
                                  v
                         Semantic View Builder
                                  |
                    +-------------+-------------+
                    |                           |
             Actual-State Views          Desired-State Views
                    |                           |
                    +-------------+-------------+
                                  |
                                  v
                         spätere Reconciliation
                         / Desired-vs-Actual Diff
```

Eine spätere Reconciliation darf beide Perspektiven korrelieren. Sie ist jedoch eine explizite Transformation und kein implizites Zusammenwerfen beider Graphen.

---

## 3. Graph-Perspektiven

Jeder Canonical Graph besitzt eine explizite `perspective`.

Initiale Werte:

```text
actual
desiredTemplate
desiredDeployment
```

### `actual`

Der Graph beschreibt den belegten Iststand aus einem Collector bzw. einer freigegebenen Iststandsquelle.

### `desiredTemplate`

Der Graph beschreibt die durch IaC deklarierte Architektur eines Templates. Konditionale Ressourcen können vorhanden sein, auch wenn ihre konkrete Aktivierung für einen bestimmten Kunden/Deploymentkontext noch nicht aufgelöst ist.

### `desiredDeployment`

Der Graph beschreibt den für einen konkreten Deploymentkontext aufgelösten bzw. hinreichend bestimmten Desired State. Dafür dürfen ausschließlich freigegebene, sanitizte Deploymentmetadaten und nicht geheime Parameterwerte bzw. Secret-Referenzen verwendet werden.

Regel:

> Ein einzelner Canonical Graph enthält genau eine Perspektive.

Actual- und Desired-Nodes dürfen nicht allein aufgrund gleicher Namen zu einem gemeinsamen Node zusammengeführt werden.

---

## 4. Bicep als technische Source of Truth für Desired State

Für IaC-verwaltete Workloads gilt:

- die versionierte Bicep-Quelle ist die fachliche IaC-Source-of-Truth,
- die DocumentationEngine liest nicht aus einer manuell parallel gepflegten ARM-Datei,
- ein kompiliertes ARM-Template darf als deterministisches Maschinenartefakt für Parsing/Normalisierung dienen,
- Source-Repository, Commit, Bicep-Pfad und Compiler-/Buildmetadaten bleiben als Evidence erhalten,
- bei modularen Bicep-Deployments müssen die tatsächlich verwendeten Module eindeutig nachvollziehbar sein.

Die Engine darf Bicep nicht mit einer beliebigen, nicht dokumentierten Compiler-Version kompilieren und daraus einen scheinbar reproduzierbaren Vertrag ableiten.

Die konkrete Toolchain- und Versionsstrategie wird im Bicep-Adapter-Workchunk technisch festgelegt. Die Adapter-Schnittstelle muss jedoch Compiler-/Buildprovenance aufnehmen können.

---

## 5. Bicep Input Package – fachlicher Mindestbedarf

Der endgültige physische Paketvertrag bleibt Bestandteil der allgemeinen Input-Contract-Entscheidung. Fachlich muss ein Bicep/IaC-Input jedoch mindestens folgende Informationen liefern können:

### Source Provenance

- Repository-/Workload-Kennung,
- Commit/SHA oder gleichwertige immutable Version,
- Pfad der Root-Bicep-Datei,
- ggf. Modulquellen bzw. Modul-Manifest,
- Build-/Pipeline-Identität,
- Erstellungszeitpunkt des Artefakts.

### Compiler Provenance

- Bicep-Version,
- ggf. Azure-CLI-Version,
- Buildmodus,
- Hash des kompilierten Templates,
- Nachweis, dass der Output zum angegebenen Source-Commit gehört.

### Template Contract

- Ressourcen und Scopes,
- Parent-/Child-Strukturen,
- Abhängigkeiten,
- Resource-/Module-Symbolbeziehungen soweit technisch ableitbar,
- Parameterdefinitionen,
- sichere Parameterkennzeichnung,
- Outputs,
- Bedingungen bzw. konditionale Ressourcen.

### Deployment Context – nur für `desiredDeployment`

- nicht geheime, freigegebene Parameterwerte,
- Secret-Namen/-Referenzen statt Secret-Werten,
- logische Customer-/Environment-/Deployment-Profile,
- Informationen, die zur sicheren Auswertung von Conditions/Scopes benötigt werden.

Secret-Werte dürfen weder in Canonical Graphs noch Evidence noch Diagramm-/Dokumentartefakte übernommen werden.

---

## 6. Mapping auf den Canonical Core

Der Bicep Desired-State Adapter verwendet denselben Core-Contract wie Collector-Adapter.

Beispiel:

```text
resource vm 'Microsoft.Compute/virtualMachines@...' = {...}
        -> provider = azure
        -> providerType = Microsoft.Compute/virtualMachines
        -> kind = compute
        -> perspective = desiredTemplate oder desiredDeployment
```

Beziehungen werden bevorzugt aus maschinenauflösbaren Bicep-/ARM-Strukturen gebildet, beispielsweise:

- Parent-/Child-Beziehungen,
- Scope-Beziehungen,
- symbolische Resource References,
- `resourceId`-/ID-basierte Referenzen,
- explizite `dependsOn`,
- Property-Referenzen auf andere Ressourcen,
- Module Outputs/Inputs, soweit eindeutig auflösbar.

Nicht zulässig:

- Relationship-Erzeugung nur aus Ressourcennamen,
- Ergänzung typischer Azure-Komponenten,
- Annahme, dass eine konditionale Ressource für einen konkreten Kunden aktiviert ist, wenn der Deploymentkontext dies nicht belegt.

---

## 7. Evidence für Bicep

Bicep-generierte Nodes und Relationships benötigen dieselbe Evidence-Pflicht wie Actual-State-Daten.

Geeignete Evidence-Informationen sind insbesondere:

```text
sourceType = iac/bicep
sourceProvider = azure
repository = <logical repo>
commit = <immutable sha>
rootTemplate = <path>
sourceObjectId = <symbolic/resource identity>
path = <source span / compiled-json pointer>
compilerVersion = <version>
compiledArtifactHash = <sha256>
```

Die konkrete technische Feldstruktur bleibt Teil der späteren Serialisierungsentscheidung. Die Information muss jedoch maschinenlesbar erhalten bleiben.

---

## 8. Konditionale Ressourcen und Parameter

Bicep beschreibt nicht immer für jeden Kunden denselben effektiven Ressourcensatz.

Deshalb muss der Adapter unterscheiden können zwischen:

```text
declared
conditional
enabled
disabled
unresolved
```

Diese Zustände sind **Desired-State-Metadaten** und keine Actual-State-Behauptung.

Für `desiredTemplate` ist `conditional` bzw. `unresolved` zulässig.

Für `desiredDeployment` muss der Adapter soweit möglich die konkrete Aktivierung aus einem freigegebenen Deploymentkontext auflösen. Kann eine für die View relevante Condition nicht sicher ausgewertet werden, wird dies sichtbar als `unresolved` behandelt und nicht geraten.

---

## 9. Actual vs. Desired – Reconciliation

Bicep wird von Anfang an eingeplant, aber Desired-vs-Actual-Reconciliation ist eine eigene fachliche Schicht.

Zielbild:

```text
Canonical Graph (actual)
        +
Canonical Graph (desiredDeployment)
        |
        v
Reconciliation / Correlation
        |
        +--> managed + matching
        +--> desired but missing
        +--> actual but unmanaged/not in desired
        +--> property/config drift soweit belastbar vergleichbar
```

Wichtig:

- Korrelation erfolgt über stabile technische Identitäten bzw. eindeutig ableitbare IDs,
- Name-only Matching ist unzulässig,
- fehlende Korrelation ist ein eigener Zustand und keine automatische Driftbehauptung,
- der Reconciliation-Output ist kein ungeprüfter Canonical Graph, sondern ein eigenes View-/Diff-Modell.

Die vollständige Drift Engine ist nicht Bestandteil von DE-WC-01, wird aber jetzt als initiale Architekturkomponente eingeplant und nicht in eine unbestimmte spätere Zukunft verschoben.

---

## 10. Dokumentations- und Diagrammverwendung

Die DocumentationEngine muss die Perspektive eines Diagramms explizit kennen.

Beispiele:

### Iststandsdiagramm

```text
Perspective: actual
Source: AzureInfrastructureCollector
```

### IaC-Architekturdiagramm

```text
Perspective: desiredTemplate
Source: Bicep Commit <sha>
```

### Deployment-Sollbild

```text
Perspective: desiredDeployment
Source: Bicep Commit <sha> + sanitized Deployment Profile
```

### Drift-/Abgleichsicht

```text
Actual: Collector Snapshot <id>
Desired: IaC Commit <sha>
```

Ein Desired-State-Diagramm darf nicht als gemessener Kunden-Iststand beschriftet werden.

---

## 11. Verhältnis zum Azure-P9-Gate

Der AzureInfrastructureCollector-P9 bleibt weiterhin Voraussetzung für den **produktiven Actual-State-Azure-Adapter**.

Der Bicep Desired-State Adapter ist davon unabhängig und kann bereits vor P9 geplant und technisch prototypisiert werden.

Damit gilt:

```text
Azure Actual State
    -> Collector P9 erforderlich

Azure Desired State aus Bicep
    -> kein P9-Blocker
```

Das ermöglicht, Canonical Model, Semantic Views und Diagrammrenderer bereits mit realen versionierten IaC-Strukturen zu entwickeln, ohne den späteren Actual-State-Contract vorwegzunehmen.

---

## 12. Geplante Workchunks

### DE-WC-01 – Canonical Infrastructure Core

Der Core wird perspektivfähig implementiert. Zusätzlich zu Nodes, Relationships, Evidence und Coverage muss der Graph-Envelope mindestens seine `perspective` tragen.

DE-WC-01 implementiert noch keinen vollständigen Bicep-Parser, aber sein technischer Contract darf Bicep/Desired-State nicht ausschließen.

### DE-WC-02 – Semantic View Contracts

Views müssen angeben, welche Perspektiven sie akzeptieren:

- `actual`,
- `desiredTemplate`,
- `desiredDeployment`,
- später `reconciled`.

### DE-WC-03 – Initiale Provider-/Source-Adapter

Dieser Workchunk wird zweigeteilt:

1. Bicep Desired-State Adapter als initialer produktnaher Adapter gegen versionierte IaC-Fixtures/Artefakte,
2. Azure Actual-State Adapter zunächst als Fixture-/Prototype-Mapping; produktive Finalisierung bleibt P9-gegated.

### DE-WC-04 – Desired/Actual Reconciliation Contract

- stabile Korrelationsschlüssel,
- Match-/Missing-/Unmanaged-/Unresolved-Zustände,
- keine Name-only-Korrelation,
- Evidence auf beiden Seiten,
- kein Rendererwissen in der Reconciliation.

Die vollständige Property-Drift-Tiefe wird nur für Felder umgesetzt, deren Semantik provider- und source-seitig eindeutig vergleichbar definiert ist.

---

## 13. Gate für den Bicep-Adapter

Der Bicep Desired-State Adapter darf erst als produktionsnah `IMPLEMENTIERT` gelten, wenn mindestens nachgewiesen ist:

- immutable Source-Provenance,
- deterministische Toolchain-/Compiler-Provenance,
- kein Secret-Wert gelangt in Canonical Model/Evidence/Logs,
- Ressourcen und Relationships besitzen Evidence,
- Conditions werden nicht geraten,
- gleiche IaC-Inputs erzeugen denselben kanonischen Desired Graph,
- mindestens ein modularer Bicep-Fixture wird korrekt aufgelöst,
- Provider-/Resource-Typen bleiben nachvollziehbar,
- No-Invention-Gates greifen,
- Source-/Fachdokumentation und kanonischer Umsetzungsplan weisen denselben Status aus.

---

## 14. Nicht beschlossen durch dieses Dokument

Weiterhin offen bleiben:

- konkrete Programmiersprache der DocumentationEngine,
- konkrete Bicep-Parsing-Bibliothek,
- ob primär Bicep AST, Bicep CLI JSON/RPC oder kompiliertes ARM als technisches Parser-Backend verwendet wird,
- finale physische Paketstruktur,
- finale JSON-/YAML-/Protobuf-Serialisierung des Canonical Model,
- finale Renderer-/Layouttechnologie,
- konkrete Tiefe einer späteren Property-Drift-Engine.

Beschlossen ist jedoch, dass Bicep/IaC **von Anfang an** als Desired-State-Quelle in Core, Adapter- und View-Architektur berücksichtigt wird und kein nachträglicher Sonderweg wird.
