# DocumentationEngine – Canonical Infrastructure Model

Stand: 2026-09-01
Status: **BESCHLOSSENER Core-Contract für DE-WC-01; Implementierung noch offen**

## 1. Zweck

Dieses Dokument definiert den providerunabhängigen Kernvertrag der `DocumentationEngine` für Infrastrukturknoten, Beziehungen, Evidenz und Datenabdeckung.

Der Vertrag liegt bewusst **zwischen Provider-Adaptern und der semantischen View-Schicht**.

```text
Provider-/Collector-Artefakte
        |
        v
Provider Adapter
        |
        v
Canonical Infrastructure Model
        |
        +--> Relationship Graph
        +--> Evidence / Coverage
        |
        v
Semantic View Builder
        |
        +--> Document View Model
        `--> Diagram View Model
```

Der Core-Contract ist weder Azure-spezifisch noch Renderer-spezifisch. Er legt keine Programmiersprache, keine konkrete Serialisierung und keine Rendertechnologie fest.

Für Azure gilt weiterhin die bestehende P9-Grenze: Das globale Modell darf jetzt definiert werden; der produktive Azure-Provider-Adapter und der endgültige Azure→DocumentationEngine-Relationship-Contract werden erst gegen das stabilisierte P9-Schema finalisiert.

---

## 2. Verbindliche Prinzipien

### DE-CAN-001 – Providerunabhängiger Kern

Das globale Modell darf keine Azure-Ressourcentypen als universelle Semantik voraussetzen.

Provider-spezifische Typen bleiben als Herkunftsinformation erhalten und werden durch Provider-Adapter auf eine kanonische Semantik abgebildet.

Beispiel:

```text
Microsoft.Compute/virtualMachines
        -> Azure Adapter
        -> kind = compute
```

Eine spätere Hyper-V-VM kann ebenfalls auf `kind = compute` abgebildet werden, ohne Azure-Semantik in den Core zu übernehmen.

### DE-CAN-002 – Stabile technische IDs vor Anzeigenamen

Relationships, Deduplizierung und Korrelation verwenden stabile technische IDs bzw. daraus deterministisch erzeugte kanonische IDs.

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

Der Renderer interpretiert keine Azure-, OPNsense- oder sonstige Providerlogik. Providerwissen endet spätestens am Provider-Adapter beziehungsweise Semantic View Builder.

---

## 3. Kernobjekte

DE-WC-01 definiert vier verpflichtende Kernobjekte:

1. `InfrastructureNode`
2. `Relationship`
3. `EvidenceReference`
4. `CoverageRecord`

Ein separates renderer-spezifisches Diagrammobjekt gehört ausdrücklich **nicht** zu DE-WC-01.

---

## 4. InfrastructureNode

Ein `InfrastructureNode` repräsentiert ein durch Input belegtes technisches Objekt.

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

`other` ist zulässig, wenn ein belegtes Providerobjekt noch keiner globalen Klasse sinnvoll zugeordnet werden kann. In diesem Fall bleibt `providerType` verpflichtend erhalten.

Die Liste ist versioniert erweiterbar. Neue Klassen werden nicht ad hoc in einzelnen Adaptern erfunden, sondern zentral registriert.

### 4.4 Scopes und Container

Technische Scopes bzw. reale Container – beispielsweise Tenant, Subscription, Resource Group, VNet oder vergleichbare Providerobjekte – werden als belegte `InfrastructureNode`-Objekte modelliert, wenn sie für Dokumentation oder Relationship-Auflösung relevant sind.

Hierarchie wird über explizite Relationships wie `contains` abgebildet.

Semantische Diagrammzonen wie `Connectivity`, `Workloads` oder `Operations` sind dagegen View-Model-Konstrukte und keine InfrastructureNodes.

---

## 5. Relationship

Ein `Relationship` verbindet zwei vorhandene `InfrastructureNode`-Objekte.

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

Die Liste bildet fachliche Semantik ab und ist kein 1:1-Abbild der Relationship-Namen eines bestimmten Collectors.

Beispiele:

```text
Azure: UsesNetworkInterface
    -> canonical: uses

Azure: BackedByVm
    -> canonical: backedBy

Azure: ProtectsResource
    -> canonical: protects
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

Eine deterministische Ableitung aus einer stabilen Resource-ID, einem expliziten Foreign Key oder einer gleichwertigen maschinenauflösbaren Referenz ist zulässig und muss als `deterministicDerivation` markiert werden.

