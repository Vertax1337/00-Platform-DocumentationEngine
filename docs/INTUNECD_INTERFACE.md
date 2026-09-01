# DocumentationEngine – IntuneCD / Intune Source Interface

> **Status:** Fachlicher Consumer-/Adaptervertrag BESCHLOSSEN; technisches Snapshot-Schema und Adapterimplementierung OFFEN.  
> **Stand:** 2026-09-01

## 1. Zweck

Dieses Dokument definiert die DocumentationEngine-Seite des Intune Cross-Project Contracts.

Der plattformweite Ownership-/Provisionierungsvertrag wird in `00-Platform / PlatformBootstrap` geführt. Die DocumentationEngine dupliziert diesen Vertrag nicht, sondern spezifiziert ausschließlich, wie validierte Intune-Actual-/Desired-Artefakte in die providerunabhängigen Canonical Models überführt werden.

Zielpfad:

```text
IntuneCD / Intune Default Deployment
        ↓
BSSE Intune Snapshot / Provenance Contract
        ↓
Intune Source Adapter
        ↓
Canonical Graph [actual|desiredDeployment]
        ↓
Semantic View Builder
        ↓
Document View Model / Diagram View Model
```

## 2. Perspektiven

Verbindliche Zuordnung:

```text
IntuneCD-Backup eines realen Kunden-Tenants
→ Canonical Graph [actual]

freigegebener Stand aus 30-IDD / IntuneDefaultDeployment
→ Canonical Graph [desiredDeployment]
```

Ein Git-Commit allein definiert keine Desired-State-Perspektive. Der Input muss seine freigegebene Rolle als Desired State über den Cross-Project-/Snapshot-Contract nachweisen.

Die DocumentationEngine führt Actual und Desired nicht implizit zusammen.

## 3. Input-Grenze

Die DocumentationEngine konsumiert nicht „irgendeinen IntuneCD-Ordner“, sondern einen validierten, versionierten Source-/Snapshot-Contract mit referenzierten Provider-Artefakten.

Der Contract muss mindestens folgende Semantik verfügbar machen:

- Contract-/Schema-Version,
- CustomerNumber,
- Microsoft Entra Tenant ID,
- Snapshot-ID,
- Perspektive,
- Capture-/Build-Provenance,
- Source Tool und Version,
- immutable Repository-/Commit-Provenance,
- referenzierte Artefakte und Integritätsnachweise,
- enthaltene, ausgeschlossene und nicht verfügbare Domänen,
- Validation-/Security-/Sanitization-Status.

Die exakte physische Serialisierung dieses Contracts ist noch offen.

## 4. Provider-Artefakte

IntuneCD bleibt quellspezifischer Producer. Seine nativen JSON-/YAML-/Assignment-/Summary-/Compare-Artefakte sind Evidence-/Source-Material, aber nicht selbst das providerunabhängige Canonical Model.

Die DocumentationEngine darf ihre Canonical Semantik nicht an bloße Dateinamen oder eine bestimmte IntuneCD-Verzeichnisversion koppeln.

Bei Änderungen der nativen IntuneCD-Struktur muss der Adapter bzw. der vorgelagerte Snapshot-Contract angepasst werden, ohne die providerunabhängigen Core-Verträge unnötig zu verändern.

## 5. Stable Identity

Objektidentitäten müssen aus belegten technischen Source-IDs abgeleitet werden.

Für Intune-/Entra-Objekte gilt:

- Microsoft-Graph-/Provider-IDs werden bevorzugt als Source Object IDs erhalten,
- Dateinamen sind keine kanonische Identität,
- Anzeigenamen sind keine kanonische Identität,
- Name-only-Korrelation ist unzulässig,
- Relationships dürfen nur aus direkten oder deterministisch ableitbaren Source-Belegen entstehen.

IntuneCDs `--append-id` kann später einen stabileren Betriebs-/Dateivertrag unterstützen, ist aber keine Voraussetzung für die kanonische Identität.

## 6. Canonical-Mapping

Der bestehende providerunabhängige Core wird zunächst wiederverwendet. Intune-spezifische neue Core-Kinds werden nur eingeführt, wenn reale Source-Daten einen fachlichen Bedarf belegen.

Bereits vorhandene Core-Kinds können unter anderem für folgende Intune-Semantik verwendet werden:

```text
policy
application
identity
device
security
management
other
```

Bereits vorhandene Relationship-Kinds enthalten beispielsweise:

```text
assignedTo
memberOf
uses
dependsOn
manages
```

