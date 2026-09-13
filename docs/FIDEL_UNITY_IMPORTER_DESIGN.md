# Fidel — Unity Import Orchestrator

Status: research-backed design specification

Design date: 2026-08-30

Implementation authority: **not granted**

Recommended baseline: **Unity 6.3 LTS**, with an exact patch pinned per project and an explicit compatibility lane for Unity 6.0 LTS.

## Agent-ready brief

**Goal:** Design a Unity importer named **Fidel** that can receive any file presented as Unity-related, determine what it actually is, import it correctly when a supported and trusted path exists, and otherwise stop safely with a precise explanation and remedy.

**Deliverable:** A version-aware Unity Editor package design covering intake, identification, provenance, licensing, security, import planning, native and custom adapters, validation, rollback, audit records, and a curated registry of official Unity, standards-body, government, and university sources.

**Context:** Fidel should be deeply grounded in authoritative game-development information. He must prefer Unity's own import pipeline and official packages, then formal standards, then reputable government or university sources.

**Constraints:** Do not claim that every arbitrary byte stream is importable. Do not scrape, clone, train on, or redistribute Unity documentation or Asset Store content. Do not execute untrusted scripts, plug-ins, package hooks, or native code during inspection. Do not import licensed assets without item-level rights evidence. Preserve Unity GUIDs and `.meta` relationships whenever they are valid.

**Acceptance:** Every intake ends in one explicit state: `VERIFIED`, `READY_WITH_APPROVAL`, `NEEDS_ADAPTER`, `REFERENCE_ONLY`, `QUARANTINED`, or `REJECTED`. A successful result has deterministic import evidence, preserved references, a provenance/license record, a conflict-free destination, and a reversible transaction record.

**Assumption:** Fidel is an Editor-only package for Unity 6.x. Runtime downloading/loading is a separate product and is not silently included.

## 1. The truthful product promise

Fidel is **universal at identification, routing, and explanation**, not magical at conversion.

He accepts any candidate file or directory into quarantine and answers:

1. What is it, based on extension, magic bytes, schema, and container structure?
2. Is it source content, a Unity-native asset, a package, a project, a runtime artifact, or executable code?
3. Which Unity version, render pipeline, platform, packages, and external tools does it require?
4. Is its source and license acceptable for the intended project?
5. Can Unity import it natively, can a vetted adapter import it, or must Fidel block it?
6. What will be created or changed, and can those changes be reversed?
7. Did the actual import pass semantic, reference, compile, visual, and determinism checks?

Fidel must never report “imported properly” merely because Unity created an asset. “Properly” means the asset satisfies the selected project profile and its family-specific validation contract.

## 2. Product boundaries

### Included

- External models, textures, audio, video, fonts, text/data, shaders, and common archives.
- Unity-native assets such as scenes, prefabs, materials, animations, controllers, presets, VFX assets, and ScriptableObject assets.
- Unity asset packages (`.unitypackage`) and UPM packages from registry, Git, local folder, or `.tgz` sources.
- Managed and native plug-in candidates, but only through the executable-content security lane.
- Full Unity project folders as migration sources, never as ordinary single-file imports.
- Standards-backed adapters, beginning with glTF/GLB, USD, PLY, BVH/ASF/AMC, GeoTIFF, DEM, LAS/LAZ, and other formats only after a vetted adapter exists.
- Official-source discovery records for Unity, standards bodies, government, and universities.

### Explicitly not promised

- Recovering original source projects from AssetBundles, players, APK/AAB/IPA files, executables, or proprietary compiled data.
- Decompilation, DRM circumvention, license bypass, or extraction of third-party assets from built games.
- Silent project-version upgrades or downgrades.
- Automatic decisions when scale, rig type, texture semantics, loop intent, render pipeline, or license is ambiguous.
- Runtime remote-content loading. That requires a separate threat model and API.
- A bundled mirror of “all Unity assets” or a trained model containing Unity's documentation.

