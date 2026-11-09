---
name: multi-panel-benchmark-plotting
description: Use when you have decomposition traces over an archetype-number sweep
  (k = 2..30) on data with known ground-truth activity patterns, and must quantify/report
  how many nontrivial archetype columns each method recovers per k.
when_to_use_negative:
- Data has no ground-truth identity/activity patterns, so coverage cannot be computed
  against the 13 reference patterns
- You need a single fixed-k decomposition result (e.g. cell-state network construction),
  not a comparative sweep of nontrivial-column counts across k
- Input is already a final clustering or annotation table — this skill evaluates decomposition
  traces, not cluster-label outputs
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: ACTIONet
  role: Framework whose reduce_kernel and nested/ranged ACTION decomposition supply
    the per-k archetype traces (nontrivial columns) that are counted and plotted across
    noise levels
  repo: https://github.com/shmohammadi86/ACTIONet
- name: ACTION
  role: Reference implementation of runSPA (successive projection for simplex vertices)
    and runACTION/ranged AA used to generate SPA-only and SPA+ACTION traces over the
    k sweep
  repo: https://github.com/shmohammadi86/ACTION.git
- name: R
  role: Environment for running the ACTION R interface and producing the coverage-curve
    plots
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:82d972c973b8378920c69f8a73580290325cc784d1fdce4c6dd46efb43af6816
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTION
  - ACTIONet
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multi-panel-benchmark-plotting

## Summary

Benchmark-plotting skill for archetypal-decomposition sweeps: compute pattern-coverage curves (fraction of true identity patterns recovered) as a function of archetype number k and noise level, contrasting SPA+ACTION against SPA alone, and render them as multi-panel figures mirroring Supplementary Fig. 1b-d.

## When to use

Use when you have decomposition traces over an archetype-number sweep (k = 2..30) on data with known ground-truth activity patterns, and must quantify/report how many nontrivial archetype columns each method recovers per k.

## When NOT to use

- Data has no ground-truth identity/activity patterns, so coverage cannot be computed against the 13 reference patterns
- You need a single fixed-k decomposition result (e.g. cell-state network construction), not a comparative sweep of nontrivial-column counts across k
- Input is already a final clustering or annotation table — this skill evaluates decomposition traces, not cluster-label outputs

## Inputs

- Synthetic scRNA-seq benchmark dataset from ref. 12 (20 instances of cells with 13 activity patterns), with known ground-truth pattern labels
- Gene-by-cell count matrix (e.g. Matrix Market expression.mm with gene_names.txt) or reduced kernel matrix from an.pp.reduce_kernel / reduceGeneExpression
- ACTION decomposition traces C_trace{k}, H_trace{k} for k = 2..30 and SPA-only column assignments per k

## Outputs

- Pattern-coverage curves (fraction of the 13 activity patterns recovered vs k) for SPA+ACTION and SPA alone at each noise level
- Multi-panel figure file supp_fig1bcd_coverage_curves.png (Supplementary Fig. 1b-d analogue)
- Per-k counts of nontrivial archetype columns for each decomposition method

## How to apply

Load the synthetic scRNA-seq benchmark (ref. 12; 20 instances of cells with 13 activity patterns). For each noise level, first apply the randomized SVD-based reduction step (an.pp.reduce_kernel) to obtain the low-rank kernel approximation used by ACTION, then run the ACTION decomposition over k = 2..30 exploiting the nested/ranged formulation so all k <= 30 solutions come from a single run (runACTION with k_min/k_max), counting nontrivial archetype columns per k. Run plain SPA-only archetypal analysis independently for each k in the same sweep and count its nontrivial columns. Convert nontrivial-column counts to pattern coverage — the fraction of the 13 true activity patterns recovered — as a function of k for both SPA+ACTION and SPA alone at each noise level. Plot the resulting coverage curves (Supplementary Fig. 1b-d layout) and save as supp_fig1bcd_coverage_curves.png; the same curve data also demonstrates where the number of nontrivial columns plateaus (Supplementary Fig. 1).

## Related tools

- **ACTIONet** (Framework whose reduce_kernel and nested/ranged ACTION decomposition supply the per-k archetype traces (nontrivial columns) that are counted and plotted across noise levels) — https://github.com/shmohammadi86/ACTIONet
- **ACTION** (Reference implementation of runSPA (successive projection for simplex vertices) and runACTION/ranged AA used to generate SPA-only and SPA+ACTION traces over the k sweep) — https://github.com/shmohammadi86/ACTION.git
- **R** (Environment for running the ACTION R interface and producing the coverage-curve plots)

## Examples

```
R: S_r <- reduceGeneExpression(expression, dim = 30, method = 1); out <- runACTION(S_r, k_min = 2, k_max = 20, thread_no = 8)
```

## Evaluation signals

- Number of nontrivial columns plateaus as k increases (Supplementary Fig. 1 plateau criterion)
- SPA+ACTION coverage reaches 13/13 identity patterns at every tested noise level
- ACTION-based coverage saturates (all existing identity patterns discovered) at a smaller k than SPA alone
- Every SPA+ACTION and SPA-alone trace was computed over the same k sweep (k = 2..30) at matched noise levels, so curves are directly comparable
- Output figure file supp_fig1bcd_coverage_curves.png reproduces the Supplementary Fig. 1b-d panel structure

## Limitations

- Benchmark validity depends on the ref. 12 synthetic dataset (20 instances, 13 activity patterns); coverage claims do not transfer to datasets without ground-truth patterns
- Datasets are cited only by reference numbers without accessions or download links, so reproducing the benchmark requires obtaining ref. 12 data separately
- No software/tool versions stated for ACTIONet, R, Python, UMAP, or Leiden implementations, affecting reproducibility of the sweep
- The ACTION decomposition relies on a randomized SVD-based reduction step, so nontrivial-column counts may vary slightly across runs due to randomness

## Evidence

- [discussion] the number of nontrivial columns plateaus (Supplementary Fig. 1): "the number of nontrivial columns plateaus (Supplementary Fig. 1)"
- [abstract] superior performance on synthetic data: "Using multiple synthetic and real data sets, we demonstrate ACTIONet's superior performance relative to existing alternatives"
- [discussion] reduction step for scalable decomposition: "an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size"
- [readme] ranged ACTION call returning traces for all k: "[C_trace, H_trace] = runACTION(S_r, 2, 20, 8);

% C_trace{k} : cells x k, each archetype as a convex mixture of cells"
- [discussion] synthetic dataset description: "Synthetic scRNA-seq data sets were obtained from ref. 12. The data set contains 20 instances of cells with 13 activity patterns"
