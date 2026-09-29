# Operator quickstart

**Read this before treating anything in this repository as a working system.**
`AGENTS.md` describes six AI agents coordinating over five Matrix rooms to run
procurement, stock, fulfilment and delivery for an OTC pharmaceutical storefront
under 薬機法 compliance. **None of that is in this repository.** What is here is a
ClojureScript (shadow-cljs + reagent + re-frame + jp-go-dds) scaffold whose own
component says so:

```
appview/etzhayyim-wasm-pharma-f0963b54/cljs/src/pharma/app.kotoba
  :page/heading "etzhayyim-wasm-pharma-f0963b54"
  :page/description "Vite entry scaffold after SvelteKit cleanup."
```

(That description text is a fossil of the frontend's previous life as a
Svelte/Vite scaffold — see "What changed" below. It was kept verbatim rather
than invented, since inventing a new description would misrepresent what this
scaffold actually is.)

This document is repository orientation only. It does not describe how to dispense
or sell medicines, and nothing here should be read as compliance guidance — the
compliance layer the architecture names is one of the parts that is absent.

Steps marked ✅ were run against this tree. Recorded output is real.

---

## 0. What changed (2026-08-26)

The frontend was migrated from Svelte 5 + Vite to ClojureScript (shadow-cljs)
+ reagent + re-frame, rendered with `jp-go-dds` (デジタル庁デザインシステム),
per this workspace's repo-wide Svelte/React → cljs migration (ADR-2608260900).
`appview/etzhayyim-wasm-pharma-f0963b54/svelte/` (9 files: a single
`App.svelte` with a heading + one paragraph, no interactivity, no routing) was
removed and replaced by
`appview/etzhayyim-wasm-pharma-f0963b54/cljs/`. Nothing else in this
repository was touched — there is no backend TypeScript here to migrate
separately (the `component.wasm` this repo's `kotodama.jsonld` declares was
never tracked either, see §2).

## 1. Measure the gap yourself ✅

Everything `AGENTS.md` describes is searchable, so do not take the paragraph above
on trust:

```bash
for kw in Matrix pharma-rx01 lexicon xrpc 薬機; do
  printf '%s outside AGENTS.md: ' "$kw"
  git grep -l "$kw" -- . | grep -v AGENTS.md | wc -l
done
```

Actual output — every one is zero:

```
Matrix outside AGENTS.md:        0
pharma-rx01 outside AGENTS.md:        0
lexicon outside AGENTS.md:        0
xrpc outside AGENTS.md:        0
薬機 outside AGENTS.md:        0
```

The six agent nanoids (`pharma-rx01` … `pharma-cp01`), the Matrix rooms, the
XRPC/lexicon surface and the 薬機法 handling appear in the documentation and
nowhere else.

## 2. Check the actor manifest against the tree ✅

Both JSON-LD files parse, and the manifest makes one claim that can be checked:

```bash
python3 -c "
import json,io
for f in ['PROJECT.jsonld',
          'appview/etzhayyim-wasm-pharma-f0963b54/kotodama.jsonld']:
    d = json.load(io.open(f)); print(f, '-> parses,', len(d), 'keys')
    if 'component' in d: print('   declares component:', d['component'])
"
ls appview/etzhayyim-wasm-pharma-f0963b54/component.wasm
```

Actual output:

```
PROJECT.jsonld -> parses, 6 keys
appview/etzhayyim-wasm-pharma-f0963b54/kotodama.jsonld -> parses, 17 keys
   declares component: {'path': 'component.wasm'}
ls: appview/.../component.wasm: No such file or directory
```

**The actor manifest points at a WebAssembly component that does not exist**, and
no `.wasm` file is tracked anywhere in the repository. Anything that loads this
manifest expecting a component will fail at that path.

Three more values in that manifest are generic scaffold defaults rather than
anything domain-specific. They are listed because they read like policy and are
not:

| field | value | what it is |
|---|---|---|
| `duties` | `["勤労の義務"]` | scaffold default, not a pharmacy duty |
| `kpi` | `["credits の獲得"]` | scaffold default, not a business metric |
| `governance.complianceFrameworks` | `["pharma-policy"]` | names a framework; no policy file exists in this repo |

## 3. Build the appview ✅

```bash
cd appview/etzhayyim-wasm-pharma-f0963b54/cljs
npm install
```

Requires network access once (to resolve `deps.edn`/npm dependencies,
including the `jp-go-dds` git dependency and the `react`/`react-dom` npm
packages reagent needs); after that it is fully local. Go through the
repo-wide resource governor rather than invoking `shadow-cljs` directly —
high-load builds are limited to one at a time across the workspace:

```bash
node <root>/scripts/resource-guard.mjs run build -- amu compile --target wasm32-browser app
```

Actual output:

```
[:app] Compiling ...
[:app] Build completed. (111 files, 110 compiled, 0 warnings, 18.13s)
```

Produces a compiled bundle in `public/js/`, served alongside the committed
`public/index.html`.

## 4. Test the appview ✅

```bash
node <root>/scripts/resource-guard.mjs run build -- amu compile --target wasm32-browser test
node out/tests.js
```

Actual output:

```
Testing pharma.app-test

Ran 4 tests containing 6 assertions.
0 failures, 0 errors.
```

(The "re-frame: Subscribe was called outside of a reactive context" lines
that also print are a benign re-frame FAQ warning from subscribing outside a
reagent render in a `cljs.test` context — not a failure.)

## 5. Serve the built bundle

`public/index.html` + `public/js/` is a static bundle; serve `public/` with any
static file server (there is no dev-server preview script here — shadow-cljs's
own `watch` build is for iterative development, not for smoke-testing a
release build) and confirm `/` returns the page with `<div id="app">`
populated by the mounted reagent view — the heading
`etzhayyim-wasm-pharma-f0963b54` — **not** an OTC pharmacy storefront UI
(see §6).

## 6. What this repository is, in one line

A migration seed. `migration.edn` records it as 14 tracked files extracted
verbatim from `etzhayyim/root` at `60-apps/etzhayyim-project-pharma`, with the
source revision and tree hash pinned, plus the additions declared in
`:identity/:allowed-additions`. The architecture document travelled with the
extraction and describes the system as designed, not as delivered. The
frontend scaffold that shipped with it has since been ported from Svelte to
ClojureScript (§0) — the port is faithful (same two lines of text, same
layout), not a redesign.

**So the useful next step is not to run this — it is to decide whether the
described system exists elsewhere and should be pointed at, or does not exist and
the document should say so.** That is a question about the project, not about this
tree, and this quickstart does not answer it.