Ein Mapping wird erst dann verbindlich, wenn der konkrete Source-Beleg und die Semantik eindeutig sind. Der Adapter darf nicht allein aus einem ähnlich klingenden Namen einen Node-Kind oder eine Relationship ableiten.

## 7. Assignments und Relationships

Intune-Assignments sind ein zentraler provider-spezifischer Relationship-Bereich.

Ein Beispiel für eine zulässige Abbildung ist nur dann erlaubt, wenn die Source das Assignment explizit belegt:

```text
Policy / Application
        ↓ assignedTo
Entra Group / belegtes Assignment Target
```

Unaufgelöste Targets bleiben unresolved bzw. werden über Coverage/Evidence entsprechend kenntlich gemacht. Sie werden nicht aus Namen nachgebaut.

## 8. Evidence und Coverage

Jeder aus Intune erzeugte Canonical Node und jede Canonical Relationship benötigt Evidence.

Evidence muss auf den konkreten Snapshot-/Source-Kontext zurückführbar sein und mindestens den relevanten Provider-Artefaktbezug sowie die immutable Snapshot-/Commit-Provenance erhalten können.

Nicht erhobene oder ausgeschlossene Domänen werden über Coverage sichtbar gemacht und nicht als „nicht vorhanden“ interpretiert.

Beispiele:

```text
collected
partial
notCollected
notApplicable
unavailable
```

## 9. Native IntuneCD-Dokumentation

`IntuneCD-startdocumentation` kann weiterhin als native Intune-Spezial-/Technikeransicht genutzt werden.

Verbindlich gilt jedoch:

```text
native IntuneCD Markdown/HTML
≠ Canonical Source of Truth
≠ Document View Model
```

Die DocumentationEngine parst diese native Markdown-Dokumentation nicht als technische Eingangsquelle für das Canonical Model.

## 10. IntuneCD Compare

`IntuneCD-startcompare` kann als provider-spezifische Compare-/Evidence-Hilfe genutzt werden.

Sein Ergebnis ersetzt nicht den providerunabhängigen Reconciliation-Contract der DocumentationEngine.

Der spätere Reconciliation-Pfad bleibt:

```text
Canonical Graph [actual]
        +
Canonical Graph [desiredDeployment]
        ↓
explizite Reconciliation
        ↓
matched / desiredMissing / actualUnmanaged / unresolved / belastbarer propertyDrift
```

Name-only-Korrelation ist auch hier unzulässig.

## 11. Security-Grenze

Secret- oder hochsensitive Werte dürfen nicht ungeprüft in Canonical Graph, Evidence, Logs, Diagramme oder Dokumentartefakte übernommen werden.

Der vorgelagerte Snapshot-/Security-Contract muss nachweisen können, welche Provider-Artefakte validiert und sanitizt wurden.

Die konkrete Verantwortungsabgrenzung zwischen Intune Producer, `SecurityValidation` und Engine-eigener Contract-Validation bleibt technisch offen.

## 12. Nicht durch diesen Vertrag entschieden

Noch offen bleiben:

- exakte Snapshot-Dateibenennung,
- JSON-/YAML-/sonstige Schema-Technologie,
- technische Serialisierung des Canonical Core,
- konkrete IntuneCD-Version/Fork-Strategie,
- Monitor-Runtime und Azure-Repos-Authentifizierung,
- Microsoft-Graph-Permissions,
- exakte Intune-Objekt-/Property-Mappings,
- Umfang initial unterstützter Intune-Domänen,
- Property-Level-Driftsemantik,
- konkrete Semantic Intune Views und Diagramme,
- Pipeline-/Trigger-Vertrag.

## 13. Initialer Workchunk

### DE-WC-03.1 – Intune Source Adapter Contract / Fixtures

**Status:** FACHLICH BESCHLOSSEN / TECHNISCHE IMPLEMENTIERUNG OFFEN

Der erste technische Intune-Workchunk soll:

- mindestens einen kleinen validierten IntuneCD-Actual-Fixture definieren,
- mindestens einen korrespondierenden `desiredDeployment`-Fixture aus `IntuneDefaultDeployment` ermöglichen,
- Snapshot-/Provenance-Metadaten vollständig erhalten,
- stabile Source Object IDs verwenden,
- mindestens eine explizit belegte Assignment-Relationship abbilden,
- Coverage für nicht erhobene Domänen ausweisen,
- keine native Markdown-Dokumentation parsen,
- keine Name-only-Korrelation verwenden,
- keine Reconciliation implizit im Adapter durchführen.

Die konkrete Programmiersprache, Serialisierung und Parserstrategie bleiben durch diesen Workchunk unentschieden.
