# DocumentationEngine – Canonical Infrastructure Model

Stand: 2026-09-01
Status: **BESCHLOSSENER Core-Contract für DE-WC-01; Implementierung noch offen**

## 1. Zweck

Dieses Dokument definiert den providerunabhängigen Kernvertrag der `DocumentationEngine` für Infrastrukturknoten, Beziehungen, Evidenz und Datenabdeckung.

Der Vertrag liegt bewusst **zwischen Source-/Provider-Adaptern und der semantischen View-Schicht**.

```text
Collector-/IaC-Artefakte
        |
        v
Source / Provider Adapter
        |
        v
Canonical Infrastructure Model
        |
        +--> Relationship Graph
        +--> Evidence / Coverage
        +--> Graph Perspective
        |
        v
Semantic View Builder
        |
        +--> Document View Model
        `--> Diagram View Model
```

Der Core-Contract ist weder Azure-spezifisch noch Renderer-spezifisch. Er legt keine Programmiersprache, keine konkrete Serialisierung und keine Rendertechnologie fest.

Für Azure-Actual-State gilt weiterhin die bestehende P9-Grenze: Das globale Modell darf jetzt definiert werden; der produktive Azure-Collector-Adapter und der endgültige Azure→DocumentationEngine-Relationship-Contract werden erst gegen das stabilisierte P9-Schema finalisiert.

Für IaC-verwaltete Workloads wird Bicep dagegen von Beginn an als **First-Class-Desired-State-Quelle** eingeplant. Details: [`../IAC_BICEP_INTERFACE.md`](../IAC_BICEP_INTERFACE.md).

---

## 2. Verbindliche Prinzipien

### DE-CAN-001 – Providerunabhängiger Kern

Das globale Modell darf keine Azure-Ressourcentypen als universelle Semantik voraussetzen.

Provider-spezifische Typen bleiben als Herkunftsinformation erhalten und werden durch Adapter auf eine kanonische Semantik abgebildet.

Beispiel:

```text
Microsoft.Compute/virtualMachines
        -> Azure Actual-State Adapter oder Bicep Desired-State Adapter
        -> kind = compute
