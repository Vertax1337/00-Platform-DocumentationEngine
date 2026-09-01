# DocumentationEngine – Collector-Schnittstelle und Verantwortungsgrenze

Stand: 2026-09-01
Status: **Logische Verantwortungsgrenze beschlossen; providerunabhängiger Core-Contract beschlossen; technischer Provider-/Exchange-Contract weiterhin offen**

## 1. Zweck

Dieses Dokument trennt die Verantwortlichkeiten zwischen Infrastruktur-Collectoren und der `DocumentationEngine`.

Es soll verhindern, dass dieselbe Erfassungs-, Normalisierungs- oder Dokumentationslogik in mehreren Repositories parallel entsteht.

---

## 2. Grundmodell

```text
Hersteller-/Plattformquelle
        |
        v
      Collector
        |
        v
Normalisierte / validierte Fakten
        |
        v
DocumentationEngine
        |
        v
Technikerorientierte Dokumentation
```

Die `DocumentationEngine` ist **kein Collector**.

---

## 3. Verantwortlichkeit des Collectors

Ein Collector ist verantwortlich für:

- Zugriff auf seine jeweilige technische Quelle,
- Read-only-/Least-Privilege-Sicherheitsgrenzen,
- Parsing,
- Normalisierung,
- Datenminimierung,
- Secret-/PII-Filterung soweit Teil des Collector-Konzepts,
- stabile technische Identifikatoren,
- explizite Relationships,
- schema-stabile Ausgabe,
- technische Validierung des erfassten Iststands.

Beispiel `AzureInfrastructureCollector`:

- Azure Resource Graph / freigegebene Azure-Lesepfade,
- Resource Groups / Core,
- Netzwerk,
- Compute,
- AVD,
- Storage,
- Backup,
- Key Vault,
- spätere fachliche Collector-Domänen.

Die DocumentationEngine soll diese Azure-Daten **nicht erneut live aus Azure beschaffen**.

---

## 4. Verantwortlichkeit der DocumentationEngine

Die DocumentationEngine ist verantwortlich für:

- Input-Contract-Validierung,
- Zusammenführung mehrerer freigegebener Collector-Quellen,
- herstellerübergreifendes internes Modell,
- semantische Gruppierung,
- Techniker-View-Modelle,
- Texte und Tabellen,
- Diagram View Models,
- Dokument-Templates,
- deterministisches Rendering,
- Ausgabeformate,
- Qualitätsprüfung der Dokumentationsartefakte.

Sie darf normalisierte Fakten strukturieren und deterministisch ableiten, aber nicht stillschweigend neue Infrastruktur-Fakten erzeugen.

Der globale providerunabhängige Core-Contract ist in [`docs/architecture/CANONICAL_MODEL.md`](architecture/CANONICAL_MODEL.md) definiert.

---

## 5. AzureInfrastructureCollector – aktuell bekannte Eingangsdomänen

Der derzeit real validierte Azure-Collector arbeitet modular und erzeugt mehrere Fachdateien statt einer einzelnen monolithischen `inventory.json`.

Aktuell bekannte normalisierte Bereiche umfassen insbesondere:

```text
Inventory/
|- resourceGroups.json
|- resources.json
|- network.json
|- compute.json
|- avd.json
|- storage.json
|- backup.json
`- keyVault.json

