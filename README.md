# Fidel

Fidel is an Editor-only Unity 6.3 LTS import orchestrator that safely quarantines, hashes, identifies, validates, and routes non-code assets through Unity's native import pipeline—with archive defenses, deterministic dry-run manifests, provenance records, and auditable rollback.

> **Status:** Research-backed design, Phase A implementation planning, and an authorized nonfunctional Editor-only C# scaffold. No importer behavior is implemented.

## Scaffold boundary

`Packages/com.fidel.importer` contains only Unity package metadata, an Editor-only assembly definition, and an empty C# marker type. It has no runtime assembly, Unity API calls, importer, postprocessor, package dependency, network behavior, or asset-processing behavior.

## Documentation

- [Unity importer design](docs/FIDEL_UNITY_IMPORTER_DESIGN.md)
- [Phase A planning-only implementation plan](docs/FIDEL_PHASE_A_PLAN.md)

Phase A is deliberately bounded to quarantine, hashing, format probes, archive safety, dry-run manifests, non-code native routing, provenance records, and Edit Mode fixture design. Unity launch, package installation, candidate scripts or plug-ins, external downloads, DCC applications, Asset Store access, project migration, and implementation remain governed by the explicit authorization recorded in the plan.
