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

**Approach (per Q2=c):** pragmatic middle — keep tests meaningful, accept output-format drift when the new format is more correct. Update assertions if rendered CSS changes.

- [ ] Read `/home/ivan/DEVELOPMENT/Fun.Css` source to map current API
- [ ] Document API changes in this file (see *Fun.Css API delta* section below)
- [ ] Patch `Fun.Blazor.Tests/DomTests.fs`
  - [ ] line 133: `style { width 10 }`
  - [ ] line 372: `overflowHidden`
  - [ ] line 373: `height`
- [ ] Patch `Fun.Blazor.Tests/PostRenderFragmentTests.fs`
  - [ ] lines 32, 43, 55: `color "red"` (3 sites)
- [ ] Patch `Fun.Blazor.Tests/ServerTests.fs`
  - [ ] line 94: `color` in style block
  - [ ] line 304: `color` in ruleset block
- [ ] Update any test assertions affected by output-format drift
- [ ] `dotnet test Fun.Blazor.Tests/Fun.Blazor.Tests.fsproj -c Release` — all green
- [ ] Commit C and update this plan

---

## Z: Remove dead `#if !NET6_0` (and `#if NET8_0_OR_GREATER`) directives

**Per Q3=cleanup:** include the `NET8_0_OR_GREATER` site too (dead since matrix is net8+). Keep `NET9_0_OR_GREATER` sites — they remain meaningful (net8 in matrix doesn't satisfy).

### Source files — `#if !NET6_0` (25 sites across 8 files)

- [ ] `Fun.Blazor/Core.fs` (1 site)
- [ ] `Fun.Blazor/DslCore.fs` (3 sites)
- [ ] `Fun.Blazor/DslComponentBuilder.fs` (2 sites)
- [ ] `Fun.Blazor/DslDomAttrBuilder.fs` (1 site)
- [ ] `Fun.Blazor/DslElementBuilder.extentions.fs` (2 sites)
- [ ] `Fun.Blazor/DslSections.fs` (1 site)
- [ ] `Fun.Blazor.Server/DIExtensions.fs` (14 sites)
- [ ] `Fun.Htmx/DslHtmxBuilder.fs` (1 site)

### Source files — `#if NET8_0_OR_GREATER` (1 site, dead)

- [ ] `Fun.Htmx/DslSse.fs` (1 site)

### Keep — `#if NET9_0_OR_GREATER` (still meaningful)

- (no action) `Fun.Blazor/Core.fs` (1 site)
- (no action) `Fun.Blazor/DIComponent.fs` (2 sites)

### fsproj cleanup — 4 conditional ItemGroups across 2 files

- [ ] `Fun.Blazor.Server/Fun.Blazor.Server.fsproj`: drop `=='net6.0'` block, inline `!='net6.0'` block into main
- [ ] `Fun.Htmx/Fun.Htmx.fsproj`: drop `=='net6.0'` block, inline `!='net6.0'` block into main

### Verification

- [ ] `grep -rEn "NET6_0|NET8_0_OR_GREATER" Fun.Blazor*/*.fs Fun.Htmx/*.fs` returns zero hits
- [ ] `dotnet build` clean across `net8.0;net9.0;net10.0`
- [ ] `dotnet test` still green (depends on C done)
- [ ] Commit Z and update this plan

---

## Publish workflow + overlay 4.1.10-IvanTheGeek.4

- [ ] Add `Test` step to `.forgejo/workflows/publish-dev.yml` (between Build and Pack)
- [ ] Add `Test` step to `.forgejo/workflows/publish-stable.yml` (between Build and Pack)
- [ ] Bump `Fun.Blazor/Fun.Blazor.fsproj` `<Version>` to `4.1.10-IvanTheGeek.4`
- [ ] CHANGELOG entry for `.4` (covers C + Z + CI test step)
- [ ] Commit, push to `forgejo`
- [ ] Verify publish-dev CI ran tests successfully
- [ ] Verify `4.1.10-IvanTheGeek.4.<UTC-ts>` appears on Forgejo feed
- [ ] Final plan update + commit

---

## Final cleanup

- [ ] Update `/home/ivan/DEVELOPMENT/Fun.Blazor-contribution-plan.md`:
  - Add `.4` to released versions table
  - Note CI now runs tests
  - Remove resolved items from open questions

---

## Fun.Css API delta

_Populated during C, item 2._

(empty)