## 3. Architecture

```text
External file/folder/source URL
            |
            v
  [1. Quarantine Intake] -- hash, immutable copy, limits, provenance
            |
            v
  [2. Probe + Classifier] -- extension + magic + schema + container inventory
            |
            v
  [3. Policy Gate] ------- trust, license, executable content, version, platform
            |
            v
  [4. Import Planner] ---- adapter, dependencies, profile, destinations, conflicts
            |
            v
  [5. Owner Review] ------ only for code/packages/network/overwrite/material ambiguity
            |
            v
  [6. Transaction] ------- snapshot -> Unity/adapter import -> GUID-safe commit
            |
            v
  [7. Validators] -------- semantic + references + compile + consistency + preview
            |
            v
  [8. Ledger] ------------ evidence, citations, license, hashes, outputs, rollback
```

### Package shape

Working package identity: `<owner.reverse-domain>.fidel.importer`

Menu: `Tools > Fidel`

Root namespace: `Fidel.Importer`

```text
Packages/<package-id>/
  Editor/
    Intake/
    Probes/
    Planning/
    Adapters/
    Policies/
    Validation/
    Transactions/
    UI/
  Runtime/                 # empty in v1; avoids accidental runtime scope
  Tests/
    EditMode/
    Fixtures/
  Documentation~/
  Samples~/
  package.json
```

Project-owned data:

```text
ProjectSettings/Fidel/
  project-profile.json
  source-registry.json
  policy.json
  import-ledger.jsonl
  rules/
Library/Fidel/
  cache/                   # disposable derived data only
  reports/
```

Quarantine lives outside `Assets` and `Packages` so Unity cannot automatically import or compile candidate content. The default should be a per-user local application-data directory with a project-specific identifier. No candidate enters Unity-controlled folders until its plan is approved.

## 4. Core modules

| Module | Responsibility | Hard rule |
|---|---|---|
| `FidelIntake` | Copy or reference an input, hash it, capture original name/source, impose size and nesting limits | Never inspect by executing or opening in its authoring application |
| `FidelProbeRegistry` | Run bounded, read-only probes and combine evidence | Extension alone is never sufficient for executable or container types |
| `FidelPolicyEngine` | Evaluate source tier, license, code risk, target version, platform, and project constraints | Unknown license or hidden executable content blocks commit |
| `FidelPlanner` | Select adapter, dependencies, profile, destinations, conflicts, and checks | Produces a dry-run manifest before changing the project |
| `FidelAdapterRegistry` | Delegate to Unity-native importers or a pinned, vetted adapter | No network calls inside an importer |
| `FidelTransaction` | Snapshot affected files and manifests, batch safe changes, commit, and roll back | `.meta`, package manifests, and lock files are part of the transaction |
| `FidelValidatorRegistry` | Validate by asset family and project profile | “No Console error” is necessary but never sufficient |
| `FidelSourceLibrarian` | Store citations, source metadata, license evidence, and rule provenance | Catalog links and metadata; never mirror restricted corpora |
| `FidelLedger` | Record immutable input/output evidence and decisions | Logs are outside the import dependency graph and cannot alter outputs |

Conceptual extension interfaces:

- `IFileProbe`: identifies a format and returns evidence without mutation.
- `IImportAdapter`: declares supported versions/extensions, plans outputs, imports, and reports dependencies.
- `IImportProfile`: supplies project-specific policy and recommended settings.
- `IAssetValidator`: checks actual imported outputs and returns machine-readable findings.
- `IKnowledgeCitation`: links every non-obvious rule to an authoritative source and version.
- `ILicenseEvaluator`: maps item-level rights evidence to allowed project uses; `UNKNOWN` is fail-closed.

## 5. Intake state machine