summary.json
manifest.json
readOnlyVerification.json
```

Dieser Stand ist eine wichtige Erkenntnis für den späteren technischen Input-Contract der DocumentationEngine.

### Noch nicht beschlossen

Es ist noch offen, ob die Engine:

- dieses Artefaktpaket direkt als Contract übernimmt,
- einen vorgelagerten Aggregator erhält,
- ein eigenes versioniertes Exchange-Schema erhält,
- mehrere Collector-Pakete in einem Build verarbeitet.

Diese Entscheidung wird nicht in diesem Dokument vorweggenommen.

---

## 6. Relationships als bevorzugte Faktenbasis

Die DocumentationEngine soll explizite Relationships bevorzugen.

Beispiele aus dem Azure-Collector:

```text
VNet -> ContainsSubnet -> Subnet
VNet -> PeeredWith -> VNet
Subnet -> SecuredBy -> NSG
VM -> UsesNetworkInterface -> NIC
VM -> UsesOsDisk -> Managed Disk
Session Host -> BackedByVm -> VM
Application Group -> UsesHostPool -> Host Pool
Protected Item -> UsesBackupPolicy -> Backup Policy
Protected Item -> ProtectsResource -> Azure Resource
```

Diese Beziehungen eignen sich für deterministische Dokumentationsaussagen und Diagrammkanten.

### 6.1 Abhängigkeit zur P9 Relationship Engine

**Status:** BESCHLOSSENE CONTRACT-GRENZE / globaler Core spezifiziert / P9 IM COLLECTOR NOCH OFFEN

P3, P4, P5 und P6 des `AzureInfrastructureCollector` verwenden derzeit teilweise fachmodulspezifische Relationship-Arrays. Diese Strukturen sind als aktuelle Faktenquelle und für die Anforderungsanalyse nutzbar, bilden aber noch nicht den finalen produktiven Azure→DocumentationEngine-Relationship-Contract.

Die in P9 vorgesehene `Relationship Engine` vereinheitlicht die Azure-seitigen Relationships. Erst das stabilisierte und versionierte P9-Schema bildet die Grundlage für den produktiven Azure-Relationship-Contract und den Azure-Provider-Adapter.

P9 definiert dabei ausschließlich den kanonischen Azure-seitigen Input. Das globale providerübergreifende Infrastructure-/Relationship-Modell, die Diagrammsemantik und die Anforderungen der DocumentationEngine werden zentral in der DocumentationEngine festgelegt.

Dieser globale Core ist inzwischen mit `InfrastructureNode`, `Relationship`, `EvidenceReference` und `CoverageRecord` fachlich spezifiziert. Provider-Typen und Provider-Relationship-Typen bleiben dabei erhalten und werden kontrolliert auf kanonische Semantik abgebildet.

Damit gilt:

```text
fachmodulspezifische Azure-Relationships
        -> Collector P9 Relationship Engine
        -> kanonischer Azure-Relationship-Input
        -> Azure-Provider-Adapter
        -> Canonical Infrastructure Model
        -> Semantic View Builder
