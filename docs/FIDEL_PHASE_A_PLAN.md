# Fidel Phase A — Planning-Only Implementation Plan

Status: **OWNER REVIEW REQUIRED — IMPLEMENTATION BLOCKED**

Planning authority: **approved on 2026-08-30**

Implementation authority: **not granted**

Plan version: `FIDEL-PHASE-A-PLAN-1`

Parent design: `FIDEL_UNITY_IMPORTER_DESIGN.md`

## Authorization record

> AUTHORIZE FIDEL PHASE A PLANNING ONLY for an Editor-only Unity 6.3 LTS package: quarantine, hashing, format probes, archive safety, dry-run manifests, non-code native import routing, provenance ledger, and edit-mode fixture design. Exclude Unity launch, package installation, scripts/plugins, external downloads, DCC applications, Asset Store access, project migration, and implementation until the exact plan is reviewed.

Decision classification: **approved for planning only**. This document is the authorized output. It does not create a Unity project, package, script, plug-in, fixture, quarantine directory, manifest, or ledger; it does not launch Unity or another application; and it does not import, download, install, migrate, or modify any game asset.

## 1. Outcome

Phase A will produce a private, Editor-only package named **Fidel** that accepts a local file, archive, or directory into a quarantine outside Unity-controlled folders; hashes and classifies it without executing it; inventories supported archives under strict resource and path rules; produces a deterministic dry-run manifest; and, only after an exact in-tool approval, routes supported non-code content through Unity's native importer in a new isolated test project. Every observation, decision, output, and rollback result is written to a provenance ledger.

Fidel's Phase A promise is deliberately bounded:

- Universal **intake, identification, safe routing, and explanation**.
- Native import only when the input is non-code, locally available, rights-accounted, supported by Unity 6.3 LTS, and unambiguous under the project profile.
- Honest `NEEDS_ADAPTER`, `REFERENCE_ONLY`, `QUARANTINED`, or `REJECTED` results for everything else.
- No claim that arbitrary bytes can be converted into a usable Unity asset.

## 2. Frozen boundary

### Included in the later Phase A implementation

- An embedded, Editor-only Unity package with no runtime assembly.
- Local file and directory intake into an external quarantine.
- Streaming SHA-256 and deterministic directory-bundle hashes.
- Extension, signature, header, schema, and container probes.
- Bounded ZIP, GZip, and constrained TAR safety handling.
- Detection-only routing for unsupported archives such as 7z and RAR.
- Exact dry-run manifests, collision checks, and a no-overwrite transaction.
- Native non-code routing for approved models, textures, audio, video, fonts, text/data, and compatible Unity-native serialized assets.
- Append-only, tamper-evident provenance/import ledger records.
- Synthetic, inert, or locally generated Edit Mode fixtures and tests.
- Static, import, determinism, rollback, and residue evidence in the isolated project.

### Excluded from Phase A

- Runtime code, runtime loading, players, builds, Play Mode, runtime tests, and visual/audible acceptance.
- Package installation or dependency acquisition. If the Unity Test Framework is not already present and usable, implementation stops.
- Candidate scripts, shaders, assemblies, managed/native plug-ins, postprocessors, custom importers, package hooks, or other executable content.
- UPM, Git, registry, `.tgz`, and `.unitypackage` installation or execution.
- Custom `ScriptedImporter` or `AssetPostprocessor` implementations.
- Network access, source connectors, remote URLs, external downloads, telemetry, update checks, or cloud services.
- Asset Store login, browsing, downloads, licenses, or content.
- Blender, Maya, 3ds Max, Photoshop, command-line converters, codecs, antivirus tools, or any other external/DCC application launch by Fidel.
- Direct import of DCC-native model files such as `.blend`, `.ma`, `.mb`, `.max`, `.c4d`, or `.skp` when an external application is required.
- Existing production projects, project migration, render-pipeline conversion, source-control operations, or donor-project analysis.
- Overwriting or deleting owner assets. Rollback may remove only files created by the same Fidel test/import job in its exact job-owned destination.
- Mirroring, scraping, training on, or redistributing Unity documentation, Unity Learn, or Asset Store corpora.

Any excluded capability requires a later, separately scoped phase and fresh owner authorization.

## 3. Recommended implementation baseline

These values form one reviewable default bundle. They are recommended, not yet approved for execution.

