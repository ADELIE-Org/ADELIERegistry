# ADELIERegistry

A [local Julia registry](https://github.com/GunnarFarneback/LocalRegistry.jl) for
the [ADELIE-org](https://github.com/ADELIE-org) packages. It lets you install all
the ADELIE packages by name — `Pkg.add("ISOAP")` — with inter-package
dependencies resolved automatically, instead of juggling `dev` paths or
`[sources] {path=...}` blocks (which break in CI).

## Registered packages

| Package | Depends on (within ADELIE) |
|---|---|
| `AlgebraCore` | — |
| `ScalarAlgebra` | `AlgebraCore` |
| `FieldAlgebra` | `AlgebraCore`, `ScalarAlgebra` |
| `VOFTools` | — |
| `Vofinit` | — |
| `VofiJul` | — |
| `FrontIntrinsicOps` | — |
| `FrontCartesianGeometry` | `FrontIntrinsicOps` |
| `FrontSpaceTimeDEC` | `FrontIntrinsicOps` |
| `CartesianGrids` | — |
| `ISOAP` | `CartesianGrids` |
| `CartesianGeometry` | `CartesianGrids`, `VofiJul`, `Vofinit` |

## Install on a new machine

```julia
using Pkg
Pkg.Registry.add("General")  # if this depot has no General registry yet
Pkg.Registry.add(Pkg.RegistrySpec(url="https://github.com/ADELIE-Org/ADELIERegistry.git"))

# then add whichever packages you need — deps resolve through the registry:
Pkg.add(["CartesianGrids", "ISOAP", "VofiJul", "Vofinit", "VOFTools", "FrontIntrinsicOps", "FrontCartesianGeometry", "FrontSpaceTimeDEC", "CartesianGeometry"])
```

Or from the Pkg REPL (`]`):

```
registry add https://github.com/ADELIE-Org/ADELIERegistry.git
add CartesianGrids ISOAP VofiJul Vofinit VOFTools FrontIntrinsicOps FrontCartesianGeometry FrontSpaceTimeDEC CartesianGeometry
```

The registry only stores metadata; package source is fetched from each package's
own GitHub repo under `ADELIE-Org`.

`AlgebraCore` / `ScalarAlgebra` / `FieldAlgebra` are ADELIE-Org **mirror forks** of
the upstream symbolic-algebra trio (`vlc1/*`), kept at identical names + UUIDs so
downstream packages depend on them unchanged. Note: General already registers an
unrelated `FieldAlgebra` (uuid `0b62d163-…`); ours is uuid
`37466282-f9df-4509-9b24-142db341d2b3`, so depend on it by UUID — a bare
`Pkg.add("FieldAlgebra")` is ambiguous.

## Using it in CI / docs

The ADELIE package workflows add this registry before building. The one line that
matters:

```yaml
- name: Add ADELIE local registry
  run: |
    julia -e 'using Pkg; Pkg.Registry.add("General"); Pkg.Registry.add(Pkg.RegistrySpec(url="https://github.com/ADELIE-Org/ADELIERegistry.git"))'
```

(Put it before `julia-actions/julia-buildpkg`, or before `Pkg.develop`/`Pkg.instantiate`
in a Docs workflow.) The `new-adelie-package` scaffold already includes it.

## Publishing a package / new version

Each registered package must be **committed and pushed** to its GitHub remote
first — the registry records the git-tree-hash of `HEAD` and expects it reachable
on `origin`.

From the ADELIE workspace root, `register_all.jl` registers every package in
dependency order:

```bash
julia register_all.jl
```

To publish a **new version** of one package: bump `version` in its `Project.toml`,
commit + push, then:

```julia
using Pkg; Pkg.add("LocalRegistry")
using LocalRegistry
register("CartesianGrids.jl"; registry="ADELIERegistry", push=true)
```

Always register a package *after* the ADELIE packages it depends on.

## Updating to the latest registered versions

```julia
using Pkg
Pkg.Registry.update("ADELIERegistry")
Pkg.update()
```