---

## 6. EvidenceReference

`EvidenceReference` bildet die Rückverfolgbarkeit zu einer freigegebenen Quelle ab.

### 6.1 Mindestfelder

| Feld | Bedeutung |
|---|---|
| `id` | eindeutige Evidence-ID innerhalb des Builds |
| `sourceType` | Collector/Artefakt-/Konfigurationsklasse |
| `sourceProvider` | Provider bzw. Quellsystem |
| `snapshotId` | Snapshot-/Erfassungs-/Build-Identität |
| `artifact` | logischer Artefaktname oder Artefaktpfad |

### 6.2 Optionale Felder

| Feld | Bedeutung |
|---|---|
| `collector` | Collector-Kennung und Version, soweit vorhanden |
| `sourceObjectId` | stabile Quellobjekt-ID |
| `sourceRelationshipId` | stabile Quellbeziehungs-ID |
| `path` | maschinenlesbarer Pfad/JSON-Pointer zum Ursprungsobjekt |
| `capturedAt` | Erfassungszeitpunkt, falls geliefert |
| `derivation` | dokumentierte deterministische Ableitungsregel |

Evidence ist technische Provenance. Sie darf keine Secrets oder nicht freigegebenen RAW-Werte in Dokumentationsartefakte übernehmen.

---

## 7. CoverageRecord

Ein `CoverageRecord` beschreibt, ob eine fachliche Domäne für einen Scope erhoben wurde.

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

- `collected` – Domäne wurde für den angegebenen Scope fachlich erhoben.
- `partial` – Domäne ist nur teilweise abgedeckt.
- `notCollected` – Domäne war nicht Bestandteil der Erhebung.
- `notApplicable` – Domäne ist für den Scope fachlich nicht anwendbar.
- `unavailable` – Domäne sollte erhoben werden, war technisch/vertraglich aber nicht verfügbar.

`notCollected` und `unavailable` dürfen nicht in positive Infrastrukturbehauptungen übersetzt werden.

---

## 8. Canonical Graph

Der Canonical Graph besteht aus:

```text
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

---

## 9. Provider-Adapter-Vertrag

Ein Provider-Adapter ist verantwortlich für:

- Prüfung des erwarteten Provider-/Collector-Contracts,
- deterministische ID-Bildung,
- Mapping von Providerobjekten auf `InfrastructureNode`,
- Mapping expliziter Provider-Relationships auf kanonische Relationships,
- Erhalt des ursprünglichen `providerType` und `providerRelationshipType`,
- Erzeugung der Evidence-Referenzen,
- Übernahme/Abbildung von Coverage,
- Fail-Closed bei nicht auflösbaren Pflichtreferenzen.

Ein Provider-Adapter darf **nicht**:

- typische Infrastruktur ergänzen,
- Relationship-Fakten aus Anzeigenamen raten,
- View-/Layoutentscheidungen treffen,
- Rendererattribute erzeugen.

### 9.1 Azure-P9-Gate

Für Azure dürfen vorhandene fachmodulspezifische Relationships für Fixtures und nichtproduktive Mappingtests verwendet werden.

Der produktive Azure-Adapter bleibt jedoch blockiert, bis:

1. P9 im `AzureInfrastructureCollector` abgeschlossen ist,
2. das P9-Schema stabilisiert und versioniert wurde,
3. der Azure→DocumentationEngine-Contract dagegen geprüft wurde.

### 9.2 Weitere Provider

OPNsense, Hyper-V, Switch/Layer-2 und spätere Provider verwenden denselben Core-Contract. Provider-spezifische Erweiterungen bleiben im Adapter bzw. in `properties`, sofern keine globale Semantik beschlossen ist.

---

## 10. Bicep / Desired State

Bicep ist im bisherigen produktiven Collector-Pfad **keine beschlossene primäre Iststandsquelle** der DocumentationEngine.

Für eine spätere Erweiterung ist jedoch folgende saubere Integrationsgrenze vorgesehen:

```text
Bicep / kompiliertes ARM
        -> eigener Desired-State-Adapter
        -> Canonical Model oder vergleichbares versioniertes Desired-State-Modell