| Parameter | Exact Phase A recommendation | Rule |
|---|---|---|
| Unity Editor | `6000.3.23f1` | Pin exactly. Unity lists this patch as released August 26, 2026. Do not silently move to another patch. |
| Unity line | Unity 6.3 LTS | Unity identifies 6.3 as the latest LTS line and supports it through December 2027. |
| Test project | `<FIDEL_WORKSPACE>/work/Fidel-Phase-A-6000.3.23f1` | Must be a new, isolated project. Never point Phase A at a production project. |
| Template/pipeline | Empty 3D project using the Built-in Render Pipeline | Avoids SRP package installation; Fidel core remains pipeline-neutral. |
| Editor host | Windows x64 | Phase A acceptance is for the named Editor host only. macOS/Linux validation is a later compatibility lane. |
| Player targets | None | Editor-only package; no player build or target-platform test. |
| Package ID | `com.fidel.importer` | Private embedded-package ID for the sandbox. Replace with an owner-controlled reverse-domain ID before public distribution. |
| Display name | `Fidel Importer` | Stable user-facing identity. |
| Root namespace | `Fidel.Importer` | Editor assembly and test assembly use explicit namespaces. |
| Package placement | `<test-project>\Packages\com.fidel.importer` | Embedded local package; no registry or Git install. |
| Product quarantine | `%LOCALAPPDATA%\Fidel\Quarantine\<project-id>\<job-id>` | Derived through the platform API, never by trusting an unresolved environment string. Outside `Assets` and `Packages`. |
| Test quarantine override | `<FIDEL_WORKSPACE>/work/Fidel-Quarantine` | Keeps the test lane bounded and inspectable. It must remain outside the Unity project. |
| Import destination | `Assets/FidelImports/<safe-source-name>-<sha256-first-12>/` | New directory per manifest. No overwrite or merge in Phase A. |
| Package dependencies | None added by Fidel | Use only APIs shipped with the pinned Editor. Existing Test Framework may be referenced only after preflight. |
| Network policy | Deny | No HTTP, sockets, package resolution, telemetry, update checks, or remote sources. |

