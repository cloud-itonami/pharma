# Operator quickstart

**Read this before treating anything in this repository as a working system.**
`CLAUDE.md` describes six AI agents coordinating over five Matrix rooms to run
procurement, stock, fulfilment and delivery for an OTC pharmaceutical storefront
under 薬機法 compliance. **None of that is in this repository.** What is here is a
Vite/Svelte scaffold whose own component says so:

```
appview/etzhayyim-wasm-pharma-f0963b54/svelte/src/App.svelte
  <h1>etzhayyim-wasm-pharma-f0963b54</h1>
  <p>Vite entry scaffold after SvelteKit cleanup.</p>
```

This document is repository orientation only. It does not describe how to dispense
or sell medicines, and nothing here should be read as compliance guidance — the
compliance layer the architecture names is one of the parts that is absent.

Steps marked ✅ were run against this tree. The one marked ⚠ NOT WALKED says why.

---

## 1. Measure the gap yourself ✅

Everything `CLAUDE.md` describes is searchable, so do not take the paragraph above
on trust:

```bash
for kw in Matrix pharma-rx01 lexicon xrpc 薬機; do
  printf '%s outside CLAUDE.md: ' "$kw"
  git grep -l "$kw" -- . | grep -v CLAUDE.md | wc -l
done
```

Actual output — every one is zero:

```
Matrix outside CLAUDE.md:        0
pharma-rx01 outside CLAUDE.md:        0
lexicon outside CLAUDE.md:        0
xrpc outside CLAUDE.md:        0
薬機 outside CLAUDE.md:        0
```

The six agent nanoids (`pharma-rx01` … `pharma-cp01`), the Matrix rooms, the
XRPC/lexicon surface and the 薬機法 handling appear in the documentation and
nowhere else. 16 tracked files, of which 9 are the Svelte scaffold.

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

## 3. Build the appview ⚠ NOT WALKED

`svelte/package.json` declares `vite build` and depends on
`@bufbuild/protobuf`, `@connectrpc/connect`, `@connectrpc/connect-web` and
`@etzhayyim/design-system`. **There is no lockfile and no `node_modules`**, so a
build requires a network install. It was not run while writing this and is
therefore not claimed to work. If you run it, go through the repo-wide resource
governor rather than invoking the build directly — high-load builds are limited to
one at a time across the workspace:

```bash
node <root>/scripts/resource-guard.mjs run build -- \
  npm --prefix appview/etzhayyim-wasm-pharma-f0963b54/svelte run build
```

Expect a build of the placeholder page above, not of an application.

---

## 4. What this repository is, in one line

A migration seed. `migration.edn` records it as 14 tracked files extracted
verbatim from `etzhayyim/root` at `60-apps/etzhayyim-project-pharma`, with the
source revision and tree hash pinned, plus the additions declared in
`:identity/:allowed-additions`. The architecture document travelled with the
extraction and describes the system as designed, not as delivered.

**So the useful next step is not to run this — it is to decide whether the
described system exists elsewhere and should be pointed at, or does not exist and
the document should say so.** That is a question about the project, not about this
tree, and this quickstart does not answer it.