```

Eine spätere Hyper-V-VM kann ebenfalls auf `kind = compute` abgebildet werden, ohne Azure-Semantik in den Core zu übernehmen.

### DE-CAN-002 – Stabile technische IDs vor Anzeigenamen

Relationships, Deduplizierung und spätere Korrelation verwenden stabile technische IDs bzw. daraus deterministisch erzeugte kanonische IDs.

Friendly Names und Display Names sind Präsentationsdaten und dürfen keine alleinige Beziehungsgrundlage bilden.

Die konkrete Stringkodierung kanonischer IDs bleibt Implementierungsdetail. Verbindlich ist:

- deterministisch,
- innerhalb eines Engine-Builds eindeutig,
- provider-/namespace-sicher,
- aus stabiler Source Identity reproduzierbar.

### DE-CAN-003 – Evidence ist First-Class

Jeder kanonische Infrastrukturknoten und jede kanonische Infrastrukturbeziehung muss auf mindestens eine freigegebene Faktenquelle zurückführbar sein.

Die Engine darf keine unbelegten Infrastrukturknoten oder -kanten erzeugen.

Deterministische Ableitungen sind nur zulässig, wenn:

- sie aus maschinenauflösbaren Fakten entstehen,
- keine freie Namensheuristik verwendet wird,
- die Quellfakten vollständig als Evidence referenziert werden,
- die Ableitung als solche gekennzeichnet wird.

### DE-CAN-004 – Coverage ist explizit

Nicht erhobene oder nur teilweise erhobene Domänen werden als Coverage-Zustand modelliert und nicht mit typischen Standardressourcen aufgefüllt.

### DE-CAN-005 – Semantische Gruppen sind keine erfundenen Infrastrukturressourcen

Workloads, Diagrammzonen oder andere fachliche Gruppierungen, die erst durch den `Semantic View Builder` entstehen, werden nicht als reale Infrastrukturressourcen in den Canonical Graph geschrieben.

Das Canonical Infrastructure Model enthält belegte Infrastruktur- und Serviceobjekte sowie belegte bzw. deterministisch ableitbare technische Relationships. Präsentationsgruppen gehören in die View Models.

### DE-CAN-006 – Renderer bleibt infrastrukturfrei

Der Renderer interpretiert keine Azure-, Bicep-, OPNsense- oder sonstige Source-/Providerlogik. Source-/Providerwissen endet spätestens am Adapter beziehungsweise Semantic View Builder.

### DE-CAN-007 – Actual und Desired werden nicht stillschweigend vermischt

Jeder Canonical Graph besitzt genau eine explizite Perspektive.

Initiale Perspektiven:

```text
actual
desiredTemplate
desiredDeployment
```

Ein Graph mit `actual` beschreibt belegten Iststand. Ein Graph mit `desiredTemplate` oder `desiredDeployment` beschreibt deklarierte bzw. deploymentbezogene IaC-Sollinformation.

Actual- und Desired-Nodes werden nicht allein aufgrund gleicher Namen zusammengeführt. Eine spätere Desired-vs-Actual-Reconciliation ist eine explizite Transformation mit eigenem Vertrag.

---

## 3. Kernobjekte und Graph-Envelope

DE-WC-01 definiert vier verpflichtende First-Class-Kernobjekte:

1. `InfrastructureNode`
2. `Relationship`
3. `EvidenceReference`
4. `CoverageRecord`

Zusätzlich besitzt der Canonical Graph einen verpflichtenden Envelope mit mindestens:

- `perspective`,
- Build-/Graph-Identität,
- Modell-/Contract-Version,
- Source-Set-/Input-Referenzen, soweit technisch erforderlich.

Der Envelope ist keine fünfte Infrastrukturressource, sondern Kontext des Graphen.

Ein separates renderer-spezifisches Diagrammobjekt gehört ausdrücklich **nicht** zu DE-WC-01.

---

## 4. InfrastructureNode

Ein `InfrastructureNode` repräsentiert ein durch Input belegtes technisches Objekt innerhalb genau einer Graph-Perspektive.

### 4.1 Pflichtfelder

| Feld | Bedeutung |
|---|---|
| `id` | kanonische, stabile, eindeutige Node-ID |
| `provider` | Herkunftsprovider bzw. Provider-Namespace |
| `providerType` | ursprünglicher provider-spezifischer Typ |
| `kind` | providerunabhängige kanonische Objektklasse |
| `name` | technischer Name aus der Quelle |
| `evidenceRefs[]` | mindestens eine Evidence-Referenz |

### 4.2 Optionale Kernfelder

| Feld | Bedeutung |
|---|---|
| `displayName` | lesbarer Anzeigename, falls explizit vorhanden |
| `location` | Standort/Region, wenn fachlich belegt |
| `properties` | provider- oder domänenspezifische normalisierte Detailfelder |
| `tags` | freigegebene Tags/Metadaten |
| `sourceIds[]` | stabile technische Ursprungs-IDs, soweit zusätzlich erforderlich |

Provider-spezifische Detailfelder dürfen in `properties` erhalten bleiben. Sie dürfen jedoch nicht als globale Core-Semantik ausgegeben werden, wenn keine providerübergreifende Bedeutung beschlossen wurde.

Desired-State-spezifische technische Metadaten wie `declared`, `conditional`, `enabled`, `disabled` oder `unresolved` dürfen als klar gekennzeichnete Desired-State-Eigenschaften erhalten bleiben. Sie sind keine Actual-State-Behauptung.

### 4.3 Initiale kanonische `kind`-Klassen

Der initiale Bedarf umfasst mindestens:

```text
scope
compute
host
network
subnet
interface
endpoint
gateway
dns
storage
database
identity
application
service
backup
vault
security
management
policy
device
other
```

`other` ist zulässig, wenn ein belegtes Source-/Providerobjekt noch keiner globalen Klasse sinnvoll zugeordnet werden kann. In diesem Fall bleibt `providerType` verpflichtend erhalten.

Die Liste ist versioniert erweiterbar. Neue Klassen werden nicht ad hoc in einzelnen Adaptern erfunden, sondern zentral registriert.

### 4.4 Scopes und Container

Technische Scopes bzw. reale Container – beispielsweise Tenant, Subscription, Resource Group, VNet oder vergleichbare Providerobjekte – werden als belegte `InfrastructureNode`-Objekte modelliert, wenn sie für Dokumentation oder Relationship-Auflösung relevant sind.

Hierarchie wird über explizite Relationships wie `contains` abgebildet.

Semantische Diagrammzonen wie `Connectivity`, `Workloads` oder `Operations` sind dagegen View-Model-Konstrukte und keine InfrastructureNodes.

---

## 5. Relationship

Ein `Relationship` verbindet zwei vorhandene `InfrastructureNode`-Objekte innerhalb desselben Canonical Graphs.

### 5.1 Pflichtfelder

| Feld | Bedeutung |
|---|---|
| `id` | deterministische, eindeutige Relationship-ID |
| `sourceNodeId` | vorhandene kanonische Node-ID |
| `targetNodeId` | vorhandene kanonische Node-ID |
| `kind` | kanonische providerunabhängige Relationship-Semantik |
| `providerRelationshipType` | ursprünglicher Relationship-Typ bzw. Herkunftsbezeichner, sofern vorhanden |
| `evidenceRefs[]` | mindestens eine Evidence-Referenz |
| `evidenceClass` | `direct` oder `deterministicDerivation` |

### 5.2 Optionale Felder

| Feld | Bedeutung |
|---|---|
| `properties` | technische Relationship-Metadaten |
| `sourceRelationshipId` | stabile ID der Quellbeziehung, falls vorhanden |

### 5.3 Initiale kanonische Relationship-Arten

Der providerunabhängige Mindestbedarf umfasst:

```text
contains
connectedTo
uses
dependsOn
hosts
backedBy
protects
securedBy
routesThrough
attachedTo
memberOf
monitoredBy
storesOn
resolvesThrough
assignedTo
exposes
manages
```

Die Liste bildet fachliche Semantik ab und ist kein 1:1-Abbild der Relationship-Namen eines bestimmten Collectors oder IaC-Werkzeugs.

Beispiele:

```text
Azure Collector: UsesNetworkInterface
    -> canonical: uses

