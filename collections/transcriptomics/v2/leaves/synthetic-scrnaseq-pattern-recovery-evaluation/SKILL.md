---
name: synthetic-scrnaseq-pattern-recovery-evaluation
description: Use when you need to validate that a cell-state decomposition recovers
  a known number of ground-truth identity/activity patterns (e.g., 13 activity patterns)
  on synthetic single-cell RNA-seq data with controlled noise.
when_to_use_negative:
- Your data has no ground-truth pattern labels — coverage against the 13 true activity
  patterns cannot be computed; use internal criteria (e.g., plateau of nontrivial
  columns) instead
- You need a single fixed k solution rather than a k = 2..30 sweep; the nested ACTION
  formulation exists precisely for multi-resolution sweeps, not one-shot decomposition
- Input is scATAC-seq chromatin accessibility — use the ATACtion reduction path (chromVAR/LSI)
  rather than the gene-expression kernel reduction
tools:
- name: ACTIONet
  role: R/Python environment providing the randomized SVD reduction step (an.pp.reduce_kernel)
    and the ACTION decomposition used in the benchmark
  repo: https://github.com/shmohammadi86/ACTIONet
- name: ACTION
  role: Reference numerical core providing runSPA (successive projection) and the
    nested runACTION archetype sweep k_min..k_max whose nontrivial columns are counted
  repo: https://github.com/shmohammadi86/ACTION
- name: R
  role: Language for the recommended stable ACTIONet R-release package used to run
    the sweep and coverage analysis
  repo: https://github.com/shmohammadi86/ACTIONet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:80990394b8d41520f349057c8179fbbcdf7ef0b0f6ae509e6e8b26e6f51dd5ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTION
  - ACTIONet
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# synthetic-scrnaseq-pattern-recovery-evaluation

## Summary

Benchmarks whether an archetypal decomposition (SPA alone vs SPA combined with ACTION's randomized-SVD reduction and nested archetype sweep) recovers all known activity patterns in a synthetic scRNA-seq dataset with 13 ground-truth patterns, by computing pattern coverage as a function of archetype number k = 2..30 across noise levels.

## When to use

Use when you need to validate that a cell-state decomposition recovers a known number of ground-truth identity/activity patterns (e.g., 13 activity patterns) on synthetic single-cell RNA-seq data with controlled noise.

## When NOT to use

- Your data has no ground-truth pattern labels — coverage against the 13 true activity patterns cannot be computed; use internal criteria (e.g., plateau of nontrivial columns) instead
- You need a single fixed k solution rather than a k = 2..30 sweep; the nested ACTION formulation exists precisely for multi-resolution sweeps, not one-shot decomposition
- Input is scATAC-seq chromatin accessibility — use the ATACtion reduction path (chromVAR/LSI) rather than the gene-expression kernel reduction

## Inputs

- Synthetic scRNA-seq benchmark dataset from ref. 12 (20 instances of cells with 13 activity patterns), as a gene x cell expression matrix (Matrix Market `expression.mm` with `gene_names.txt` and `sample_annotations.txt` in the ACTION demo layout)
- Noise-level variants of the same synthetic dataset
- Ground-truth labels mapping cells to the 13 true activity patterns
- Archetype-number sweep range (k = 2..30)

## Outputs

- Per-k counts of nontrivial archetype columns for SPA+ACTION and SPA-only at each noise level
- Pattern coverage curves (fraction of the 13 activity patterns recovered vs k) per noise level
- Coverage plot file, e.g. supp_fig1bcd_coverage_curves.png, corresponding to Supplementary Fig. 1b-d

## How to apply

Load the synthetic scRNA-seq benchmark from ref. 12 (20 instances of cells with 13 activity patterns). For each noise level, first apply the randomized SVD-based reduction step (an.pp.reduce_kernel, kernel space with PCA_dim=30 in the reference ACTION implementation) to obtain the low-rank approximation used by ACTION. Run the ACTION decomposition over the archetype-number sweep k = 2..30 using its nested/ranged formulation (a single runACTION call yields all k <= k_max solutions), and count nontrivial archetype columns per k; independently run plain SPA-only archetypal analysis for each k and count its nontrivial columns. Compute pattern coverage as the fraction of the 13 true activity patterns recovered as a function of k for both SPA+ACTION and SPA alone at each noise level, then plot the coverage curves (Supplementary Fig. 1b-d). The decision point is the saturation k: the rationale is that the SPA+ACTION combination should recover all patterns at every noise level and saturate at a smaller k than SPA alone.

## Related tools

- **ACTIONet** (R/Python environment providing the randomized SVD reduction step (an.pp.reduce_kernel) and the ACTION decomposition used in the benchmark) — https://github.com/shmohammadi86/ACTIONet
- **ACTION** (Reference numerical core providing runSPA (successive projection) and the nested runACTION archetype sweep k_min..k_max whose nontrivial columns are counted) — https://github.com/shmohammadi86/ACTION
- **R** (Language for the recommended stable ACTIONet R-release package used to run the sweep and coverage analysis) — https://github.com/shmohammadi86/ACTIONet

## Examples

```
S_r <- reduceGeneExpression(expression, dim = 30, method = 1); out <- runACTION(S_r, k_min = 2, k_max = 30, thread_no = 8)
```

## Evaluation signals

- The number of nontrivial archetype columns plateaus as k increases (Supplementary Fig. 1), i.e., adding archetypes beyond saturation yields only trivial columns
- SPA+ACTION coverage reaches 13/13 (all identity patterns) at every tested noise level
- ACTION-based coverage saturates (all existing patterns discovered) at a smaller k than SPA alone
- Every archetype remains a plausible cell: archetype columns C are convex mixtures of observed cells (columns sum to 1), and H gives cells as convex mixtures of archetypes
- Recovery is consistent across the 20 dataset instances at each noise level

## Limitations

- The synthetic dataset is cited only by reference number (ref. 12) without an accession or download link, so reproduction depends on obtaining it externally
- No software/tool versions are stated for ACTIONet, R, Python, UMAP, or Leiden implementations (CMake 3.19 is the only version given), which may affect exact reproducibility
- Supplementary Fig. 1 is referenced but not included in the provided article text, so the expected plateau/saturation appearance must be inferred from the validation criteria
- The SPA-only baseline requires running archetypal analysis independently for each k, whereas ACTION's nested formulation obtains all k <= 30 solutions in a single run

## Evidence

- [discussion] Synthetic scRNA-seq data sets were obtained from ref. 12. The data set contains 20 instances of cells with 13 activity patterns: "Synthetic scRNA-seq data sets were obtained from ref. 12. The data set contains 20 instances of cells with 13 activity patterns"
- [discussion] the number of nontrivial columns plateaus (Supplementary Fig. 1): "the number of nontrivial columns plateaus (Supplementary Fig. 1)"
- [discussion] an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size: "an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size"
- [methods] devtools::install_github("shmohammadi86/ACTIONet", ref = "R-release"): "devtools::install_github("shmohammadi86/ACTIONet", ref = "R-release")"
- [readme] [C_trace, H_trace] = runACTION(S_r, 2, 20, 8);: "% Run ACTION over archetype counts 2 through 20, on 8 threads
[C_trace, H_trace] = runACTION(S_r, 2, 20, 8);"
