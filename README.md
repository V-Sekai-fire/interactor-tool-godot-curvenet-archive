# TOOL_godot_curvenet_archive

Retired algorithm specs, C++ mirrors, and benchmarks from the
[TOOL_godot_curvenet](https://github.com/V-Sekai-fire/TOOL_godot_curvenet)
project. Each file here was at one point on the production solver path
and was retired after measurement showed it could not meet the project's
performance targets.

This archive exists to preserve the trajectory of the ~100 loops of
solver experimentation that led to the project's current architecture
(Direct Delta Mush at runtime + Hierarchical Sparsify-Compensate at
bind time). The TOMBSTONE headers in each file document what failed
and why, so future contributors do not re-attempt the same paths.

## Contents

### `lean/Curvenet/` — Lean 4 specs (with `native_decide` proofs on
small instances)

| Module | Retired because |
|---|---|
| `ChebyshevAccel.lean` | Wang 2015 Chebyshev acceleration: 15× speedup at 5k but no `ρ` converged at 81k |
| `HeavyEdgeMatching.lean` | Karypis-Kumar 1998 connectivity-aware coarsener: same plateau as principal-axis bucketing on 81k Mire |
| `KernelProjection.lean` | Constant-null-space projection: hypothesised cause of 81k stall, falsified by `diag_70k_cg_baseline` measurement |
| `MultiLevelSchwarz.lean` | Recursive multilevel Schwarz: 7-level hierarchy at 81k, identical residual plateau ~3.7 |
| `TwoLevelSchwarz.lean` | Smith-Bjørstad-Gropp 1996 two-level Schwarz: 193 iters at 5k, stalls at 81k (same root cause) |

### `src/curvenet/` — C++ mirrors of the Lean specs

Each header carries a `TOMBSTONE` block at the top documenting the
specific failure mode and pointing to the diagnostic that retired it.

### `tests/` — RapidCheck property tests + benchmarks/diagnostics

- `test_*.cpp` — RC props for the retired modules (all green when
  archived; algorithms are mathematically correct on small instances,
  they just don't fit the production performance regime)
- `bench_hsc_*.cpp` — HSC variants that didn't beat the production
  ICC baseline (block V-cycle was net-negative; 81k parallel was
  bandwidth-bound and slower than serial ICC)
- `diag_*.cpp` — superseded diagnostics from the multilevel/Schwarz era

## Why preserve at all?

The math in these files is correct (proofs pass, RC props green) but
the algorithms didn't fit the project's combined constraints:

- ≥ 50k vertices
- < 5 ms per 12-RHS frame on M2 Pro / < 0.8 ms on Quest 3
- No third-party libraries (no Eigen, no SuiteSparse)
- All linear algebra in-house, mirrored by Lean specs

Anyone attempting an iterative-runtime solver for cot-Laplacian
character deformation at PCVR-class budgets will likely re-derive these
ideas. The TOMBSTONE headers and accompanying diagnostics save them the
~100 loops it took us to rule each path out.

The current production architecture is documented in the main repo's
`docs/PERF_BASELINE.md` "Current architecture" section, with the
impossibility analysis in `docs/IMPOSSIBILITY.md` (still accurate for
iterative-runtime regimes).

## License

MIT, matching the parent project.