Azure Collector: BackedByVm
    -> canonical: backedBy

Bicep/ARM: symbolische Resource Reference
    -> canonical: uses / dependsOn / attachedTo
       abhängig von der fachlich definierten Property-Semantik
```

Provider-spezifische Semantik bleibt über `providerRelationshipType` erhalten.

### 5.4 Richtung

`sourceNodeId` und `targetNodeId` definieren die kanonische Richtung.

Bidirektionale technische Sachverhalte dürfen nicht durch willkürliche doppelte Kanten erzeugt werden. Der jeweilige Relationship-Vertrag legt fest, ob eine Semantik gerichtet oder symmetrisch interpretiert wird.

### 5.5 Verbotene Heuristiken

Nicht zulässig sind Relationships, die ausschließlich entstehen aus:

- ähnlichen Namen,
- Namenspräfixen/-suffixen,
- typischen Referenzarchitekturen,
- räumlicher Nähe im Diagramm,
- generativer KI-Inferenz.

Eine deterministische Ableitung aus einer stabilen Resource-ID, einem expliziten Foreign Key, einer Bicep-/ARM-Resource-Reference oder einer gleichwertigen maschinenauflösbaren Referenz ist zulässig und muss als `deterministicDerivation` markiert werden.

---

## 6. EvidenceReference

`EvidenceReference` bildet die Rückverfolgbarkeit zu einer freigegebenen Quelle ab.

### 6.1 Mindestfelder

| Feld | Bedeutung |
|---|---|
| `id` | eindeutige Evidence-ID innerhalb des Builds |
| `sourceType` | Collector/IaC/Artefakt-/Konfigurationsklasse |
| `sourceProvider` | Provider bzw. Quellsystem |
| `snapshotId` | Snapshot-/Erfassungs-/Build-Identität |
| `artifact` | logischer Artefaktname oder Artefaktpfad |

### 6.2 Optionale Felder

| Feld | Bedeutung |
|---|---|
| `collector` | Collector-Kennung und Version, soweit vorhanden |
| `sourceObjectId` | stabile Quellobjekt-ID |
| `sourceRelationshipId` | stabile Quellbeziehungs-ID |
| `path` | maschinenlesbarer Pfad/JSON-Pointer/Source-Span zum Ursprungsobjekt |
| `capturedAt` | Erfassungs- bzw. Buildzeitpunkt, falls geliefert |
| `derivation` | dokumentierte deterministische Ableitungsregel |
| `repository` | logische Source-Repository-Kennung bei versionierten IaC-/Konfigurationsquellen |
| `commit` | immutable Source-Version, z. B. Git SHA |
| `toolchain` | relevante Parser-/Compiler-/Build-Provenance |
| `artifactHash` | Hash eines abgeleiteten Maschinenartefakts, soweit vorhanden |

Evidence ist technische Provenance. Sie darf keine Secrets oder nicht freigegebenen RAW-Werte in Dokumentationsartefakte übernehmen.

Für Bicep muss Evidence die IaC-Quelle auf einen konkreten versionierten Stand zurückführen können. Details stehen in `docs/IAC_BICEP_INTERFACE.md`.

---

## 7. CoverageRecord

Ein `CoverageRecord` beschreibt, ob eine fachliche Domäne für einen Scope innerhalb der jeweiligen Perspektive abgedeckt wurde.

### 7.1 Pflichtfelder

| Feld | Bedeutung |
|---|---|
| `domain` | fachliche Domäne, z. B. `rbac`, `network`, `backup` |
| `status` | definierter Coverage-Zustand |
| `evidenceRefs[]` | Quelle der Coverage-Aussage, soweit vorhanden |

Optional kann ein Coverage-Eintrag auf einen konkreten Scope/Node begrenzt werden.

### 7.2 Coverage-Zustände

Verbindliche initiale Zustände:

```text
collected
partial
notCollected
notApplicable
unavailable
```

Bedeutung:

- `collected` – Domäne wurde für den angegebenen Scope fachlich erhoben bzw. für die konkrete Source-Perspektive vollständig verarbeitet.
- `partial` – Domäne ist nur teilweise abgedeckt.
- `notCollected` – Domäne war nicht Bestandteil der Erhebung bzw. Source.
- `notApplicable` – Domäne ist für den Scope fachlich nicht anwendbar.
- `unavailable` – Domäne sollte vorliegen, war technisch/vertraglich aber nicht verfügbar.

`notCollected` und `unavailable` dürfen nicht in positive Infrastrukturbehauptungen übersetzt werden.

---

## 8. Canonical Graph

Der Canonical Graph besteht fachlich aus:

```text
perspective
nodes[]
relationships[]
evidence[]
coverage[]
```

Die konkrete physische Serialisierung bleibt vorerst offen. Unabhängig vom späteren Format gelten folgende Invarianten.

### DE-CAN-VAL-001 – Eindeutige IDs

- Node-IDs sind eindeutig.
- Relationship-IDs sind eindeutig.
- Evidence-IDs sind eindeutig.

### DE-CAN-VAL-002 – Referentielle Integrität

- `sourceNodeId` existiert.
- `targetNodeId` existiert.
- jede `evidenceRef` existiert.
- keine verwaiste Relationship ist zulässig.

### DE-CAN-VAL-003 – Evidence-Pflicht

- jeder Node besitzt mindestens eine Evidence-Referenz,
- jede Relationship besitzt mindestens eine Evidence-Referenz,
- eine deterministische Ableitung dokumentiert ihre Ursprungsfakten.

Verletzungen sind harte Fehler.

### DE-CAN-VAL-004 – Bekannte Semantik

`kind`-Werte von Nodes und Relationships müssen aus dem versionierten zentralen Registry-Vertrag stammen.

Unbekannte Providerobjekte werden über `kind = other` erhalten statt durch eine erfundene Klasse falsch normalisiert zu werden.

### DE-CAN-VAL-005 – Keine widersprüchliche Coverage

Für dieselbe Domain und denselben Scope dürfen keine logisch widersprüchlichen aktiven Coverage-Zustände entstehen.

### DE-CAN-VAL-006 – Deterministische Reihenfolge

Bei identischem Input muss die normalisierte Ausgabe in einer festgelegten kanonischen Sortierung reproduzierbar sein.

Mindestens:

- Nodes nach kanonischer ID,
- Relationships nach `sourceNodeId`, `kind`, `targetNodeId`, `id`,
- Evidence nach ID,
- Coverage nach Scope/Domain/Status.

Die physische Serialisierung darf andere technische Sortieranforderungen ergänzen, aber nicht nichtdeterministisch werden.

### DE-CAN-VAL-007 – Keine Semantic-View-Artefakte im Core

Diagrammzonen, Layoutpositionen, Iconpfade, Farben und Rendererattribute sind im Canonical Infrastructure Model unzulässig.

### DE-CAN-VAL-008 – Perspektive ist Pflicht

Jeder Graph muss genau eine bekannte Perspektive deklarieren.

Ein Graph darf nicht gleichzeitig `actual` und `desired*` sein.

### DE-CAN-VAL-009 – Kein implizites Desired/Actual-Merging

Eine Adapter- oder Core-Transformation darf nicht stillschweigend Actual- und Desired-Objekte aufgrund von Namen oder visueller Ähnlichkeit zusammenführen.

Eine spätere Korrelation muss über einen expliziten Reconciliation-Contract erfolgen.

---

## 9. Source-/Provider-Adapter-Vertrag

Ein Source-/Provider-Adapter ist verantwortlich für:

- Prüfung des erwarteten Input-Contracts,
- Festlegung der passenden Graph-Perspektive,
- deterministische ID-Bildung,
- Mapping von Source-/Providerobjekten auf `InfrastructureNode`,
- Mapping expliziter bzw. deterministisch ableitbarer Relationships auf kanonische Relationships,
- Erhalt des ursprünglichen `providerType` und `providerRelationshipType`,
- Erzeugung der Evidence-Referenzen,
- Übernahme/Abbildung von Coverage,
- Fail-Closed bei nicht auflösbaren Pflichtreferenzen.

Ein Adapter darf **nicht**:

- typische Infrastruktur ergänzen,
- Relationship-Fakten aus Anzeigenamen raten,
- Actual und Desired ungeprüft zusammenführen,
- View-/Layoutentscheidungen treffen,
- Rendererattribute erzeugen.

### 9.1 Azure-Actual-State-P9-Gate

Für Azure dürfen vorhandene fachmodulspezifische Collector-Relationships für Fixtures und nichtproduktive Mappingtests verwendet werden.

Der produktive Azure-Actual-State-Adapter bleibt jedoch blockiert, bis:

1. P9 im `AzureInfrastructureCollector` abgeschlossen ist,
2. das P9-Schema stabilisiert und versioniert wurde,
3. der Azure→DocumentationEngine-Contract dagegen geprüft wurde.

### 9.2 Bicep Desired-State Adapter

Der Bicep Desired-State Adapter ist **kein P9-abhängiger Adapter**.

Er wird als initialer IaC-Adapter eingeplant und bildet versionierte Bicep-/IaC-Fakten auf `desiredTemplate`- bzw. `desiredDeployment`-Graphen ab.

Verbindliche Detailanforderungen: [`../IAC_BICEP_INTERFACE.md`](../IAC_BICEP_INTERFACE.md).

### 9.3 Weitere Provider

OPNsense, Hyper-V, Switch/Layer-2 und spätere Provider verwenden denselben Core-Contract. Provider-spezifische Erweiterungen bleiben im Adapter bzw. in `properties`, sofern keine globale Semantik beschlossen ist.

---

## 10. Actual State, Desired State und Reconciliation

### 10.1 Actual State

Produktive Azure-Iststandsdokumentation verwendet weiterhin freigegebene Collector-Artefakte und später den P9-basierten Azure-Actual-State-Adapter.

```text
AzureInfrastructureCollector
        -> Azure Actual-State Adapter
        -> Canonical Graph [actual]
