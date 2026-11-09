---
name: single-cell-log-normalization
description: Use when raw UMI count matrices must be transformed to a common sequencing-depth
  scale via log(1 + scaled expression) — e.g., for exploratory visualization, marker
  plots, or when reproducing benchmarks of how downstream co-expression estimators
  behave on log-normalized data.
when_to_use_negative:
- When the downstream goal is statistically justified cell-type-specific co-expression
  inference — the article shows log-normalized correlations (e.g., locCSN = 0.61,
  propr = 0.50 for an independent pair) are inflated false positives; use CS-CORE
  on raw UMI counts instead
- When the measurement model should account for count distribution assumptions beyond
  Poisson — the article defers negative-binomial adaptations to future work, so log-normalization
  does not substitute for a count-level model
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Provides the scRNA-seq object infrastructure and normalization pipeline within
    which log-normalized counts are computed and stored
- name: CS-CORE
  role: Proposed alternative that models raw UMI counts with a measurement model for
    sequencing depth and measurement error, avoiding log-normalized correlations
  repo: github.com/ChangSuBiostats/CS-CORE
- name: CSCORE (R package)
  role: R implementation used to run CS-CORE in the benchmark workflows instead of
    computing correlations on log-normalized data
  repo: github.com/ChangSuBiostats/CS-CORE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:2896dd84d6e84b546ed884760844c3d75a28393e9374909f6d710ef05efac090
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CS-CORE
  - CSCORE (R package)
  - Seurat
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# single-cell-log-normalization

## Summary

Transforms scRNA-seq UMI counts into log-normalized values, log(1 + scaled expression), where scaled expression is raw counts multiplied by sequencing-depth scale factors. In this paper's evaluations, applying co-expression methods (locCSN, Pearson, propr, Spearman, SpQN) on such log-normalized counts inflated false-positive co-expression even for truly independent gene pairs, motivating CS-CORE's model-based alternative.

## When to use

Use when raw UMI count matrices must be transformed to a common sequencing-depth scale via log(1 + scaled expression) — e.g., for exploratory visualization, marker plots, or when reproducing benchmarks of how downstream co-expression estimators behave on log-normalized data.

## When NOT to use

- When the downstream goal is statistically justified cell-type-specific co-expression inference — the article shows log-normalized correlations (e.g., locCSN = 0.61, propr = 0.50 for an independent pair) are inflated false positives; use CS-CORE on raw UMI counts instead
- When the measurement model should account for count distribution assumptions beyond Poisson — the article defers negative-binomial adaptations to future work, so log-normalization does not substitute for a count-level model

## Inputs

- Raw UMI count matrix (genes × cells) from scRNA-seq, e.g., counts in a Seurat object such as blish_covid.seu.rds
- Simulated negative-binomial UMI counts for a gene pair (benchmark scenario)

## Outputs

- Scaled count matrix (raw counts multiplied by sequencing-depth scale factors)
- Log-normalized expression matrix, log(1 + scaled expression)
- Benchmark outputs such as coexpression_estimates_independent_pair.csv containing per-method estimates (locCSN=0.61, Pearson=0.14, propr=0.50, Spearman=0.07, SpQN=0.14 for an independent pair)

## How to apply

First compute sequencing-depth scale factors from the raw UMI count matrix (e.g., library-size-based size factors per cell) and derive scaled counts by multiplying each cell's raw counts by its scale factor. Then apply the log transform log(1 + scaled expression) to obtain the log-normalized matrix used by downstream methods (e.g., Seurat-style normalized data). In this paper's Fig. 2 benchmark, the transform was applied to simulated negative-binomial UMI counts for two independent genes with mean ranks 269 and 351 among 28,412 genes in excitatory neurons, then locCSN, Pearson, propr, Spearman, and SpQN were applied to the log-normalized pair. The rationale for caution: the article reports that co-expression correlations computed on log-normalized UMI data are confounded by sequencing-depth variation and measurement error, yielding inflated false positives. In CS-CORE's vignette workflow, log-normalization is confined to preprocessing for the co-expression tests themselves, which instead model raw UMI counts through a measurement model.

## Related tools

- **Seurat** (Provides the scRNA-seq object infrastructure and normalization pipeline within which log-normalized counts are computed and stored)
- **CS-CORE** (Proposed alternative that models raw UMI counts with a measurement model for sequencing depth and measurement error, avoiding log-normalized correlations) — github.com/ChangSuBiostats/CS-CORE
- **CSCORE (R package)** (R implementation used to run CS-CORE in the benchmark workflows instead of computing correlations on log-normalized data) — github.com/ChangSuBiostats/CS-CORE

## Examples

```
scaled_counts <- sweep(raw_counts, 2, colSums(raw_counts), '/'); lognorm <- log1p(scaled_counts)
```

## Evaluation signals

- On a simulated independent negative-binomial gene pair (mean ranks 269 and 351 among 28,412 genes), methods on log-normalized data should reproduce locCSN = 0.61, Pearson = 0.14 (p < 0.01), propr = 0.50, Spearman = 0.07 (p = 0.02), SpQN = 0.14
- The five reproduced estimates should be written to coexpression_estimates_independent_pair.csv and match the reported values
- Validation expectation: large locCSN (0.61) and propr (0.50) values on a truly independent pair demonstrate inflated false positives from log-normalized single-cell data
- Scaled counts must equal raw counts multiplied by per-cell sequencing-depth scale factors before applying log(1 + x)

## Limitations

- Log-normalized correlations are seriously confounded by varying sequencing depths across cells, producing inflated false-positive co-expression findings
- Log normalization itself does not model measurement errors or correct estimation bias; the article found most existing methods on such data had inflated false positives and biased co-expression estimates
- Even count-model alternatives like CS-CORE currently use only a Poisson measurement model; negative-binomial adaptations are deferred to future work

## Evidence

- [other] Workflow step 2: log-normalization formula: [evidence span withheld — anchor status: not_found]
- [results] Finding: spurious co-expression from log-normalized independent pair: [evidence span withheld — anchor status: not_found]
- [intro] Confounding by sequencing depth: "gene co-expressions measured via correlations of UMI counts across cells can be seriously confounded by varying sequencing depths, resulting in inflated false positive findings"
- [abstract] Methods performance on log-normalized data: "most existing methods suffered from inflated false positives as well as biased co-expression estimates and clustering analysis"
- [readme] CS-CORE as the model-based alternative: "`CS-CORE` is a R package for cell-type-specific co-expression inference from single cell RNA-sequencing data."