| State | Meaning | Next action |
|---|---|---|
| `QUARANTINED` | Candidate is copied and hashed, but not trusted | Probe only |
| `IDENTIFIED` | Format and container structure are supported by evidence | Evaluate policy |
| `READY_WITH_APPROVAL` | A deterministic plan exists but a consequential decision remains | Owner reviews exact manifest/risk |
| `NEEDS_ADAPTER` | Format is valid but no approved adapter supports it | Add and validate an adapter in a separate phase |
| `REFERENCE_ONLY` | Useful runtime/build artifact but not recoverable source | Register provenance; do not pretend to convert |
| `REJECTED` | Corrupt, mislabeled, malicious, incompatible, or unlicensed | Preserve report; no retry without new evidence |
| `IMPORTED` | Unity created outputs | Run all applicable validators |
| `VERIFIED` | Import and acceptance contract passed | Record and hand off for owner visual/runtime acceptance if required |
| `ROLLED_BACK` | Transaction was reverted | Report exact restored and non-restorable state |

## 6. Format routing matrix

Unity's current supported-type reference lists built-in model, image, audio/video, native, code/plug-in, shader, text, font, and built-in scripted importers. Fidel should generate its native-format registry from a reviewed, versioned rule file based on that reference rather than hard-code “forever” assumptions.

| Family | Examples | Fidel route | Important validation |
|---|---|---|---|
| 3D models | `.fbx`, `.obj`, `.dae`, `.3ds`, `.dxf`, `.blend`, `.ma`, `.mb`, `.max`, `.c4d`, `.skp`, SpeedTree | Delegate to Unity model importers; prefer interchange formats over DCC-native files | units/scale, axes, hierarchy, meshes, UVs, normals/tangents, materials, rig/avatar, clips, blendshapes, LODs, bounds, NaN/degenerate data |
| Open 3D interchange | `.gltf`, `.glb`, `.usd`, `.usda`, `.usdc`, `.usdz`, `.abc`, `.ply` | Pinned official or standards-body adapter | extension coverage, external URI closure, PBR mapping, animation/skin fidelity, unsupported-feature report |
| Images/textures | `.png`, `.jpg`, `.tif`, `.tga`, `.psd`, `.exr`, `.hdr`, `.dds`, `.ktx`, `.svg`, etc. | Unity texture/IHV importer or official package | dimensions, channels, alpha, color space, normal-map intent, mipmaps, NPOT policy, compression, platform overrides, max size |
| Layered 2D source | `.psb`, supported `.psd` workflows, `.aseprite` where official package applies | Unity released 2D importer package | layer hierarchy, sprite rects, pivots, bones, naming, reimport stability |
| Audio | `.wav`, `.aiff`, `.mp3`, `.ogg`, `.flac`, trackers | Unity audio importer | sample rate, channels, clipping, loudness note, load type, compression, loop markers/intent, platform memory budget |
| Video | `.mp4`, `.webm`, `.mov`, `.avi`, and Unity-listed formats | Unity video importer/transcoder | codec/container support by target, dimensions, frame rate, audio, transcode result, seek/loop behavior |
| Fonts | `.ttf`, `.otf`, `.ttc`, `.dfont` | Unity font importer; optional approved TextMesh Pro asset generation | license embedding rights, glyph coverage, atlas settings, fallback chain |
| Text/data | `.json`, `.xml`, `.yaml`, `.csv`, `.bytes`, `.po`, `.txt`, etc. | `TextAsset` or schema-specific `ScriptedImporter` | encoding, BOM, schema, canonical ordering, locale, numeric culture, external references |
| Shaders/UI/VFX | `.shader`, `.hlsl`, `.compute`, `.raytrace`, `.uss`, `.uxml`, `.vfx*` | Unity-native or released package importer | target graphics APIs, pipeline, includes, variants, compilation on required targets |
| Unity-native serialized assets | `.unity`, `.prefab`, `.asset`, `.mat`, `.anim`, `.controller`, `.preset`, etc. | Preserve source file plus `.meta`; import only into compatible version/project context | GUID graph, missing scripts, type/serialization compatibility, render-pipeline references |
| Managed/native code | `.cs`, `.asmdef`, `.asmref`, `.dll`, `.aar`, `.jar`, `.so`, `.dylib`, `.bundle`, source plug-ins | Executable-content lane; disabled/quarantined until separate approval | signatures/hashes, assembly/native metadata, platform filters, API compatibility, static findings, compile and domain reload |
| UPM packages | registry ID, Git URL, folder, `.tgz` | Package Manager with pinned version/commit and lockfile | package manifest, dependencies, source trust, signatures where available, samples, tests, compilation, package diff |
| Asset packages | `.unitypackage` | Non-executing inventory, explicit content selection, then Unity import; untrusted packages default blocked | path/GUID collisions, scripts/plugins, overwrite set, license receipt, callbacks, compile |
| Generic archives | `.zip`, `.tar`, `.gz`, `.7z` | Extract into quarantine only; recursively classify bounded contents | traversal, symlinks, nested archives, compression ratio, duplicate/case-colliding paths, executable content |
| Full Unity project | directory with `Assets`, `Packages`, `ProjectSettings` | Migration-source workflow, not ordinary import | exact Editor version, packages/lock, GUID graph, project settings, render pipeline, input system, scenes, code, licenses |
| AssetBundles/players | `.unity3d`, bundle files, `.exe`, `.apk`, `.aab`, `.ipa`, built data | `REFERENCE_ONLY` or reject | build target/version metadata when available; never claim source recovery |