```

### 10.2 Desired State aus Bicep

Für IaC-verwaltete Workloads ist Bicep von Beginn an als gewünschte Architekturquelle eingeplant.

```text
Bicep / deterministisches IaC-Artefakt
        -> Bicep Desired-State Adapter
        -> Canonical Graph [desiredTemplate|desiredDeployment]
```

Bicep ersetzt nicht den Collector als Iststandsbeweis. Der Collector ersetzt nicht Bicep als versionierten IaC-Sollvertrag.

### 10.3 Reconciliation

Desired-vs-Actual wird als eigener Contract eingeplant:

```text
Canonical Graph [actual]
        +
Canonical Graph [desiredDeployment]
        -> Reconciliation
        -> Match / Missing / Unmanaged / Unresolved / belastbare Drift
```

Name-only Matching ist verboten. Der Reconciliation-Output gehört nicht ungeprüft zurück in einen Canonical Graph und darf keine nicht belegte Infrastruktur erfinden.

---

## 11. Testvertrag für DE-WC-01

Die spätere Core-Implementierung muss mindestens folgende automatisierte Contract-Tests enthalten.

### 11.1 Positive Fixtures

- minimaler `actual`-Graph mit zwei Nodes und einer Relationship,
- minimaler `desiredTemplate`-Graph mit Bicep-/IaC-Evidence,
- mehrere Scopes mit `contains`,
- Providerobjekt mit `kind = other` und erhaltenem `providerType`,
- direkte Relationship mit Evidence,
- deterministisch abgeleitete Relationship aus stabiler technischer Referenz,
- Coverage für `collected`, `partial`, `notCollected`, `notApplicable`, `unavailable`,
- gleiche kanonische Semantik aus mindestens zwei unterschiedlichen Source-/Provider-Namensräumen.

### 11.2 Negative Fixtures / Fail Closed

- doppelte Node-ID,
- doppelte Relationship-ID,
- fehlender Source-Node,
- fehlender Target-Node,
- fehlende Evidence-Referenz,
- Node ohne Evidence,
- Relationship ohne Evidence,
- unbekannter nicht registrierter `kind`,
- widersprüchliche Coverage,
- Relationship nur aus Namensheuristik,
- Semantic-View-/Layoutattribute im Canonical Core,
- fehlende oder unbekannte Graph-Perspektive,
- Versuch, Actual- und Desired-Objekte implizit zu einem Graph zu mischen.

### 11.3 Determinismus

Gleiche fachliche Inputs in unterschiedlicher Eingabereihenfolge müssen denselben kanonisch sortierten Graph erzeugen.

### 11.4 Provider-/Source-Unabhängigkeit

Mindestens zwei unterschiedliche Source-/Provider-Fixtures müssen beweisen, dass Core-Validierung und Relationship-Semantik keine Azure-Collector-spezifischen Typnamen voraussetzen.

Ein Bicep-Desired-State-Fixture darf hierfür ausdrücklich als zweite Source-Klasse verwendet werden.

### 11.5 No-Invention-Regression

Für jeden ausgegebenen Node und jede Relationship muss eine Evidence-Kette vorhanden sein.

Ein Test muss explizit verhindern, dass typische Referenzarchitektur-Komponenten ohne Quellbeleg in den Canonical Graph gelangen.

### 11.6 Perspektivtreue

Tests müssen beweisen:

- `actual` wird nicht als Desired State umgedeutet,
- Bicep `desiredTemplate` wird nicht als gemessener Iststand ausgegeben,
- nicht aufgelöste Conditions werden nicht als aktiv geraten,
- Reconciliation ist keine implizite Core-Funktion.

---

## 12. DE-WC-01 – Umsetzung und Gate

### Ziel

Den in diesem Dokument spezifizierten providerunabhängigen, perspektivfähigen Core-Contract als stabile Grundlage für Collector-, IaC-, Semantic-View- und Renderer-Workchunks technisch implementierbar festlegen.

### Nicht Bestandteil

- finale Input-Paket-/Dateistruktur,
- produktiver Azure-P9-Actual-State-Adapter,
- vollständiger Bicep-Parser/Desired-State-Adapter,
- OPNsense-Adapter,
- Desired-vs-Actual-Reconciliation-Implementierung,
- Document View Model,
- Diagram View Model,
- Semantic View Builder Implementierung,
- Programmiersprache der Engine,
- konkrete JSON-/YAML-/Protobuf-Serialisierung,
- Renderer/Layoutbibliothek,
- Azure Icon Integration,
- Markdown/PDF/DOCX-Rendering,
- PipelineTemplates-Integration.

DE-WC-01 muss jedoch Graph-Perspektiven, IaC-Evidence und die spätere Bicep-Adapterfähigkeit bereits im Core technisch ermöglichen. Bicep darf nicht als nachträglicher Sonderweg eine inkompatible zweite Modellwelt benötigen.

### Geplante Repository-Grundstruktur nach Freigabe der Implementierung

Die spätere physische Codebasis soll die beschlossenen Verantwortungsgrenzen abbilden. Eine mögliche minimale Struktur ist:

```text
contracts/
  canonical/
  providers/
  sources/
    bicep/

