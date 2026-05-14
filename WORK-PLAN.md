# Fun.Blazor Overlay — Work Plan

**Updated:** 2026-05-14
**Branch:** `ivan-integrated`
**Target release:** `4.1.10-IvanTheGeek.4`

This plan covers two parallel cleanups (C and Z) plus a workflow improvement, all bundled into the `.4` release.

## Status legend

- `[ ]` Pending
- `[~]` In progress
- `[x]` Done

---

## Pre-requisites (committed with this plan)

- [x] Bump sibling TFMs to `net8.0;net9.0;net10.0` so the test suite restores
  - [x] `Fun.Blazor.Server/Fun.Blazor.Server.fsproj`
  - [x] `Fun.Blazor.HtmlTemplate/Fun.Blazor.HtmlTemplate.fsproj`
  - [x] `Fun.Htmx/Fun.Htmx.fsproj`

---

## C: Patch test suite to IvanTheGeek Fun.Css API

**Outcome:** simpler than anticipated. No API rewrite needed — the fork's CSS CustomOperations were moved into an `[<AutoOpen>] module CssBuilderGenerated` inside `Fun.Css/CssBuilder.generated.fs`. F# only sees `[<AutoOpen>]` extension modules when the parent namespace is open. The three test files only had `open Fun.Blazor`, not `open Fun.Css`. Adding `open Fun.Css` to each made all errors disappear without any source-line changes.