### DCC-native model rule

Unity's documentation explains that native authoring files can require the corresponding external application and may be converted through FBX. Fidel therefore recommends committed FBX/other interchange outputs for production and treats direct `.blend`, `.ma`, `.mb`, `.max`, and similar imports as environment-dependent. External application launch is a separate approved action, not an automatic fallback.

## 7. Project profile and import decisions

There is no universally correct texture compression, model scale, rig type, or audio load mode. Fidel therefore requires a version-controlled project profile containing:

- exact Unity Editor version and allowed compatibility range;
- target platforms and graphics APIs;
- Built-in/URP/HDRP pipeline and versions;
- color space and unit convention (`1 Unity unit = 1 meter` unless project overrides);
- 2D/3D project intent;
- texture families and suffix rules such as `_BaseColor`, `_Normal`, `_Mask`, `_Emission`;
- model conventions, rig definitions, animation naming, and LOD thresholds;
- audio categories, streaming/decompress policy, loudness targets, and loop policy;
- package/source allowlists and prohibited licenses;
- warning budget and required validators;
- destination rules and collision policy.

Fidel may recommend a setting from file evidence or naming conventions, but ambiguity remains visible. For example, `_N` may suggest a normal map, but Fidel must show the inference and allow confirmation before changing texture type.

Unity Presets are the first choice for reusable importer settings. `AssetPostprocessor` is used only for deterministic, narrowly scoped rules that Presets cannot express. `ScriptedImporter` is reserved for formats Unity does not natively handle or for an explicitly reviewed override.

## 8. Transaction and GUID safety

Before commit, Fidel records:

- SHA-256 of every input and sidecar;
- exact destination paths;
- current hashes of every file to be overwritten;
- related `.meta` files and GUIDs;
- affected `Packages/manifest.json` and `Packages/packages-lock.json`;
- Fidel, adapter, rule-pack, Unity Editor, package, render-pipeline, and platform versions;
- rollback limits.

Commit rules:

1. Never move or rename an imported Unity asset without its `.meta` file.
2. Never silently replace a valid existing GUID.
3. Detect case-only and Unicode-normalization path collisions before writing.
4. Use Unity `AssetDatabase` operations for project asset moves/copies once content is inside the project.
5. Batch only operations safe for `StartAssetEditing`/`StopAssetEditing`, always with guaranteed cleanup.
6. Separate content import from code/package import because scripts and assemblies can trigger compilation and domain reload.
7. Keep an exact snapshot of touched project files. A package or migration whose effects cannot be fully reversed is labeled accordingly before approval.