src/
  core/
    model/
    relationships/
    coverage/
    validation/
  adapters/
    bicep/
  semantic/
  reconciliation/
  diagrams/
  documents/

tests/
  unit/
  contracts/
  fixtures/
    bicep/
  golden/
```

Diese Ordnerstruktur ist eine Verantwortungsstruktur und noch keine Festlegung auf eine konkrete Sprache oder Rendererbibliothek.

### Implementierungs-Gate

DE-WC-01 darf erst als technisch `IMPLEMENTIERT` gelten, wenn:

- der Core-Contract im Repository versioniert ist,
- Node-/Relationship-/Evidence-/Coverage-Modelle technisch repräsentiert sind,
- der Graph-Envelope eine validierte Perspektive trägt,
- alle Core-Invarianten automatisiert validiert werden,
- positive und negative Contract-Fixtures vorhanden sind,
- Determinismus nachgewiesen ist,
- Provider-/Source-Unabhängigkeit nachgewiesen ist,
- mindestens ein Bicep-/Desired-State-Core-Fixture ohne Sondermodell validiert wird,
- No-Invention-/Evidence-Gates fail-closed sind,
- Actual und Desired nicht implizit vermischt werden können,
- keine Azure-P9-Abhängigkeit in den globalen Core eingeschleust wurde,
- keine Renderer-/Layouttechnologie vorgezogen wurde,
- Fachdokumentation und kanonischer Umsetzungsplan denselben bestätigten Status enthalten.

Bis dieses Gate erfüllt ist, ist der Vertrag **BESCHLOSSEN**, aber die Core Engine **nicht IMPLEMENTIERT**.

---

## 13. Nächste Workchunks

Nach DE-WC-01 folgen fachlich:

1. **DE-WC-02 – Semantic View Contracts**
   - providerunabhängige View-Anforderungen,
   - fünf Standard-View-Typen,
   - Perspektivvertrag für `actual`, `desiredTemplate`, `desiredDeployment` und später `reconciled`,
   - klare Trennung Infrastrukturgraph vs. Präsentationsgruppen.
2. **DE-WC-03 – Initiale Source-/Provider-Adapter**
   - Bicep Desired-State Adapter als initialer produktnaher IaC-Adapter,
   - nichtproduktives Azure-Actual-State-Mapping gegen vorhandene Fixture-Daten,
   - produktiver Azure-Actual-State-Adapter weiterhin P9-gegated,
   - OPNsense-Vertrag nach Sichtung.
3. **DE-WC-04 – Desired/Actual Reconciliation Contract**
   - stabile Korrelationsschlüssel,
   - Match-/Missing-/Unmanaged-/Unresolved-Zustände,
   - Evidence auf beiden Seiten,
   - keine Name-only-Korrelation.

Renderer- und Layoutauswahl erfolgen erst, wenn Canonical Model und Diagram View Model stabil genug sind, um die Technologie anhand eines echten Contracts zu bewerten. Bicep kann dabei bereits vor Abschluss von Azure-P9 als reale Desired-State-Datenbasis für Prototypen und Contract-Tests dienen.
