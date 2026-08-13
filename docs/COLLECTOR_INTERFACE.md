# DocumentationEngine – Collector-Schnittstelle und Verantwortungsgrenze

Stand: 2026-08-13
Status: **Logische Verantwortungsgrenze beschlossen; technischer Contract weiterhin offen**

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

Die DocumentationEngine soll eine vorhandene Relationship nicht durch Namensheuristik ersetzen.

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

---

## 8. Umgang mit fehlenden Collector-Domänen

Die DocumentationEngine muss Datenabdeckung explizit behandeln können.

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

---

## 11. Noch offener technischer Contract

Vor Implementierung der Core Engine müssen mindestens festgelegt werden:

- Paketstruktur,
- Schema-Versionierung,
- Collector-Typ/Provider-Kennung,
- Pflichtmetadaten,
- Snapshot-/Erfassungszeitpunkt,
- IDs und Namespaces,
- Relationship-Schema,
- Fehler-/Partial-Coverage-Modell,
- Schema-/Security-Validation-Verantwortung,
- Kompatibilitätsregeln bei unterschiedlichen Collector-Versionen.

Diese Punkte gehören in Phase 1 des kanonischen `IMPLEMENTATION_PLAN.md`.
