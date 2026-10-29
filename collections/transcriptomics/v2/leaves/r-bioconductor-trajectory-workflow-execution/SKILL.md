---
name: r-bioconductor-trajectory-workflow-execution
description: Use when comparing pseudo-time/trajectory inference performance across
  methods on scRNA-seq data with known capture or developmental stage times, especially
  embryo development datasets evaluated by R-squared between inferred pseudo-time
  and real time.
when_to_use_negative:
- The input data lack any known capture time or developmental stage labels, so no
  R-squared against real time can be computed
- The task is clustering or cell classification, where scDHA's main function (adjusted
  Rand Index via mclust) is the appropriate output rather than scDHA.pt
- The dataset is a scVI-style latent representation — the article notes the scVI package
  alone does not support pseudo-time inference and requires combining it with downstream
  methods
tools:
- name: scDHA
  role: Generates the low-dimensional embedding (non-negative kernel autoencoder +
    stacked Bayesian self-learning module) and infers pseudo-time via scDHA.pt
  repo: https://github.com/duct317/scDHA
- name: Monocle
  role: State-of-the-art trajectory inference baseline run on the same preprocessed
    datasets for R-squared comparison
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: TSCAN
  role: State-of-the-art trajectory inference baseline producing a competing pseudo-time
    ordering
- name: Slingshot
  role: State-of-the-art trajectory inference baseline producing a competing pseudo-time
    ordering
- name: SCANPY
  role: PAGA-based trajectory analysis baseline producing a competing pseudo-time
    ordering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:5bb2e27bab0f74a4c103c1787c4c227a6561b756d8c13d95b277365083cd34aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scDHA
  tools:
  - Monocle
  - SCANPY
  - Slingshot
  - TSCAN
  - scDHA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# r-bioconductor-trajectory-workflow-execution

## Summary

Benchmark single-cell pseudo-time inference by running scDHA.pt on the scDHA low-dimensional embedding and comparing its R-squared values against Monocle, TSCAN, Slingshot, and SCANPY on the Yan (GSE36552), Goolam (E-MTAB-3321), and Deng (GSE45719) mouse embryo scRNA-seq datasets.

## When to use

Use when comparing pseudo-time/trajectory inference performance across methods on scRNA-seq data with known capture or developmental stage times, especially embryo development datasets evaluated by R-squared between inferred pseudo-time and real time.

## When NOT to use

- The input data lack any known capture time or developmental stage labels, so no R-squared against real time can be computed
- The task is clustering or cell classification, where scDHA's main function (adjusted Rand Index via mclust) is the appropriate output rather than scDHA.pt
- The dataset is a scVI-style latent representation — the article notes the scVI package alone does not support pseudo-time inference and requires combining it with downstream methods

## Inputs

- Raw count matrices of mouse embryo scRNA-seq datasets: Yan (GSE36552), Goolam (E-MTAB-3321), Deng (GSE45719)
- Known embryonic capture/stage time labels per cell (e.g., Goolam$label)
- Preprocessed, log-transformed expression matrix (rows as cells/samples, columns as genes)

## Outputs

- Per-cell pseudo-time orderings from scDHA.pt, Monocle, TSCAN, Slingshot, and SCANPY
- R-squared values between each tool's pseudo-time and real developmental time per dataset
- Comparison table saved as pseudo_time_r2_results.csv

## How to apply

Download the three embryo scRNA-seq count matrices from GEO/ArrayExpress (GSE36552 for Yan, E-MTAB-3321 for Goolam, GSE45719 for Deng), preprocess and log-transform (e.g., log2(counts+1) as in the scDHA Goolam example), and embed cells into the scDHA low-dimensional space via the non-negative kernel autoencoder plus stacked Bayesian self-learning module (scDHA() with a fixed seed for reproducibility). Apply scDHA.pt to the scDHA dimension-reduction output to infer each cell's pseudo-time. In parallel, run Monocle, TSCAN, Slingshot, and SCANPY (PAGA-based) on the same preprocessed datasets to obtain their pseudo-time orderings. For every tool and dataset, compute the R-squared between inferred pseudo-time and known embryonic capture/stage time, assemble the values into a comparison table (pseudo_time_r2_results.csv), and check it against paper-reported values: scDHA.pt 0.93 vs Monocle 0.84 on Yan, with scDHA.pt ranking highest on all three datasets.

## Related tools

- **scDHA** (Generates the low-dimensional embedding (non-negative kernel autoencoder + stacked Bayesian self-learning module) and infers pseudo-time via scDHA.pt) — https://github.com/duct317/scDHA
- **Monocle** (State-of-the-art trajectory inference baseline run on the same preprocessed datasets for R-squared comparison) — https://github.com/cole-trapnell-lab/monocle3
- **TSCAN** (State-of-the-art trajectory inference baseline producing a competing pseudo-time ordering)
- **Slingshot** (State-of-the-art trajectory inference baseline producing a competing pseudo-time ordering)
- **SCANPY** (PAGA-based trajectory analysis baseline producing a competing pseudo-time ordering)

## Examples

```
library(scDHA); data('Goolam'); data <- t(Goolam$data); data <- log2(data + 1); result <- scDHA(data, seed = 1); pt <- scDHA.pt(result)
```

## Evaluation signals

- The reproduced pseudo_time_r2_results.csv matches paper-reported values, with scDHA.pt at 0.93 vs Monocle 0.84 on the Yan dataset
- scDHA.pt achieves the highest R-squared among the five tools on every one of the three datasets (Yan, Goolam, Deng)
- Repeated runs with a fixed seed (e.g., seed = 1 in scDHA()) reproduce the same pseudo-time ordering, confirming reproducibility
- R-squared values for all tools fall in [0, 1] and correlate positively with embryonic capture/stage time
- The scDHA embedding is generated by the two-module pipeline (non-negative kernel autoencoder filtering insignificant genes/components, then the stacked Bayesian autoencoder) before scDHA.pt is applied

## Limitations

- The discussion contains no explicit limitations or caveats section, and accuracy claims in the discussion are stated as expectations rather than demonstrated results
- The comparative claim against scVI is deferred to supplementary materials (Supplementary Section 6 and Fig. 25–32) rather than presented in the main discussion
- Pseudo-time quality depends on first projecting cells into the scDHA low-dimensional space; scDHA.pt's input must be the dimension-reduction output, not raw counts

## Evidence

- [intro] state-of-the-art trajectory baselines: "Among a number of trajectory inference tools, Monocle25, TSCAN26, Slingshot27, and SCANPY13 are considered state-of-the-art"
- [intro] scDHA scope covers trajectory inference: "the scDHA software package conducts cell segregation through unsupervised learning, dimension reduction and visualization, cell classification, and time-trajectory inference"
- [readme] scDHA.pt usage contract: [evidence span withheld — anchor status: external_doc]
- [readme] Goolam preprocessing example: "Load Goolam dataset: `data('Goolam'); data <- t(Goolam$data); label <- as.character(Goolam$label)` - Log transform the data: `data <- log2(data + 1)` - Generating clustering result: `result <-"
- [intro] scDHA two-module design: "The first module is a non-negative kernel autoencoder that provides a non-negative, part-based representation of the data."