Official version evidence: [Unity 6000.3.23f1 release notes](https://unity.com/releases/editor/whats-new/6000.3.23f1) and [Unity 6 release support](https://unity.com/releases/unity-6/support).

## 4. Product workflow and states

```text
LOCAL INPUT
  -> QUARANTINED
  -> IDENTIFIED
  -> POLICY EVALUATED
  -> DRY-RUN MANIFEST
  -> READY_WITH_APPROVAL
  -> OWNER APPROVES THAT MANIFEST
  -> NATIVE IMPORT TRANSACTION
  -> IMPORT VALIDATION
  -> VERIFIED or ROLLED_BACK
```

Alternative terminal routes are `NEEDS_ADAPTER`, `REFERENCE_ONLY`, and `REJECTED`. Phase A never turns one of those states into an import by guessing.

`EditMode` is a test execution context, not an evidence class. A test result must still identify its evidence as `STATIC` or `IMPORT`; Phase A produces no `RUNTIME` or `VISUAL/AUDIBLE` evidence.

## 5. Planned package topology

No files in this topology exist yet.

```text
Packages/com.fidel.importer/
  package.json
  CHANGELOG.md
  LICENSE.md
  README.md
  Editor/
    Fidel.Importer.Editor.asmdef
    Configuration/
    Contracts/
    Intake/
    Hashing/
    Probes/
    Archives/
    Planning/
    Routing/
    Transactions/
    Validation/
    Ledger/
    UI/
  Tests/
    EditMode/
      Fidel.Importer.EditMode.Tests.asmdef
      Unit/
      Integration/
      Golden/
    Fixtures/
      Manifest/
      Provenance/
      README.md
  Documentation~/
    PhaseA.md
    Security.md
    ErrorCatalog.md
```

There is intentionally no `Runtime/` folder or runtime assembly. Binary fixtures that could be interpreted as code are not stored in the package. Adversarial fixtures are inert byte patterns, path strings, and containers that cannot execute.

Project-owned data after a later authorized implementation:

```text
ProjectSettings/Fidel/
  project-profile.json
  policy.json
  source-registry.json
  import-ledger.jsonl
Library/Fidel/
  Reports/
  TestResults/
```

`Library/Fidel` is disposable evidence/cache output. `ProjectSettings/Fidel` is reviewable project policy and ledger state. Neither becomes an imported asset.

## 6. Core contracts

The builder must implement these contracts before UI work:

| Contract | Required content |
|---|---|
| `CandidateSnapshot` | Original display name, safe source label, byte length, source file facts, quarantine-relative path, SHA-256, sidecars, and stability result. |
| `BundleSnapshot` | Canonically sorted relative files, per-file SHA-256, total bytes, file count, rejected links/reparse points, and bundle root hash. |
| `ProbeEvidence` | Probe ID/version, bytes examined, claimed extension, observed signature/schema/container, confidence enum, mismatches, and citations. |
| `ArchiveInventory` | Archive type, entry facts, safe normalized path, sizes, method, encryption/link flags, nested depth, aggregate counters, and policy findings. |
| `PolicyDecision` | Rights status, executable-content status, version/platform requirements, decision state, stable findings, and exact remedy. |
| `DryRunManifest` | Immutable inputs, proposed destinations, adapter/native route, importer profile, conflicts, intended mutations, validations, rollback scope, and decision hash. |
| `ImportResult` | Manifest decision hash, actual outputs, GUIDs, importer types/settings, logs, validation findings, residue check, and rollback result. |
| `LedgerRecord` | Schema version, event type, prior-record hash, record hash, project/package/rule versions, evidence references, decision actor/type, and privacy-safe source metadata. |

All public result types use stable enums and error codes. Free-form text explains a result but never determines behavior.

## 7. Work breakdown

| ID | Work package | Depends on | Required output | Completion check |
|---|---|---|---|---|
| `A0` | Exact preflight and owner freeze | Reviewed plan | Editor/project/package/quarantine/test contract | All paths resolve inside the approved sandbox; no production project; exact Editor patch matches. |
| `A1` | Editor-only package scaffold | `A0` | Embedded package, Editor assembly, tests assembly, documentation shell | Compiles with no runtime assembly and no new dependency. |
| `A2` | Configuration, contracts, errors | `A1` | Versioned JSON schemas, limits, states, errors, canonical serialization rules | Golden serialization is stable across repeated runs and cultures. |
| `A3` | Quarantine and SHA-256 | `A2` | File/directory snapshotter, atomic quarantine copy, cancellation and residue handling | Source-change, partial-copy, duplicate-content, and large-stream fixtures pass. |
| `A4` | Probe registry | `A2`, `A3` | Ordered bounded probes and evidence merger | Extension/signature/schema matches and mismatches return deterministic states. |
| `A5` | Archive safety | `A3`, `A4` | ZIP/GZip/constrained-TAR inventory and extraction-to-quarantine only | Every archive escape/resource/link/encryption fixture blocks before a write outside the job root. |
| `A6` | Policy and dry-run manifest | `A2`–`A5` | Strict policy engine, destination planner, conflict report, canonical manifest | Same inputs/profile produce byte-identical decision material and no project write. |
| `A7` | Non-code native router | `A6` | Allowlisted Unity importer routes and fail-closed executable/DCC routes | Every supported family selects exactly one native route; forbidden inputs never enter `Assets`. |
| `A8` | No-overwrite transaction and rollback | `A7` | Job-owned destination transaction, actual-output inventory, rollback | Failed import removes only its own new destination and leaves zero job residue. |
| `A9` | Provenance ledger | `A2`, `A6`, `A8` | Append-only JSONL record writer with hash chain and privacy filter | Interrupted append is detected; valid records verify; source paths/user names are not logged by default. |
| `A10` | Fidel's Desk minimal Editor UI | `A3`–`A9` | Intake, identity, plan, exact decision, and report panes | UI invokes the same tested services; no hidden alternate import path. |
| `A11` | Edit Mode fixture suite | Each package above | Pure unit, adversarial, native-import integration, determinism, rollback, and residue tests | Mandatory matrix passes twice from a clean isolated project. |
| `A12` | Documentation and evidence bundle | `A11` | Operator guide, error catalog, source citations, test results, logs, package diff, hashes | Independent QA can reproduce findings without oral context. |

Implementation order is the table order. UI does not precede the safety core, and native import does not precede a deterministic manifest.

## 8. Quarantine and hashing contract

### 8.1 File intake sequence

1. Accept only an owner-selected local filesystem path. Phase A has no URL field.
2. Resolve the absolute path once; reject missing paths, device paths, named pipes, alternate data streams, and reparse points.
3. Record source length and last-write facts for stability detection, but exclude timestamps from the deterministic content identity.
4. Create one exact job directory beneath the approved quarantine root and verify its canonical path remains under that root.
5. Stream the source once into a job-owned `.partial` file while computing SHA-256. Never load the complete candidate into memory.
6. Use restrictive file sharing when possible. If the source cannot be held stable, return `FIDEL-INT-002 SOURCE_NOT_STABLE`.
7. Flush the quarantined copy, compare source facts, and compute the stored copy's SHA-256 independently.
8. If source and stored hashes or lengths differ, delete only the job-owned partial file, retain a report, and stop.
9. Atomically rename the verified partial file within the same job directory.
10. Mark the snapshot complete and immutable by policy. A read-only bit is defense in depth, not the trust basis.
11. Before any later commit, rehash the quarantined copy and compare it with the approved manifest.
12. Never auto-purge quarantine. Purge is a separate, exact-target owner action outside this phase.

SHA-256 is the content-integrity primitive, grounded in NIST's [Secure Hash Standard, FIPS 180-4](https://csrc.nist.gov/pubs/fips/180-4/upd1/final). Hashing demonstrates byte identity; it does not establish safety, authorship, or license.

### 8.2 Directory intake

- Walk a directory without following symbolic links, junctions, mount points, or other reparse points.
- Convert each relative path to a slash-separated manifest form while retaining the original display path.
- Sort by ordinal code-point order after Unicode NFC normalization.
- Detect exact, case-folded, trailing-dot/space, and Unicode-normalization collisions.
- Hash every regular file. The bundle root hash is SHA-256 over canonical UTF-8 lines containing relative path, file length, and file SHA-256.
- A changed file, changed file count, or changed bundle root between plan and commit invalidates approval.

### 8.3 Default resource limits

These limits are profile values and are part of the manifest decision hash:

| Limit | Phase A default |
|---|---:|
| One input file | 8 GiB |
| Files in a directory bundle | 25,000 |
| Aggregate directory bytes | 20 GiB |
| Header/tail bytes per basic probe | 1 MiB each |
| Full structured-text parse | 32 MiB |
| Path segments | 64 |
| One path segment | 128 Unicode scalar values |
| Portable relative path | 240 Unicode scalar values |
| Cancellation polling | At least once per 1 MiB streamed |

Crossing a limit returns a stable block and remedy; it never silently truncates evidence.

## 9. Probe and classification contract

Probe order is fixed: filesystem facts, extension, leading/trailing signatures, container structure, bounded schema/lexical parse, then relationship/sidecar checks. More authoritative evidence overrides weaker evidence, but all mismatches remain visible.

| Family | Required evidence in Phase A | Native-route decision |
|---|---|---|
| PNG/JPEG/TIFF/PSD/PSB/EXR/HDR | Recognized file signature plus basic dimensions/header sanity where available | Route to Unity texture importer if type is supported by pinned Editor. |
| TGA/DDS and weak-signature images | Extension plus bounded structural validation | Ambiguous or malformed input remains quarantined. |
| WAV/AIFF/Ogg/FLAC/MP3 | Container/codec signature and bounded header facts | Route to Unity audio importer if supported. |
| MP4/MOV/WebM/AVI | Container signature and basic track/container facts where safely available | Route to Unity video importer; no playback or external codec launch. |
| FBX | Binary FBX signature or bounded ASCII FBX grammar evidence | Route to Unity model importer. |
| OBJ | Bounded text parse with recognized statements and valid numeric culture | Route to Unity model importer with discovered MTL/texture sidecars. |
| DAE | XML root/namespace with DTD and external entities disabled | Route if the pinned Editor supports it without DCC. |
| TTF/OTF/TTC | `sfnt`/OpenType collection signatures and table-directory bounds | Route to Unity font importer; rights evidence remains mandatory. |
| JSON/XML/CSV/text/bytes | Encoding, BOM, safe parser/schema facts, and size limit | Route to `TextAsset`; no deserialization into arbitrary types. |
| Unity YAML/native non-code | Unity YAML markers, class IDs, GUID references, `.meta`, version/source facts | Route only when compatible and not dependent on candidate code. |
| ZIP/GZip/TAR | Signature plus safe container inventory | Extract only inside quarantine, then recursively classify. |
| 7z/RAR | Signature identification only | `NEEDS_ADAPTER`; never shell out. |
| DCC-native model | Recognized extension/signature | `NEEDS_ADAPTER` with interchange-format remedy. |
| Shader/code/package/plug-in | Extension, signature, assembly/container inventory | `REJECTED` for Phase A with `EXECUTABLE_CONTENT_OUT_OF_SCOPE`. |
| Unknown or conflicting | No sufficient evidence or evidence mismatch | `NEEDS_ADAPTER`, `QUARANTINED`, or `REJECTED`; never guess. |

XML parsing prohibits DTDs and external entities. JSON and CSV parsing are data-only. Phase A contains no generic object deserializer and loads no candidate assembly.

## 10. Archive safety contract

### 10.1 Supported behavior

- **ZIP:** full bounded inventory and extraction of regular files/directories only using managed APIs; encrypted entries and unsupported methods block.
- **GZip:** unwrap one bounded stream in quarantine and re-probe the result. Concatenated/ambiguous members block unless explicitly supported and fixture-tested.
- **TAR/TAR.GZ:** constrained reader for regular POSIX ustar files/directories. PAX/GNU extensions, sparse files, links, device nodes, or unknown type flags block with `NEEDS_ADAPTER` until separately implemented and tested.
- **7z/RAR/other archives:** identify and report only. No external utility, library download, or extraction.

### 10.2 Archive limits

| Limit | Phase A default |
|---|---:|
| Entries | 25,000 |
| One expanded entry | 4 GiB |
| Total expanded bytes | 20 GiB |
| Compression ratio | 200:1 after the compressed input exceeds 1 MiB |
| Nested archive depth | 2 |
| Free-space reserve | Declared/allowed expansion plus 2 GiB |

Declared sizes are not trusted. Fidel maintains live counters while streaming and stops before crossing a limit. This addresses the data-amplification class described by [MITRE CWE-409](https://cwe.mitre.org/data/definitions/409.html).

### 10.3 Path and entry blocks

Before writing an entry, Fidel rejects:

- absolute, drive-qualified, UNC, device, or root-relative paths;
- `..`, NUL, alternate data stream syntax, or unexpected colon characters;
- Windows device names such as `CON`, `PRN`, `AUX`, `NUL`, `COM1`, or `LPT1`, including extension variants;
- trailing dots/spaces and empty/dot-only segments;
- symbolic links, hard links, junctions, mount points, FIFOs, sockets, or device entries;
- exact, case-insensitive, Unicode-normalization, or file-vs-directory collisions;
- paths whose canonical destination does not begin with the exact canonical job extraction root plus a separator;
- encrypted entries, corrupted headers/central directories, unsupported methods, or size contradictions;
- nested executable content when the enclosing archive was proposed for native import.

Path containment is checked before every create and again after parent resolution. [MITRE CWE-22](https://cwe.mitre.org/data/definitions/22.html) is the governing weakness class.

Archive extraction is never directly into `Assets`, `Packages`, the project root, or a user-selected arbitrary destination.

## 11. Dry-run manifest contract

Every import-capable job must first write a canonical JSON manifest and enter `READY_WITH_APPROVAL`. The manifest is a proposal, not authority.

Required top-level fields:

```json
{
  "schemaVersion": 1,
  "decisionHash": "sha256-of-canonical-decision-material",
  "project": {},
  "package": {},
  "profile": {},
  "inputs": [],
  "probeEvidence": [],
  "archiveInventory": [],
  "provenance": {},
  "policyDecision": {},
  "route": {},
  "destinations": [],
  "importerSettings": [],
  "conflicts": [],
  "intendedMutations": [],
  "validations": [],
  "rollback": {},
  "unsupported": []
}
```

Canonicalization rules:

- UTF-8 without BOM and LF line endings.
- Property order defined by schema; arrays sorted by their documented stable key.
- Numbers use invariant culture; no locale-sensitive parsing or formatting.
- Paths use manifest-relative forward slashes where possible.
- Decision material contains no generated time, random ID, user name, machine name, quarantine absolute path, or process ID.
- Observation timestamps and local report paths may exist in a non-decision envelope or ledger record.
- The `decisionHash` covers exact input hashes, profile/rules/package/Unity versions, destinations, settings, conflicts, validations, and rollback scope.
- Any change to covered material invalidates the prior approval.

Phase A collision policy is **fail closed, no overwrite, no merge**. The proposed destination is a new hash-suffixed directory. Existing `.meta` GUIDs are preserved only when a valid sidecar is part of the approved input, the GUID does not already exist in the target project, and all references remain within an approved closure. Fidel never invents a replacement GUID silently.

## 12. Native non-code routing and transaction

Fidel prefers Unity's native importers documented in the [Unity asset import workflow](https://docs.unity3d.com/6000.3/Documentation/Manual/import-assets.html) and [supported asset type reference](https://docs.unity3d.com/6000.3/Documentation/Manual/assets-supported-types.html). Phase A does not implement a custom importer or postprocessor.

### 12.1 Allowed Phase A native lanes

- Interchange models such as supported FBX, OBJ, and DAE that require no external application.
- Supported textures and images.
- Supported audio and video containers.
- Supported font files with explicit rights evidence.
- Text/data imported as inert `TextAsset` content.
- Compatible Unity-native non-code files with valid `.meta`/GUID closure and no required candidate scripts.

### 12.2 Blocked lanes

- `.cs`, `.asmdef`, `.asmref`, `.dll`, `.exe`, `.so`, `.dylib`, `.bundle`, `.aar`, `.jar`, shaders, compute code, native source, or any file that Unity may compile or load as code.
- `.unitypackage`, UPM package, Git package, registry package, or full Unity project.
- DCC-native model formats whose Unity route launches an authoring tool or conversion process.
- A Unity serialized asset containing unresolved `m_Script` dependencies or references outside the approved closure.
- An input with unknown rights status, a probe/signature conflict, a changed hash, or an ambiguous destination/settings decision.

### 12.3 Commit sequence for a later authorized implementation

1. Verify the project path, Editor patch, package hash, clean fixture baseline, quarantine root, and manifest decision hash.
2. Rehash every quarantined input and sidecar.
3. Re-run collision, free-space, executable-content, and provenance gates.
4. Create the exact new destination directory; no existing owner path is reused.
5. Use a guarded `AssetDatabase.StartAssetEditing`/`StopAssetEditing` block with guaranteed `finally` cleanup only for the approved non-code copy set.
6. Copy the approved bytes and sidecars; then let Unity import the exact set.
7. Obtain each `AssetImporter`, apply only the settings recorded in the manifest, and reimport when required.
8. Record actual assets, sub-assets, importer types/settings, `.meta` GUIDs, dependencies, warnings, and errors.
9. Run family and universal validators.
10. Compare actual destinations with the approved set. An unexplained delta fails.
11. On failure, roll back only the job-created destination through `AssetDatabase`, verify its resolved path and ownership marker, refresh, and assert zero job residue.
12. Append the result and rollback state to the ledger.

Unity's `.meta`/GUID rules are authoritative: [Asset metadata](https://docs.unity3d.com/6000.3/Documentation/Manual/AssetMetadata.html) and [Asset Database](https://docs.unity3d.com/6000.3/Documentation/Manual/AssetDatabase.html).

## 13. Provenance ledger contract

`ProjectSettings/Fidel/import-ledger.jsonl` is append-only during normal operation and is not an imported asset. Each canonical line contains:

- schema and event version;
- event type: `INTAKE`, `PLAN`, `OWNER_DECISION`, `IMPORT_RESULT`, `VALIDATION`, or `ROLLBACK`;
- previous canonical record hash and current record hash;
- project ID without exposing the absolute project path;
- exact Unity, Fidel package, rule-set, and profile versions/hashes;
- input content hashes and privacy-safe source label;
- source publisher/item ID/rights label/evidence reference when supplied;
- manifest decision hash and decision disposition;
- actual output relative paths/GUIDs/hashes;
- validation evidence references and rollback status;
- timestamp in the observation envelope, not deterministic decision material.

Ledger rules:

- One writer per project; concurrent append attempts fail and retry only after re-reading the tail.
- Write one complete line to a same-directory temporary file, flush, then append under an exclusive lock; validate the new tail before success.
- A torn or malformed final line blocks new appends until an explicit repair plan preserves the original bytes and documents the correction.
- Hash chaining is tamper-evident, not a cryptographic signature and not an immutable external audit service.
- Do not record Windows user names, machine names, credentials, tokens, full local source paths, or Unity account information by default.
- Rights `UNKNOWN` blocks import. Government/university affiliation alone never implies public-domain or commercial permission.
- Store compact rule statements and citations, not copied documentation or asset corpora.

## 14. Edit Mode fixture design

All mandatory fixtures are generated locally, authored for Fidel, inert, or created by the isolated Unity test project. No Asset Store content, downloaded sample, university dataset, government asset, system font redistribution, or third-party game asset is bundled.

### 14.1 Fixture classes

| Class | Mandatory fixtures | Expected evidence/result |
|---|---|---|
| Quarantine/hash | Empty file; chunk-boundary file; same bytes/different names; source-change simulation; cancellation; directory sort/collision cases | Stable SHA-256/bundle hash; changed or partial source blocks; no `.partial` residue. |
| Images | Generated valid 1x1 PNG; renamed PNG-as-JPG; truncated PNG; oversized declared dimensions | Valid route; signature mismatch block; malformed block; resource block. |
| Model | Authored minimal OBJ/MTL; missing MTL; traversal sidecar reference; bounded FBX signature samples | Native route or exact missing/unsafe dependency finding; no DCC launch. |
| Audio | Generated PCM WAV; truncated RIFF; declared-size contradiction | Native route or malformed block. |
| Text/data | UTF-8 JSON; UTF-8 BOM; invalid UTF-8; XML with prohibited DTD/entity; locale-sensitive numeric CSV | Inert `TextAsset` route or exact encoding/schema/security finding. |
| Font | Synthetic bounded `sfnt` table-directory samples for probes; optional machine-local font import test with no redistribution | Probe correctness is mandatory; valid native import is conditional on an owner-supplied or locally licensed fixture. |
| Unity native | Test-generated Material/AnimationClip plus `.meta`; duplicate GUID; missing `.meta`; serialized object with unresolved script reference | Preserved compatible closure or exact GUID/script block. |
| Archive | Safe ZIP; traversal; absolute path; ADS/device path; symlink metadata; duplicate/case/Unicode collision; encrypted entry; corrupt central directory; ratio/size/count/depth limits; TAR link/type block; nested `.cs` name | Safe quarantine-only extraction or stable block; never a write outside job root. |
| Manifest | Same inputs under different culture/time; reordered directory enumeration; changed profile; destination collision | Byte-identical canonical decision material when semantically identical; changed decision hash otherwise. |
| Ledger | Clean append; concurrent writer; torn final line; hash-chain alteration; privacy-sensitive source path | Verified chain or fail-closed repair requirement; no prohibited local identity fields. |
| Transaction | Successful native import; importer error; unexpected output; forced cancellation; rollback | Exact output set or rollback of only job-owned destination; zero residue. |

Font integration is the one conditional lane because a redistributable valid font cannot be assumed. It may not be marked `IMPORT`-verified until a rights-clear fixture is supplied or separately authorized for acquisition. This exception is visible; it is not converted into a pass.

### 14.2 Adversarial-fixture safety

- No real malware, runnable assembly, valid plug-in, exploit, or weaponized payload.
- “Executable content” fixtures use filenames and inert/truncated signatures sufficient to test blocking.
- Archive-bomb tests simulate or stream to low test-only thresholds; they do not create multi-gigabyte outputs.
- All extraction roots and Unity scratch destinations are unique job-owned directories.
- Cleanup resolves and verifies the exact path beneath the approved fixture root before removing job-created files.
- A failed cleanup is a failed test and stops the suite; it is never hidden by a second broad cleanup.

### 14.3 Planned test partitions

1. **Pure Edit Mode unit tests:** canonical JSON, hashing, path policy, probes, archive inventory, policy, and ledger validation. They do not write to `Assets`.
2. **Quarantine integration tests:** write only beneath the approved test quarantine, then verify partial/residue rules.
3. **Native import integration tests:** write only beneath `Assets/__FidelTestScratch/<run-id>` or the manifest's new `Assets/FidelImports/...` destination in the isolated project.
4. **Determinism pass:** repeat relevant tests under a second culture and randomized source enumeration; compare canonical bytes and hashes.
5. **Rollback/residue pass:** force failure after each transaction boundary and verify only job-owned outputs are removed.

The future planned command is:

```text
<UNITY_6000_3_23F1_EXE> -batchmode -nographics -quit -projectPath "<APPROVED_TEST_PROJECT>" -runTests -testPlatform EditMode -testResults "<EVIDENCE_ROOT>/Fidel-EditMode.xml" -logFile "<EVIDENCE_ROOT>/Fidel-EditMode.log"
```

It is not authorized or executed by this planning phase. If the existing pinned Editor/Test Framework does not support the planned invocation, implementation stops and returns a compatibility finding; it does not install or upgrade anything.

Unity describes its Test Framework as supporting Edit Mode tests: [Unity Test Framework manual](https://docs.unity3d.com/6000.3/Documentation/Manual/com.unity.test-framework.html).

## 15. Validation and acceptance matrix

| Gate | Owner | Required evidence | Pass condition |
|---|---|---|---|
| `G0 Scope/Preflight` | Owner + Builder | Exact path/version/package/quarantine/profile record | Recommended baseline accepted or replaced explicitly; no production project; no excluded action required. |
| `G1 Static package` | Builder | Package tree, dependency diff, compilation log, API inventory | Editor-only assembly; no runtime assembly, network API, process launch, custom importer/postprocessor, or added package dependency. |
| `G2 Safety core` | QA | Mandatory hash/probe/archive/path/policy fixtures | All expected results match; no escape, execution, truncation, or unexplained warning. |
| `G3 Native import` | QA | Edit Mode XML/log, manifests, actual output inventory, GUID/dependency report | All mandatory native fixtures import only to approved paths; actual set equals planned set. |
| `G4 Determinism/rollback` | QA | Two-run canonical hashes, forced-failure matrix, residue inventory | Decision material is byte-stable; every forced failure leaves zero job residue and no owner file change. |
| `G5 Security review` | Independent security reviewer | Threat checklist, code/API review, archive/path findings, limits, privacy review | No unresolved high-risk finding; exclusions are technically enforced, not merely documented. |
| `G6 Judge` | Independent Judge | Plan, builder evidence, QA/security reports, known exceptions | `GO` only for the exact Phase A acceptance contract; font import exception remains explicitly scoped if unresolved. |
| `G7 Owner disposition` | Owner | Consolidated report and one recommended next action | Owner accepts, requests correction, or stops. No production integration follows implicitly. |

For this planning deliverable, only an author self-check is available. It must not be represented as independent QA, independent security review, or Judge approval.

## 16. Phase A definition of done

The later implementation is complete only when all of the following are true:

1. The exact baseline in Section 3 is owner-approved and recorded.
2. Fidel has no runtime assembly and adds no package dependency.
3. Every candidate remains outside `Assets`/`Packages` until its exact manifest is approved.
4. SHA-256 and directory bundle hashes are streaming, reproducible, and rechecked before commit.
5. Probe evidence is versioned and cannot be replaced by extension-only guesses for containers or risky types.
6. Archive extraction passes all containment, collision, link, encryption, nesting, size, count, ratio, and residue fixtures.
7. Unknown archives, DCC-native files, packages, scripts, plug-ins, shaders, and executable content fail closed.
8. Canonical dry-run decision material is byte-identical for identical inputs/profile and changes whenever an approval-relevant fact changes.
9. Phase A never overwrites or merges an owner asset.
10. Unity-native imports use the pinned Editor's native importers and exact approved settings.
11. Actual outputs, sub-assets, `.meta` GUIDs, dependencies, warnings, errors, and rollback are recorded.
12. Compatible existing `.meta`/GUID relationships are preserved; conflicts block.
13. The provenance ledger verifies its hash chain and omits prohibited identity/credential fields.
14. Mandatory Edit Mode fixtures pass twice from a clean isolated project with zero job residue.
15. No network call, external process, Asset Store action, DCC launch, package install, migration, Play Mode, or production-project mutation occurred.
16. Builder, independent QA, security, Judge, and owner dispositions remain separately labeled.

## 17. Stop conditions and correction policy

Implementation must stop immediately if:

- the exact Editor patch, project path, package ID, or quarantine root differs from the approved record;
- the project is not isolated or contains unrelated owner work;
- a package/dependency installation or network call becomes necessary;
- Fidel or a candidate attempts to start an external process;
- executable content is found in an approved non-code set;
- any input hash changes after approval;
- any canonical path escapes its approved root or cannot be proven contained;
- archive limits, disk reserve, collision policy, or provenance requirements fail;
- a job would overwrite, merge with, or delete pre-existing owner content;
- a test leaves residue outside its exact job-owned root;
- Unity reports a compile/import failure that invalidates later evidence;
- the Test Framework is absent/incompatible and proceeding would require installation;
- the scope would expand to production integration, migration, packages, code-bearing candidates, external acquisition, runtime, or visual acceptance.

Ordinary defects receive at most two bounded builder correction cycles against the same acceptance contract before the work returns to the owner with the repeated cause. Security-boundary failures receive no automatic retry.

## 18. Evidence bundle for review

The later builder must hand off one self-contained evidence directory containing:

- exact baseline/preflight record;
- package tree and hashes;
- package/dependency/project diff;
- compiled API inventory showing Editor-only and prohibited-API checks;
- fixture catalog with authorship/rights labels;
- canonical manifest golden files and hashes;
- Test Framework XML and complete Editor log;
- two-run determinism comparison;
- archive/path security matrix;
- actual output/GUID/dependency inventory;
- rollback and zero-residue report;
- provenance-ledger verification report;
- known limitations and explicitly unverified lanes;
- Builder, QA, security, Judge, and owner dispositions as distinct records.

Static source review is not Unity import evidence. Edit Mode import evidence is not Play Mode, runtime, visual, audible, target-platform, or production-project acceptance.

## 19. Source basis for Phase A

The plan's implementation rules are grounded in compact, versioned citations rather than copied documentation:

- [Unity 6000.3.23f1 release notes](https://unity.com/releases/editor/whats-new/6000.3.23f1)
- [Unity 6 release support](https://unity.com/releases/unity-6/support)
- [Importing assets](https://docs.unity3d.com/6000.3/Documentation/Manual/import-assets.html)
- [Supported asset types](https://docs.unity3d.com/6000.3/Documentation/Manual/assets-supported-types.html)
- [Asset Database](https://docs.unity3d.com/6000.3/Documentation/Manual/AssetDatabase.html)
- [Asset metadata and `.meta` behavior](https://docs.unity3d.com/6000.3/Documentation/Manual/AssetMetadata.html)
- [Deterministic asset import](https://docs.unity3d.com/6000.3/Documentation/Manual/build-deterministic-assets.html)
- [Parallel import](https://docs.unity3d.com/6000.3/Documentation/Manual/ParallelImport.html)
- [Unity Test Framework](https://docs.unity3d.com/6000.3/Documentation/Manual/com.unity.test-framework.html)
- [NIST FIPS 180-4 Secure Hash Standard](https://csrc.nist.gov/pubs/fips/180-4/upd1/final)
- [MITRE CWE-22 Path Traversal](https://cwe.mitre.org/data/definitions/22.html)
- [MITRE CWE-409 Data Amplification](https://cwe.mitre.org/data/definitions/409.html)

The broader official Unity, standards-body, government, and university registry remains in the parent Fidel design. Phase A does not download or bundle any source-catalog asset.

## 20. Exact next owner gate

The plan is ready for review. No implementation begins from planning acceptance alone.

If the recommended defaults are acceptable, the next authorization is:

> **AUTHORIZE FIDEL PHASE A IMPLEMENTATION ONLY in the new isolated Unity `6000.3.23f1` Built-in Render Pipeline project at `<FIDEL_WORKSPACE>/work/Fidel-Phase-A-6000.3.23f1`, using the embedded Editor-only package `com.fidel.importer`, Windows x64 Editor tests, and test quarantine `<FIDEL_WORKSPACE>/work/Fidel-Quarantine`. Authorize creation of that isolated project, Fidel's own reviewed C# package source, locally generated inert fixtures, project policy/ledger files, and Unity launch only to create/open the sandbox, compile, and run Edit Mode tests. Do not install packages, access networks or the Asset Store, launch DCC/external applications, create a custom ScriptedImporter or AssetPostprocessor, import candidate scripts/plugins/shaders/packages, enter Play Mode, build players, migrate projects, touch production projects, or continue past independent QA, security review, and Judge disposition without a fresh owner decision.**

This next gate intentionally distinguishes authoring Fidel's own reviewed Editor code from importing untrusted candidate code, which remains excluded.