```

Die Spezifikation des globalen Engine-Modells ist kein P9-Blocker. Die produktive Finalisierung des Azure-Adapters ist dagegen von P9 abhängig.

Die DocumentationEngine soll eine vorhandene Relationship nicht durch Namensheuristik ersetzen.

Details zum globalen Modell: [`docs/architecture/CANONICAL_MODEL.md`](architecture/CANONICAL_MODEL.md).

---

## 7. Resource IDs und technische Primärschlüssel

Stabile technische IDs sind für:

- Zusammenführung,
- Deduplizierung,
- Relationship-Auflösung,
- spätere Mehrquellenkorrelation,
- Diagrammvalidierung

zu bevorzugen.

Friendly Names und Anzeigenamen dienen der Präsentation, nicht als alleinige Beziehungsgrundlage.

Im Canonical Infrastructure Core werden aus stabilen Source Identities deterministische kanonische IDs gebildet. Die konkrete Stringkodierung bleibt technische Implementierungsentscheidung.

---

## 8. Umgang mit fehlenden Collector-Domänen

Die DocumentationEngine muss Datenabdeckung explizit behandeln können.

Der Canonical Core verwendet hierfür die Coverage-Zustände:

```text
collected
partial
notCollected
notApplicable
unavailable
```

Beispiel:

Wenn RBAC-/Policy-/Monitoring-Detaildaten noch nicht Teil des gelieferten Collector-Pakets sind, darf die Engine keine typische Security-/Operations-Architektur ergänzen.

Erlaubt:

```text
RBAC: nicht erhoben
Policies: nicht erhoben
Monitoring Details: nicht erhoben
```

Nicht erlaubt:

```text
Azure Firewall vorhanden
Bastion vorhanden
DDoS vorhanden
```

nur weil diese Dienste in einer Referenzarchitektur üblich wären.

---

## 9. Dokumentationsfähige Ableitungen

Die Engine darf aus expliziten Fakten deterministische Aussagen formulieren.

Beispiel:

Input:

```text
SessionHost -> BackedByVm -> AVD-GOLDEN-IMG
```

Dokumentationsaussage:

> Der Session Host wird durch die Azure-VM `AVD-GOLDEN-IMG` bereitgestellt.

Nicht zulässig ohne weitere Datenbasis:

> Die VM ist geschäftskritisch und muss innerhalb von 15 Minuten wiederhergestellt werden.

Dies wäre eine nicht belegte betriebliche Annahme.

Deterministisch abgeleitete Core-Relationships müssen ihre Quellfakten als Evidence referenzieren und als Ableitung gekennzeichnet sein. Namensähnlichkeit ist keine zulässige Ableitungsgrundlage.

---

## 10. Schnittstellen zu weiteren Collectoren

Die Architektur ist nicht Azure-exklusiv.

Spätere oder bestehende Quellen können beispielsweise sein:

- OPNsense,
- Hyper-V / On-Premises,
- Switch-/Layer-2-Daten,
- weitere Cloud-/SaaS-Plattformen.

Jeder Collector bleibt für seine technische Erfassung und Normalisierung verantwortlich.

Die DocumentationEngine soll langfristig verschiedene normalisierte Quellen in gemeinsame Techniker- und Topologiesichten überführen können.

Alle Provider-Adapter müssen denselben globalen Core-Contract bedienen; provider-spezifische Fakten werden nicht in den Renderer verschoben.

---

## 11. Verbleibender technischer Contract

Mit DE-WC-01 sind bereits fachlich festgelegt:

- providerunabhängige Node-/Relationship-Anforderungen,
- Evidence als First-Class-Provenance,
- Coverage-Grundmodell,
- No-Invention-/Fail-Closed-Grundsätze,
- referentielle Integrität und deterministische Sortierung des Canonical Core.

Vor produktiver Core-/Provider-Integration bleiben mindestens offen:

- Paketstruktur,
- konkrete technische Schema-/Serialisierungsform,
- Schema-/Modellversionierung und Kompatibilitätsstrategie,
- Collector-Typ/Provider-Kennung im physischen Exchange-Contract,
- Pflichtmetadaten des Pakets,
- Snapshot-/Erfassungszeitpunkt im Exchange-Contract,
- konkrete ID-/Namespace-Kodierung,
- nach P9: versioniertes kanonisches Azure-Relationship-Schema,
- Mappingvertrag des produktiven Azure-Provider-Adapters auf das globale Engine-Modell,
- Schema-/Security-Validation-Verantwortung,
- Kompatibilitätsregeln bei unterschiedlichen Collector-Versionen.

Diese Punkte gehören weiterhin in Phase 1 bzw. Phase 2 des kanonischen `IMPLEMENTATION_PLAN.md`.

Für Azure ist die Reihenfolge verbindlich:

1. providerunabhängigen Diagramm- und Relationship-Bedarf in der DocumentationEngine spezifizieren – **erfüllt durch DE-WC-01-Fachspezifikation**,
2. P9 im `AzureInfrastructureCollector` vereinheitlichen und stabilisieren,
3. P9-Schema als kanonischen Azure-seitigen Input prüfen und versionieren,
4. produktiven Azure-Provider-Adapter und Azure→DocumentationEngine-Relationship-Contract finalisieren.

Referenzen:

- [`docs/architecture/CANONICAL_MODEL.md`](architecture/CANONICAL_MODEL.md)
- [`AzureInfrastructureCollector / Umsetzungsplan.md`](https://github.com/Vertax1337/AzureInfrastructureCollector/blob/main/Umsetzungsplan.md)