## 9. Security model

Default mode is **Strict**.

### Non-executing inspection

- Extension, magic-byte, header, JSON/XML/YAML schema, tar/zip directory, and assembly/PE metadata probes only.
- No shelling out to Blender, Maya, Photoshop, package scripts, codecs, or arbitrary converters during probe.
- No import-time network access. Remote sources are fetched in a separate, explicit acquisition phase, hashed, and then treated as local inputs.
- Archive protections: maximum total expanded bytes, entry count, nesting depth, path length, per-entry size, and compression ratio; reject absolute paths, `..` traversal, alternate data streams, device paths, symlinks/hardlinks unless explicitly supported, and case-colliding destinations.
- Detect double extensions and mismatches between extension and magic bytes.

### Executable-content gate

Scripts, managed assemblies, native libraries, plug-ins, packages containing them, custom importers, editor windows, postprocessors, and project migrations are security-sensitive. Fidel presents:

- exact code/binary inventory and hashes;
- source and signer when available;
- requested platforms and plug-in compatibility filters;
- package dependency diff;
- files to overwrite;
- static findings and limitations;
- containment and rollback plan.

They stay disabled or outside Unity-controlled folders until an explicit owner decision. Unknown scoped registries are blocked because Unity notes that registry packages can contain executable code.

### Importer purity

All Fidel probes and adapters must be deterministic, declare dependencies, version their behavior, avoid time/randomness/network dependence, and avoid main-process side effects. Parallel import workers mean static state and external side effects are unsafe assumptions.

## 10. Validation contracts

Every adapter has a fixture corpus with valid, boundary, malformed, adversarial, and version-skew samples.

### Universal checks

- Input hash and planned input hash still match at commit.
- Actual outputs equal the dry-run destination set or an approved explained delta.
- No unexpected overwrite, missing `.meta`, GUID collision, or broken reference.
- Import log contains no unapproved errors or warnings.
- Reimport produces the same dependency and content results where Unity supports deterministic verification.
- Importer/rule version and all registered dependencies are recorded.
- No network or unplanned external process occurred.
- Rollback rehearsal passes for the fixture class.

### Family-specific checks

- **Models:** hierarchy, object count, mesh statistics, topology hazards, bounds, unit scale, transforms, UV sets, normals/tangents, material slots, bones/weights, avatar state, clips, blendshapes, and LOD structure.
- **Textures:** dimensions, bit depth, channels, alpha, color space, texture type, mip chain, compression, max size, and target overrides.
- **Audio:** duration, sample rate, channel layout, peak/clipping, import load type, compression, memory estimate, and loop contract.
- **Video:** decodability/transcode per target, duration, dimensions, frame rate, audio, seek and loop behavior.
- **Native assets/scenes/prefabs:** dependency closure, missing scripts, broken PPtr/GUID references, serialization compatibility, and required packages.
- **Shaders:** compile for every required graphics API and render pipeline; report stripped or unsupported variants.
- **Packages/code:** dependency resolution, compilation, edit-mode tests, package tests when supplied, assembly reload, and a project diff.
- **Geospatial:** coordinate reference system, datum, vertical units, no-data values, bounds, tiling, origin shift, precision budget, and attribution.

### Evidence classes

Fidel reports evidence without conflating it:

- `STATIC`: file/schema/hash/license inspection.
- `IMPORT`: Unity importer completed and produced expected assets.
- `COMPILE`: scripts/shaders/packages compile for named targets.
- `RUNTIME`: a named test scene or player exercised the result.
- `VISUAL/AUDIBLE`: an owner or approved reviewer accepted rendered or heard output.

Static or import success never substitutes for runtime or owner acceptance when those are part of the asset's contract.

## 11. Fidel's source hierarchy

### Tier 0 — Unity primary sources

Use these to author rules and link explanations. Do not mirror or train on them.

