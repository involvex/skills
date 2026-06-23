---
name: bun-win32
description: >
  Win32 FFI binding lifecycle for bun-win32: generate, audit, and release
  @bun-win32/* packages (Win32 DLL bindings via bun:ffi on Windows). Covers:
  prerequisite checking, scaffold generation, catalog/dumpbin symbol discovery,
  FFI return-shape probing, paste-ready stub emission, FFI↔TS↔header consistency
  auditing, SAL-driven nullability checking, and pre-publish lockfile gating.
  Repo: D:\repos\bun-win32 (117 packages, strict TypeScript, Bun runtime, Biome formatting).
engines:
  - opencode
---

## Who This Skill Is For

Use this skill when an agent needs to:
- **Generate a new `@bun-win32/{name}` package** from a Windows system DLL
- **Audit an existing binding** for FFI-type / TS-type / SDK-header consistency
- **Fix nullability** (`| NULL` / `| 0n` missing or spurious) in a package
- **Understand the full development lifecycle** from scaffold to publish

## Repository Context

```
D:\repos\bun-win32\               ← repo root (WORKING_DIR)
  packages/
    core/     @bun-win32/core    shared base class + types, no DLL binding
    all/      @bun-win32/all     re-exports every PascalCase class
    template/ @bun-win32/template scaffold with WIN32_CLASS placeholders
    {name}/   @bun-win32/{name}  one package per system DLL
  scripts/
    doctor.ts    bootstrap.ts    scaffold.ts    catalog.ts
    stub.ts      audit.ts        nullcheck.ts   preflight.ts    ffi-runtime.ts
  PROMPT.md                          ← deep playbook (read on demand)
  AGENTS.md / CLAUDE.md             ← operating rules
```

Every script runs from the repo root via `bun run scripts/{name}.ts {args}`.
Scripts use `ROOT = join(import.meta.dir, '..')` to reference the repo root — this works because the skill executes them in the repo context.

## Lifecycle Commands (in order)

### 1. `bun run scripts/doctor.ts`
Check prerequisites (platform Windows, Bun ≥1.3.0, ripgrep, Windows SDK, dumpbin).

### 2. `bun run scripts/bootstrap.ts {name}`
Full orchestrated pipeline: doctor → scaffold → install → catalog → ffi-runtime → stub.
Specify `--class=ClassName` to override the auto-generated PascalCase name.
Specify `--rg=<path>` or `--dll=<path>` to override DLL/SDK search paths.

### 3. `bun run scripts/scaffold.ts {name}` (or use bootstrap)
Copy `packages/template` → `packages/{name}`, substituting WIN32_CLASS placeholders.

### 4. `bun run scripts/catalog.ts {name} --json` (or use bootstrap)
Dump DLL exports via `dumpbin //EXPORTS` and intersect with SDK header C prototypes.
Output is JSON: `{ dllPath, packageName, capturedAt, exports[], sdkIncludeRootPath }`.

### 5. `bun run scripts/ffi-runtime.ts {name}` (or use bootstrap)
Probe Bun FFI return-value shapes for each export (u64, i64, ptr, null, void, number).
Produces a table used to validate the FFI mapping.

### 6. `bun run scripts/stub.ts {name}` (or use bootstrap)
Consume `catalog.ts --json` output, map C types → FFI/TS types, emit:
- `types/{Class}.ts` type aliases
- `structs/{Class}.ts` Symbols block (alphabetized FFI declarations)
- `structs/{Class}.ts` method stubs (alphabetized, with MS Learn URL comments)
- Warnings for forwarded exports, missing prototypes, unknown C types, inferred pointers

Use `--class=ClassName` to override; `--log` to append to `.generation-log.md`.

### 7. `bun run scripts/audit.ts {name}` (or `--all`) (`--fix`)
Cross-check FFIType ↔ TS type ↔ SDK header C type consistency per symbol.
Flags: TYPE_MISMATCH, MISSING_TYPE, SPURIOUS_TYPE, RETTYPE_MISMATCH, UNKNOWN_SYM.
`--fix` applies all non-structural corrections automatically.

### 8. `bun run scripts/nullcheck.ts {name}` (or `--all`) (`--fix --strict`)
SAL-driven nullability auditor. Flags pointer/handle parameters missing `| NULL`/`| 0n`
when the SDK header marks them `_opt_` / `_Reserved_` / `OPTIONAL`.
Kinds: MISSING (fix this), SPURIOUS (info only), TYPE_MISMATCH (handle typed as pointer or vice versa),
UNMATCHED (param names don't align — review), NO_SDK (COM/DirectX/undocumented).
`--fix` adds missing nullable unions; `--strict` also exits non-zero on TYPE_MISMATCH.

### 9. `bun run scripts/preflight.ts`
Pre-publish lockfile staleness gate. Fails if `bun.lock` was not regenerated via
`rm bun.lock && bun install` before release.

### 10. `bunx tsc --noEmit`
Type-check the package. Run after every meaningful change.

## Verify Commands (run constantly)

```bash
bun run packages/{name}/index.ts          # smoke-test package loads
bun run packages/{name}/example/foo.ts    # run a demo
bunx tsc --noEmit                          # type-check
bunx biome format --write packages/{name}  # format
```

## Release Commands

```bash
rm bun.lock && bun install                    # refresh workspace version records
bun run scripts/preflight.ts                  # lockfile gate
bun run scripts/nullcheck.ts --all && bun run scripts/audit.ts --all  # zero-problem gate
# publish each package on ONE OTP:
cd packages/{name} && bun publish --access public --otp <code>
```

## Scripts Reference

| Script | Input | Output | Key Flags |
|---|---|---|---|
| `doctor.ts` | — | console diagnostics | — |
| `bootstrap.ts` | package name | full generated package | `--class`, `--rg`, `--dll` |
| `scaffold.ts` | package name | package skeleton | `--class` |
| `catalog.ts` | package name | JSON DLL∩SDK data | `--json`, `--rg`, `--dll` |
| `ffi-runtime.ts` | package name | FFI return-shape table | — |
| `stub.ts` | `catalog --json` output | paste-ready Symbols+stubs | `--class`, `--log` |
| `audit.ts` | `packages/{name}/structs/` | findings table | `--all`, `--fix` |
| `nullcheck.ts` | `packages/{name}/structs/` + SDK | nullability findings | `--all`, `--fix`, `--strict` |
| `preflight.ts` | `bun.lock` | pass/fail | — |

## Reference Guide

Bundled reference files (in `references/`) provide persistent context without re-reading the repo:

| File | Contents |
|---|---|
| `references/agents.md` | Full `AGENTS.md` — binding rules, toolchain, type conventions, FFI rules, release process, prohibited actions |
| `references/ai-core.md` | `@bun-win32/core` contract — Win32 base class, `.ptr` extension, shared types |
| `references/ai-all.md` | `@bun-win32/all` contract — re-export aggregator, when to use vs. specific package |

`PROMPT.md` (at `D:\repos\bun-win32\PROMPT.md`) is the authoritative deep playbook
for FFI type mapping, nullability, examples, and audits. Fetch it when you need
detail beyond what `AGENTS.md` provides.

## FFI Type Quick Reference

| Win32 type | FFI | TS |
|---|---|---|
| `HANDLE`, `HWND`, `HKEY`, `HMODULE`, … | `FFIType.u64` | `bigint` |
| `SIZE_T`, `*_PTR`, `LPARAM`, `LRESULT`, `WPARAM` | `FFIType.u64` | `bigint` |
| `LARGE_INTEGER`, `ULARGE_INTEGER` | `FFIType.i64` / `u64` | `bigint` |
| `DWORD`, `UINT`, `BOOL`, `HRESULT`, `INT`, `LONG`, `WORD`, `BYTE` | `FFIType.u32` / `i32` | `number` |
| `LPVOID`, `LPCWSTR`, `LPSTR`, `LPDWORD`, `LPBYTE`, … | `FFIType.ptr` | `Pointer` |
| `void` | `FFIType.void` | `void` |

**Decision rule:** Does the caller pass `.ptr` from a `Buffer`/`TypedArray` they allocated? Yes → `ptr`. No → `u64`.

**NULL representation:** `u64 → 0n`, `ptr → null`, `u32 → 0`.

## Prohibited Actions

The skill **must never**:
- Bind an export not confirmed by `dumpbin //EXPORTS`
- Guess a type, nullability, or export signature — always verify against SDK header + MS Learn
- Use `as any` / `as unknown as T` / forced casts — fix the FFI mapping instead
- Change public API (export shape, signatures, type contracts) without explicit request
- Reformat files not explicitly being edited
- Mutate shipped bindings to silence audit hints without verifying against MSDN first
- Add helpers, wrappers, or utilities not explicitly requested
- Ship without running `audit.ts --all` and `nullcheck.ts --all` (zero findings required)