```

Eine spätere Desired-vs-Actual-Diff-Funktion darf erst als eigene Architekturentscheidung eingeführt werden.

DE-WC-01 implementiert keinen Bicep-Adapter und ändert nicht die bestehende Regel, dass produktive Kundendokumentation des Azure-Iststands aus freigegebenen Collector-Artefakten entsteht.

---

## 11. Testvertrag für DE-WC-01

Die spätere Core-Implementierung muss mindestens folgende automatisierte Contract-Tests enthalten.

### 11.1 Positive Fixtures

- minimaler Graph mit zwei Nodes und einer Relationship,
- mehrere Scopes mit `contains`,
- Providerobjekt mit `kind = other` und erhaltenem `providerType`,
- direkte Relationship mit Evidence,
- deterministisch abgeleitete Relationship aus stabiler technischer Referenz,
- Coverage für `collected`, `partial`, `notCollected`, `notApplicable`, `unavailable`.

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
- Semantic-View-/Layoutattribute im Canonical Core.

### 11.3 Determinismus

Gleiche fachliche Inputs in unterschiedlicher Eingabereihenfolge müssen denselben kanonisch sortierten Graph erzeugen.

### 11.4 Providerunabhängigkeit

Mindestens eine synthetische zweite Provider-Fixture muss beweisen, dass Core-Validierung und Relationship-Semantik keine Azure-spezifischen Typnamen voraussetzen.

### 11.5 No-Invention-Regression

Für jeden ausgegebenen Node und jede Relationship muss eine Evidence-Kette vorhanden sein.

Ein Test muss explizit verhindern, dass typische Referenzarchitektur-Komponenten ohne Quellbeleg in den Canonical Graph gelangen.

---

## 12. DE-WC-01 – Umsetzung und Gate

### Ziel

Den in diesem Dokument spezifizierten providerunabhängigen Core-Contract als stabile Grundlage für alle späteren Adapter, Semantic Views und Renderer festlegen.

### Nicht Bestandteil

- finale Input-Paket-/Dateistruktur,
- produktiver Azure-P9-Adapter,
- OPNsense-Adapter,
- Bicep-Adapter,
- Document View Model,
- Diagram View Model,
- Semantic View Builder Implementierung,
- Programmiersprache der Engine,
- konkrete JSON-/YAML-/Protobuf-Serialisierung,
- Renderer/Layoutbibliothek,
- Azure Icon Integration,
- Markdown/PDF/DOCX-Rendering,
- PipelineTemplates-Integration.

### Geplante Repository-Grundstruktur nach Freigabe der Implementierung

Die spätere physische Codebasis soll die bereits beschlossenen Verantwortungsgrenzen abbilden. Eine mögliche minimale Struktur ist:

```text
contracts/
  canonical/
  providers/

src/
  core/
    model/
    relationships/
    coverage/
    validation/
  adapters/
  semantic/
  diagrams/
  documents/

tests/
  unit/
  contracts/
  fixtures/
  golden/
```

Diese Ordnerstruktur ist eine Verantwortungsstruktur und noch keine Festlegung auf eine konkrete Sprache oder Rendererbibliothek.

### Implementierungs-Gate

DE-WC-01 darf erst als technisch `IMPLEMENTIERT` gelten, wenn:

- der Core-Contract im Repository versioniert ist,
- Node-/Relationship-/Evidence-/Coverage-Modelle technisch repräsentiert sind,
- alle Core-Invarianten automatisiert validiert werden,
- positive und negative Contract-Fixtures vorhanden sind,
- Determinismus nachgewiesen ist,
- Providerunabhängigkeit durch mindestens zwei Provider-Namensräume nachgewiesen ist,
- No-Invention-/Evidence-Gates fail-closed sind,
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
   - klare Trennung Infrastrukturgraph vs. Präsentationsgruppen.
2. **DE-WC-03 – Provider-Adapter-Prototypen**
   - nichtproduktives Azure-Mapping gegen vorhandene Fixture-Daten,
   - produktiver Azure-Adapter weiterhin P9-gegated,
   - OPNsense-Vertrag nach Sichtung.
3. **später: Desired-State/Bicep-Adapter**
   - nur nach eigener Architekturentscheidung,
   - kein Bestandteil des initialen Iststandscontracts.

Renderer- und Layoutauswahl erfolgen erst, wenn Canonical Model und Diagram View Model stabil genug sind, um die Technologie anhand eines echten Contracts zu bewerten.