- [Unity 6.3 LTS support policy](https://unity.com/releases/unity-6/support)
- [Unity importing assets](https://docs.unity3d.com/6000.3/Documentation/Manual/import-assets.html)
- [Unity supported asset type reference](https://docs.unity3d.com/6000.3/Documentation/Manual/assets-supported-types.html)
- [Managing importers with scripts](https://docs.unity3d.com/6000.3/Documentation/Manual/ScriptedImporters.html)
- [AssetPostprocessor API](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/AssetPostprocessor.html)
- [Asset Database](https://docs.unity3d.com/6000.3/Documentation/Manual/AssetDatabase.html)
- [Asset metadata and `.meta` behavior](https://docs.unity3d.com/6000.3/Documentation/Manual/AssetMetadata.html)
- [Asset import determinism](https://docs.unity3d.com/6000.3/Documentation/Manual/build-deterministic-assets.html)
- [Analyze the import process](https://docs.unity3d.com/6000.3/Documentation/Manual/import-analyze.html)
- [Parallel import](https://docs.unity3d.com/6000.3/Documentation/Manual/ParallelImport.html)
- [Preset Manager](https://docs.unity3d.com/6000.3/Documentation/Manual/class-PresetManager.html)
- [Unity Package Manager concepts](https://docs.unity3d.com/6000.3/Documentation/Manual/upm-concepts.html)
- [Project package manifest](https://docs.unity3d.com/6000.3/Documentation/Manual/upm-manifestPrj.html)
- [Released Unity packages](https://docs.unity3d.com/6000.3/Documentation/Manual/pack-safe.html)
- [Asset Store package management](https://docs.unity.com/en-us/asset-store/downloads/asset-store-packages)
- [Unity security](https://unity.com/security)
- [Unity Terms of Service](https://unity.com/legal/terms-of-service)
- [Asset Store Terms and EULA](https://unity.com/legal/as-terms)

Unity's current terms restrict automated scraping/knowledge-base replication and AI training on Unity offerings, and the Asset Store terms restrict training and redistribution. Fidel therefore stores compact rule statements, version identifiers, citations, and reviewed test expectations—not wholesale copied content. Any automated access to Unity services or account assets requires a separate terms/authorization review.

Before implementation or distribution, the owner should confirm that Fidel's exact private or distributed use is covered by Unity's documentation and applicable terms, or obtain Unity's written authorization where required. Fidel's rights evaluator is a compliance aid, not legal advice.

### Tier 1 — formal standards and maintained authoritative adapters

- [Khronos glTF registry and specification](https://registry.khronos.org/glTF/)
- [Khronos UnityGLTF](https://github.com/KhronosGroup/UnityGLTF) as a possible vetted adapter, pinned by release/commit and tested against Fidel fixtures
- Unity Registry's `glTFast` package when it is available and compatible with the target Editor; verify its lifecycle state rather than assuming it is released
- Unity's officially supported `com.unity.importer.usd` package, currently documented as pre-release; pin and fixture-test it, and do not use the deprecated `com.unity.formats.usd`
- [OpenEXR reference documentation](https://openexr.com/en/latest/index.html)
- [OGC GeoTIFF standard](https://www.ogc.org/standards/geotiff/)

“Official” does not waive compatibility testing. Adapter version, license, supported features, known limitations, and exact commit remain part of the evidence.

### Tier 2 — government and public-institution sources

Initial approved source connectors:

| Source | Useful content | Rights rule |
|---|---|---|
| [NASA 3D Resources](https://science.nasa.gov/3d-resources/) | mission models, printable models, textures, imagery | Apply [NASA media guidelines](https://www.nasa.gov/nasa-brand-center/images-and-media/) per item; distinguish NASA content, third-party content, people, insignia, and endorsement restrictions |
| [Smithsonian Open Access](https://www.si.edu/openaccess) and [Smithsonian 3D](https://3d.si.edu/collections/openaccesshighlights) | CC0 2D/3D cultural and scientific assets, including GLB/glTF/OBJ | Accept only items explicitly marked CC0; keep item URL and accession metadata; third-party/trademark/privacy rights can still apply |
| [USGS 3D Elevation Program](https://www.usgs.gov/3d-elevation-program) | lidar point clouds, DEMs, topography and metadata | 3DEP states its products are free and without use restrictions; still preserve source, acquisition date, CRS/datum, accuracy, and metadata |
| [NOAA Digital Coast](https://coast.noaa.gov/digitalcoast/data/home.html) | lidar, imagery, elevation, land cover, coastal/ocean data | Rights and provider can vary by dataset; require dataset-level metadata and usage review |
| [Library of Congress Free to Use and Reuse](https://www.loc.gov/free-to-use/) and [Citizen DJ](https://citizen-dj.labs.loc.gov/) | rights-cleared/public-domain images, maps, films, and selected sounds | Use only the explicit free-to-use set or an item whose rights statement permits the intended use; keep collection rights URL and credit line |

These are source catalogs, not assets bundled with Fidel. Downloading is a separate owner-authorized network action, and every selected item receives its own provenance record.

### Tier 3 — university sources

Initial reviewed connectors:

| Source | Useful content | Rights/quality rule |
|---|---|---|
| [Stanford 3D Scanning Repository](https://graphics.stanford.edu/data/3Dscanrep/) | PLY range scans and reconstructed reference meshes | Repository permits research use and free redistribution with credit but restricts commercial product use without permission; default label `RESEARCH_ONLY` |
| [Carnegie Mellon Motion Capture Database](https://mocap.cs.cmu.edu/) | ASF/AMC and related motion-capture data | CMU says data may be used commercially but not resold directly; keep acknowledgement and quality caveats, including noisy toes/hands and non-captured finger/thumb motion |

University affiliation is not a license. Each connector must record the actual rights statement, allowed uses, attribution, dataset limitations, and retrieval date. Unknown or conflicting terms block commercial import.

## 12. Provenance and license record

Every acquired item receives a record equivalent to:

```json
{
  "recordVersion": 1,
  "inputSha256": "...",
  "originalName": "...",
  "source": {
    "tier": "GOVERNMENT",
    "publisher": "Smithsonian Institution",
    "itemUrl": "https://...",
    "retrievedUtc": "...",
    "itemId": "..."
  },
  "rights": {
    "status": "CC0-1.0",
    "evidenceUrl": "https://...",
    "attribution": "...",
    "commercialUse": true,
    "redistribution": true,
    "notes": []
  },
  "import": {
    "unityVersion": "exact project version",
    "adapter": "id@version+commit",
    "rules": "rule-pack hash",
    "profile": "project-profile hash",
    "outputs": []
  }
}
```

Supported rights labels begin with `PD-USGov`, `CC0-1.0`, specific `CC-BY-*`, `COMMERCIAL_WITH_CONDITIONS`, `RESEARCH_ONLY`, `PROPRIETARY_PROJECT_LICENSE`, and `UNKNOWN`. Fidel does not infer permissive rights from “free download,” “educational,” “official,” or “government-hosted.”

## 13. User experience — “Fidel's Desk”

The Editor window has five panes:

1. **Inbox:** select files, folders, or a previously authorized downloaded item.
2. **Identity:** detected format, evidence, hashes, source, license, Unity/version requirements, and hidden contents.
3. **Plan:** adapter, destination tree, settings, dependencies, conflicts, warnings, and validations.
4. **Decision:** one consolidated consequential approval when required; routine native imports with no risk can proceed under an existing project policy.
5. **Report:** final state, evidence classes, output GUIDs, issues, rollback, and the single recommended next action.

The persona can be warm—“Fidel found three material maps but cannot prove which is the normal map”—but status language remains exact and machine-readable.

## 14. Error contract

Every block contains:

- stable error code;
- affected input and hash;
- observed evidence;
- expected contract;
- whether any project mutation occurred;
- exact safe remedy;
- authoritative citation when applicable.

Examples:

- `FIDEL-FMT-001 EXTENSION_SIGNATURE_MISMATCH`
- `FIDEL-ARC-004 PATH_TRAVERSAL`
- `FIDEL-LIC-001 RIGHTS_UNKNOWN`
- `FIDEL-PKG-003 EXECUTABLE_CONTENT_REQUIRES_APPROVAL`
- `FIDEL-GUID-002 DESTINATION_GUID_COLLISION`
- `FIDEL-VER-001 UNITY_VERSION_INCOMPATIBLE`
- `FIDEL-REF-004 MISSING_DEPENDENCY`
- `FIDEL-DET-001 NONDETERMINISTIC_REIMPORT`
- `FIDEL-ADP-001 NO_VETTED_ADAPTER`

## 15. Delivery phases

### Phase A — core, no code execution

- Project profile, source registry, quarantine, hashing, probes, archive safety, dry-run manifest, native non-code routing, provenance ledger, and basic validators.
- Fixtures for textures, FBX/OBJ, audio, text/data, fonts, and Unity-native non-code assets.
- No package installation, scripts, plug-ins, project migration, remote acquisition, or external DCC launch.

### Phase B — official packages and open standards

- UPM inspection/planning, pinned released Unity packages, glTF/GLB, USD/Alembic where supported, PLY, geospatial adapters, and richer validators.
- Package and adapter fixture suites with exact version compatibility.

### Phase C — executable content

- Scripts, managed/native plug-ins, `.unitypackage` executable-content review, compilation lane, package dependency diff, domain reload recovery, and security evidence.
- Requires a fresh security/owner approval for the exact implementation and test environment.

### Phase D — governed project migration

- Full-project donor analysis, GUID dependency graph, selected-content migration, render-pipeline/package reconciliation, and isolated runtime/visual acceptance.

### Phase E — optional official-source acquisition connectors

- Owner-authorized, rate-limited APIs/downloads for approved sources with item-level license capture.
- Unity/Asset Store automated access remains excluded unless Unity's terms and the account's authorization explicitly permit it.

## 16. Design acceptance criteria

Implementation is acceptable only when:

1. The exact supported Unity versions and package versions are pinned and tested.
2. Every input is quarantined before Unity can import or compile it.
3. Format detection uses evidence beyond extension for risky/container types.
4. Every change is preceded by an exact dry-run manifest.
5. Unknown licenses, hidden executable content, and untrusted registries fail closed.
6. Native Unity importers are preferred; custom importers are deterministic, dependency-complete, and versioned.
7. `.meta` files, GUIDs, references, package manifests, and lockfiles are transactionally preserved.
8. Family-specific validators inspect actual outputs.
9. Reimport consistency and rollback tests pass for the supported fixture matrix.
10. Reports distinguish static, import, compile, runtime, and owner visual/audible evidence.
11. No Unity documentation, learning content, or Asset Store corpus is scraped, cloned, trained on, or redistributed.
12. Government/university assets are not bundled automatically; every item has rights evidence and provenance.
13. Unsupported inputs receive an honest `NEEDS_ADAPTER`, `REFERENCE_ONLY`, or `REJECTED` result with a remedy.

## 17. Recommended next decision

The design phase is complete enough to begin a bounded implementation plan, but it does not authorize code or Unity changes.

Recommended first build scope:

> **AUTHORIZE FIDEL PHASE A PLANNING ONLY for an Editor-only Unity 6.3 LTS package: quarantine, hashing, format probes, archive safety, dry-run manifests, non-code native import routing, provenance ledger, and edit-mode fixture design. Exclude Unity launch, package installation, scripts/plugins, external downloads, DCC applications, Asset Store access, project migration, and implementation until the exact plan is reviewed.**

The exact patch version, project path, package namespace, target platforms, and render pipeline must be selected in that plan before implementation.
