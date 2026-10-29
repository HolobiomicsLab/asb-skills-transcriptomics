---
name: single-node-runtime-benchmarking
description: Use when comparing statistical methods for co-expression detection and
  needing per-method wall-clock runtimes measured on identical simulated scRNA-seq
  inputs on one node.
when_to_use_negative:
- When accuracy alone is needed — this skill times methods but derives its precision-recall
  curves separately from BH-adjusted p-values over gene pairs
- When the goal is benchmarking on multi-node or distributed infrastructure — the
  paper's validation target is runtimes on a single Xeon Gold-class node
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: CS-CORE / CSCORE (R package)
  role: Method under benchmark; run via CSCORE IRLS/WLS on the simulated counts with
    wall-clock timing
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: locCSN
  role: Competing co-expression hypothesis-test method applied to the same simulated
    counts and timed
  repo: https://github.com/xuranw/locCSN
- name: normalisr
  role: Competing co-expression hypothesis-test method applied to the same simulated
    counts and timed
  repo: https://github.com/lingfeiwang/normalisr
- name: NoiseRegularization
  role: Competing preprocessing/co-expression method applied to the same simulated
    counts and timed
  repo: https://github.com/RuoyuZhang/NoiseRegularization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:bbf7bdc33fe65612b12d55bcd5a363621415802f67ae369f3ad5b39ff7a5739f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CS-CORE / CSCORE (R package)
  - CSCORE (R package)
  - NoiseRegularization
  - locCSN
  - normalisr
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-node-runtime-benchmarking

## Summary

Benchmark wall-clock running time of co-expression inference methods (CS-CORE, Pearson, Spearman, locCSN, normalisr, NoiseRegularization) on identical simulated UMI count inputs on a single Xeon Gold-class node, so computational competitiveness can be reported alongside detection accuracy (Fig. 3c).

## When to use

Use when comparing statistical methods for co-expression detection and needing per-method wall-clock runtimes measured on identical simulated scRNA-seq inputs on one node.

## When NOT to use

- When accuracy alone is needed — this skill times methods but derives its precision-recall curves separately from BH-adjusted p-values over gene pairs
- When the goal is benchmarking on multi-node or distributed infrastructure — the paper's validation target is runtimes on a single Xeon Gold-class node

## Inputs

- Simulated UMI count matrix (p=500 genes x n=5000 cells) generated from the Poisson measurement model x_ij|z_ij ~ Poisson(s_i * z_ij)
- Sparse ground-truth co-expression matrix (from CS-CORE on ref. 17 postmortem Alzheimer's disease scRNA-seq data)
- Installed co-expression method implementations: CSCORE (R package), locCSN, normalisr, NoiseRegularization, plus base Pearson/Spearman

## Outputs

- Per-method wall-clock running time table
- Timing values paired with precision-recall curves for the same p x p gene-pair tests (Fig. 3c-style benchmark panel)

## How to apply

Simulate UMI count data with the latent-variable Poisson measurement model x_ij|z_ij ~ Poisson(s_i * z_ij) for a fixed problem size (p=500 genes, n=5000 cells), using a sparse ground-truth co-expression matrix (in the paper, estimated with CS-CORE from ref. 17 postmortem Alzheimer's disease scRNA-seq data). Apply each hypothesis-testing co-expression method to the same simulated counts — CS-CORE via CSCORE IRLS/WLS, Pearson, Spearman, locCSN, normalisr, and NoiseRegularization — wrapping each call with a wall-clock timer. Record runtimes per method into a table so they can be plotted alongside precision-recall curves derived from BH-adjusted p-values over all p x p gene pairs. Report runtimes as measured on a single Xeon Gold-class node so the timing is reproducible and comparable to Fig. 3c of the source paper.

## Related tools

- **CS-CORE / CSCORE (R package)** (Method under benchmark; run via CSCORE IRLS/WLS on the simulated counts with wall-clock timing) — https://github.com/ChangSuBiostats/CS-CORE
- **locCSN** (Competing co-expression hypothesis-test method applied to the same simulated counts and timed) — https://github.com/xuranw/locCSN
- **normalisr** (Competing co-expression hypothesis-test method applied to the same simulated counts and timed) — https://github.com/lingfeiwang/normalisr
- **NoiseRegularization** (Competing preprocessing/co-expression method applied to the same simulated counts and timed) — https://github.com/RuoyuZhang/NoiseRegularization

## Examples

```
R: system.time({CSCORE_result <- CSCORE(sim_counts, genes = genes_selected)})  # wall-clock timing wrapped around the timed method call
```

## Evaluation signals

- All methods run on the identical simulated count matrix (p=500, n=5000), so runtime differences reflect implementation, not input differences
- A per-method running-time table is produced with entries for all six methods (CS-CORE, Pearson, Spearman, locCSN, normalisr, NoiseRegularization)
- Timing runs are anchored to a single Xeon Gold-class node so the qualitative pattern is comparable to Fig. 3c
- CS-CORE remains computationally competitive with competing methods while achieving the highest PR AUC (Fig. 3b,c pattern)

## Limitations

- Runtimes are hardware-dependent; the paper's validation target is a single Xeon Gold-class node, so absolute times may not transfer to other hardware
- The simulation uses a Poisson measurement model only; the article notes adapting CS-CORE to other count distributions such as negative binomial 'is deferred to future work', so timing conclusions are tied to this model
- CS-CORE assumes cells of the same cell type share one expression distribution, which constrains the simulation realism used for the benchmark

## Evidence

- [results] CS-CORE achieved the highest area-under-the-curve value among methods with statistical tests in the p=500, n=5000 detection simulations.: [evidence span withheld — anchor status: not_found]
- [results] CS-CORE remains computationally competitive (task card workflow step 5 validation): [evidence span withheld — anchor status: not_found]
- [intro] CS-CORE implements a fast IRLS approach for estimation and testing: "CS-CORE implements a fast and efficient iteratively re-weighted least squares approach for estimating the true correlations between underlying expression levels, together with a theoretically"
- [methods] CSCORE IRLS implementation available in the R package: "We provide these two options in [CSCORE_IRLS_cpp](CSCORE_IRLS_cpp.html), which is the function underlying `CSCORE`."
- [readme] normalisr installation and computational efficiency: [evidence span withheld — anchor status: external_doc]
