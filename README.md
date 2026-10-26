# RR-Pit-Puzzle-Structure

AutoCAD 2027 plugin that draws a pit structure — a pit boundary/frame with internal
pieces fitted inside it (Plan, Front, Rear, Side, and Auxiliary Elevation views).

- **Command:** `RR-Pit-Puzzle-Structure` (also on the **"RR Pit Puzzle"** Ribbon tab)
- **Namespace:** `RRPitPuzzle`
- **Target framework:** `net10.0-windows`
- **AutoCAD version:** 2027 (managed API DLLs referenced via `HintPath` from
  `C:\Program Files\Autodesk\AutoCAD 2027\`)
- No NuGet packages.

## Standard Operating Procedure (SOP)

This repo follows one workflow cycle for every change: **implement → build →
bug-focused review → verify → document**. Full details live in
[`project.md`](project.md) (read this first, every session) — this section is the
quick-reference version.

### 1. Repository layout

| Path | Purpose |
|---|---|
| `*.cs` (root) | Plugin source. One feature/function per file where practical — see §4 below. |
| `RR-Pit-Puzzle-Structure.csproj` | SDK-style project file, AutoCAD DLL references. |
| `PluginEntry.cs` | `IExtensionApplication` entry point + command registration. |
| `PitPuzzleRibbon.cs` | Creates the Ribbon tab/button. |
| `PitPuzzleInput.cs` / `PitPuzzleInputForm.cs` | Input model and WinForms wizard dialog. |
| `*Table.cs` (`PuzzleConfigTable`, `ColumnCtcTable`, `MetricHoleTable`) | Hardcoded lookup tables baked from source Excel files (see §3). |
| `Data\`, `PitConfig\`, `Metric Holes\` | Source-of-record `.xlsx` files for the hardcoded tables above (kept for reference, not read at runtime). |
| `SpecSheet\` | One markdown spec file per drawn-geometry feature. |
| `ChangeLog\YYYY-MM-DD.md` | Dated history of every change (one file per day, append-only). |
| `project.md` | Current-state architecture doc — the durable source of truth. |
| `build-testload.ps1` | Builds a timestamped dev DLL into `TestLoad\<stamp>\`. |
| `TestLoad\<stamp>\` | Output of each dev build (gitignored; not committed). |
| `LatestBuild\<stamp>\` | Manually checkpointed snapshot of a known-good build (see §5). |
| `bin\`, `obj\` | Standard build artifacts (gitignored). |

### 2. Build & test cycle

1. Build with:
   ```powershell
   .\build-testload.ps1
   ```
   This produces a uniquely-named DLL under `TestLoad\<stamp>\` (a fresh assembly
   name every build, since .NET assemblies never unload from a running AutoCAD
   session).
2. In AutoCAD, `NETLOAD` the printed `.dll` path and run `RR-Pit-Puzzle-Structure`
   (or use the Ribbon button) to test.
3. Verification is done by the user directly in AutoCAD. Do not write or run
   standalone test harnesses unless explicitly asked.

### 3. Data conventions

Any Excel data file supplied for the project (pit config, column CTC, metric
hole/thread reference, etc.) is converted once into a hardcoded table/dictionary
in a `.cs` file and baked into the assembly. The original `.xlsx` is kept in the
repo as the human-readable source-of-record but is **not** read at runtime. When
the source data changes, the hardcoded table is re-derived by hand and diffed
against the file — this is a resync, not a rewrite.

### 4. File-size / feature-isolation rule

No `.cs` file should grow much past ~600 lines. Every new feature or function
gets its own file:
- `partial class` file named `<ClassName>.<FeatureName>.cs` when it extends an
  existing class, or
- a new standalone static class file when the feature is self-contained.

The new file's method(s) are wired into the existing `Build()`/call site with a
single call — not inlined into an existing file's body.

### 5. "COPY BUILD" checkpoint

Saying **"COPY BUILD"** (exact phrase, case-insensitive) snapshots the most
recent `TestLoad\<stamp>\` build into a matching `LatestBuild\<stamp>\` folder:
every root `*.cs` file, `ChangeLog\`, `SpecSheet\`, `project.md`, and that
TestLoad folder's `.dll`/`.pdb`/`.deps.json`. This is a pure filesystem snapshot
on demand — it is never done automatically after a build, only when explicitly
triggered.

### 6. Documentation requirements

Every change, minor or major, updates both:
- the relevant section(s) of `project.md` (current-state architecture), and
- a `ChangeLog\YYYY-MM-DD.md` entry (what was asked, what changed, why, build
  result, which `project.md` sections were touched).

### 7. Coding conventions

- Keep AutoCAD `Transaction`s short-lived; never call `Editor.Command`/a native
  command while a transaction is open.
- Every `[CommandMethod]` is wrapped in `try/catch` with early-return guard
  clauses for null document / cancelled dialog / failed prompt.
- Layers/symbol tables use an idempotent check-then-create pattern
  (`EnsureLayer`), never assuming existence.
- Dimensions always anchor both points to real, independent geometry — never a
  shared "loose" coordinate for convenience.
- When editing a small detail (a word, value, or condition), use a minimal
  targeted edit — never delete and rewrite a whole line/block for a
  single-token change.

### 8. Deployment (production bundle)

Production deployment is a `.bundle` folder (`RR-Pit-Puzzle-Structure.bundle`,
a sibling of this project folder) with `PackageContents.xml` plus
`Contents\`/`Contents\Windows\` holding a stable-named
`RR-Pit-Puzzle-Structure.dll`/`.pdb`/`.deps.json` from a `dotnet build -c
Release` (no `AssemblyName` override). This is a manual, one-off refresh — not
part of the standing per-cycle workflow above.

## Reference project

Conventions in this project (dialog layout, settings/input persistence,
`EnsureLayer`/`EnsureDimStyle`, spec-sheet template) are adapted from
`Auto Stage MS Tower - Version 2.0`.