- [x] Read `/home/ivan/DEVELOPMENT/Fun.Css` source — found commit [`ae1dfab`](https://forgejo.ivanthegeek.com/IvanTheGeek/Fun.Css/commit/ae1dfab) documenting the exact pattern
- [x] Document API delta (see *Fun.Css API delta* section below)
- [x] Patch `Fun.Blazor.Tests/DomTests.fs` — added `open Fun.Css`
- [x] Patch `Fun.Blazor.Tests/PostRenderFragmentTests.fs` — added `open Fun.Css`
- [x] Patch `Fun.Blazor.Tests/ServerTests.fs` — added `open Fun.Css`
- [x] No assertion changes needed (no output-format drift)
- [x] `dotnet test` — **52 passed, 0 failed, ~1s**
- [x] Commit C (commit `9774633`)

---

## Z: Remove dead `#if !NET6_0` (and `#if NET8_0_OR_GREATER`) directives

**Per Q3=cleanup:** included the `NET8_0_OR_GREATER` site too (dead since matrix is net8+). Kept `NET9_0_OR_GREATER` sites — they remain meaningful (net8 in matrix doesn't satisfy).

### Source files — `#if !NET6_0` (25 sites across 8 files) ✓ all removed

- [x] `Fun.Blazor/Core.fs` (1 site)
- [x] `Fun.Blazor/DslCore.fs` (3 sites)
- [x] `Fun.Blazor/DslComponentBuilder.fs` (2 sites)
- [x] `Fun.Blazor/DslDomAttrBuilder.fs` (1 site)
- [x] `Fun.Blazor/DslElementBuilder.extentions.fs` (2 sites)
- [x] `Fun.Blazor/DslSections.fs` (1 site)
- [x] `Fun.Blazor.Server/DIExtensions.fs` (14 sites, 4 with `#else` blocks)
- [x] `Fun.Htmx/DslHtmxBuilder.fs` (1 site)

### Source files — `#if NET8_0_OR_GREATER` (1 site, dead) ✓ removed

- [x] `Fun.Htmx/DslSse.fs` (1 site)

### Keep — `#if NET9_0_OR_GREATER` (still meaningful)

- (no action) `Fun.Blazor/Core.fs` (1 site)
- (no action) `Fun.Blazor/DIComponent.fs` (2 sites)

### fsproj cleanup ✓ done

- [x] `Fun.Blazor.Server/Fun.Blazor.Server.fsproj`: dropped `=='net6.0'` block, inlined `!='net6.0'` block (became unconditional)
- [x] `Fun.Htmx/Fun.Htmx.fsproj`: dropped `=='net6.0'` block, inlined `!='net6.0'` block

### Verification ✓ done

- [x] `grep -rEn "NET6_0|NET8_0_OR_GREATER" Fun.Blazor*/*.fs Fun.Htmx/*.fs` → zero hits
- [x] `grep -E "net6\.0" {Server,Htmx,HtmlTemplate}.fsproj` → zero hits
- [x] `dotnet build` clean across `net8.0;net9.0;net10.0`
- [x] `dotnet test` → **52 passed, 0 failed**
- [x] Commit Z (commit `87432f7` + cleanup `f946b39` for the accidental `.claude/worktrees` submodule reference)

---

## Publish workflow + overlay 4.1.10-IvanTheGeek.4

- [x] Add `Test` step to `.forgejo/workflows/publish-dev.yml` (between Build and Pack)
- [x] Add `Test` step to `.forgejo/workflows/publish-stable.yml` (between Build and Pack)
- [x] Bump `Fun.Blazor/Fun.Blazor.fsproj` `<Version>` to `4.1.10-IvanTheGeek.4`
- [x] CHANGELOG entry for `.4` (covers C + Z + CI test step)
- [x] Commit, push to `forgejo` (commit `5a2aa69`)
- [x] Verify publish-dev CI ran tests successfully (Forgejo Actions API → `success`; package on feed)
- [x] Verify `4.1.10-IvanTheGeek.4.20260514204318` appears on Forgejo feed
- [x] Final plan update

---

## Final cleanup

- [x] Update `/home/ivan/DEVELOPMENT/Fun.Blazor-contribution-plan.md`:
  - Added `.4` row to released versions table; `.3` marked superseded
  - Added "Test gate" subsection
  - Added "Fun.Css `open` requirement" caveat
  - Refreshed sibling-TFM open question (mostly resolved by `.4`)

---

## Fun.Css API delta

The IvanTheGeek Fun.Css fork has restructured how CSS CustomOperations are exposed: many of them moved from being direct members of `Fun.Css.CssBuilder` to being type extensions in `Fun.Css/CssBuilder.generated.fs`, inside an `[<AutoOpen>] module CssBuilderGenerated`.

**Critical consequence for consumers:**

F#'s computation-expression CustomOperation lookup only discovers `[<AutoOpen>]` extension modules when the containing namespace (`Fun.Css`) is in scope. So a consumer file that has `open Fun.Blazor` but **not** `open Fun.Css` will compile fine for direct members of CssBuilder but fail with `FS3095` ("X is not used correctly. This is a custom operation...") or `FS0039` ("X is not defined") for CSS ops that moved to the extension module.

**Symptoms hit during Fun.Blazor.Tests compile:**

| Site | Error | Op moved to extension |
|---|---|---|
| `DomTests.fs:133` `style { width 10 }` | FS3095 | yes |
| `DomTests.fs:372` `overflowHidden` | FS0039 (not defined) | yes |
| `DomTests.fs:373` `height "100%"` | FS3095 | yes |
| `PostRenderFragmentTests.fs:32,43,55` `color "green"` | FS3095 | yes |
| `ServerTests.fs:94` `color "red"` in style | FS3095 | yes |
| `ServerTests.fs:304` `color "red"` in ruleset | FS0039 | yes |

**Fix:** one-line `open Fun.Css` added to each of the three files. No call-site changes; no assertion changes. All 52 tests pass.

**Reference:** Fun.Css commit [`ae1dfab`](https://forgejo.ivanthegeek.com/IvanTheGeek/Fun.Css/commit/ae1dfab) "Fix CI: add `open Fun.Css` to Benchmark so generated CE ops are visible" hit the same trap in the fork's own benchmark project.

**Note for future work:** the contribution plan's caveat list (or LauraExperiment findings doc) should note this — any new F# file using the Fun.Css CSS DSL via Fun.Blazor needs both `open Fun.Blazor` AND `open Fun.Css` to see the full CE op surface.
