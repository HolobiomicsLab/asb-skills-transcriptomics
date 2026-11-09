---
name: trajectory-inference-benchmarking
description: Use when comparing single-cell trajectory/pseudo-time inference methods
  on scRNA-seq data with known capture or developmental stages, to quantify which
  tool's ordering explains real time via R-squared.
when_to_use_negative:
- No gold-standard capture time or developmental stage labels exist, so R-squared
  against real time cannot be computed
- The scVI package alone is used as the trajectory tool — per the article, 'The scVI
  package itself is not capable of clustering, visualization, classification, and
  pseudo-time inference'
- Input data are not raw count matrices requiring the scDHA preprocessing/embedding
  step (e.g., already a reduced feature table)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDHA
  role: Produces the low-dimensional embedding via non-negative kernel autoencoder
    + stacked Bayesian self-learning, and infers pseudo-time with scDHA.pt
  repo: github.com/duct317/scDHA
- name: Monocle
  role: State-of-the-art competitor trajectory inference tool run on the same preprocessed
    datasets
  repo: cole-trapnell-lab/monocle3
- name: TSCAN
  role: State-of-the-art competitor trajectory inference tool
- name: Slingshot
  role: State-of-the-art competitor trajectory inference tool
- name: SCANPY
  role: State-of-the-art competitor (PAGA-based trajectory inference)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:dc7e55f4f6dde96ff8a0d32cff43c6b7bb44194bd1b67d4a58b7b29e72290aee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle
  - SCANPY
  - Slingshot
  - TSCAN
  - scDHA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# trajectory-inference-benchmarking

## Summary

Benchmarks pseudo-time inference from scDHA's hierarchical autoencoder embedding against Monocle, TSCAN, Slingshot, and SCANPY by computing R-squared between each tool's inferred pseudo-time and known embryonic capture stages on the Yan, Goolam, and Deng mouse embryo scRNA-seq datasets.

## When to use

Use when comparing single-cell trajectory/pseudo-time inference methods on scRNA-seq data with known capture or developmental stages, to quantify which tool's ordering best explains real time via R-squared.

## When NOT to use

- No gold-standard capture time or developmental stage labels exist, so R-squared against real time cannot be computed
- The scVI package alone is used as the trajectory tool — per the article, 'The scVI package itself is not capable of clustering, visualization, classification, and pseudo-time inference'
- Input data are not raw count matrices requiring the scDHA preprocessing/embedding step (e.g., already a reduced feature table)

## Inputs

- Mouse embryo scRNA-seq count matrices from GSE36552 (Yan), E-MTAB-3321, and GSE45719 (Deng)
- Known embryonic capture/stage labels per cell for R-squared computation
- scDHA low-dimensional embedding (output of the scDHA dimension-reduction step, log2-transformed input)

## Outputs

- Per-cell pseudo-time orderings from scDHA.pt, Monocle, TSCAN, Slingshot, and SCANPY
- R-squared values per tool per dataset, assembled as pseudo_time_r2_results.csv

## How to apply

Download the three mouse embryo datasets via accessions GSE36552 (Yan), E-MTAB-3321, and GSE45719 (Deng), preprocess each count matrix (e.g., log2 transform counts as in the scDHA Goolam example: log2(data + 1)), and embed cells into the scDHA low-dimensional space using scDHA's non-negative kernel autoencoder followed by the stacked Bayesian self-learning module. Run scDHA.pt on the scDHA embedding to infer pseudo-time — per the README, 'scDHA.pt: generating pseudotime. The input is demension reduction output.' Then run the four state-of-the-art competitors (Monocle, TSCAN, Slingshot, SCANPY PAGA-based) on the same preprocessed data. For each tool and dataset, compute R-squared between inferred pseudo-time and known embryonic stage, assemble the values into a comparison table (pseudo_time_r2_results.csv), and set a seed (e.g., seed = 1) so scDHA results are reproducible.

## Related tools

- **scDHA** (Produces the low-dimensional embedding via non-negative kernel autoencoder + stacked Bayesian self-learning, and infers pseudo-time with scDHA.pt) — github.com/duct317/scDHA
- **Monocle** (State-of-the-art competitor trajectory inference tool run on the same preprocessed datasets) — cole-trapnell-lab/monocle3
- **TSCAN** (State-of-the-art competitor trajectory inference tool)
- **Slingshot** (State-of-the-art competitor trajectory inference tool)
- **SCANPY** (State-of-the-art competitor (PAGA-based trajectory inference))

## Examples

```
data('Goolam'); data <- t(Goolam$data); data <- log2(data + 1); result <- scDHA(data, seed = 1); pt <- scDHA.pt(result)
```

## Evaluation signals

- Reproduced R-squared for scDHA.pt on Yan must be 0.93 versus Monocle's 0.84
- scDHA.pt must achieve the highest R-squared among all five tools on each of the three datasets
- The assembled comparison table must match the paper-reported R-squared values across scDHA.pt, Monocle, TSCAN, Slingshot, and SCANPY
- All five tools must be run on the same preprocessed count matrices to ensure a fair comparison
- scDHA results must be reproducible by setting a seed (README: 'The result is reproducible by setting seed for these functions')

## Limitations

- Accuracy claims in the discussion are stated as expectations rather than demonstrated results ('scDHA is user-friendly and is expected to be more accurate than existing autoencoders')
- Comparative analyses are partly deferred to supplementary materials (e.g., scVI comparisons in Supplementary Section 6 and Fig. 25–32)
- scDHA requires Torch; older versions required a Python/TensorFlow environment, and libtorch must be installed (torch::install_torch())

## Evidence

- [intro] state-of-the-art trajectory tools: "Among a number of trajectory inference tools, Monocle25, TSCAN26, Slingshot27, and SCANPY13 are considered state-of-the-art"
- [readme] scDHA.pt input contract: [evidence span withheld — anchor status: external_doc]
- [intro] pseudo-time as a scDHA sub-field: "the scDHA software package conducts cell segregation through unsupervised learning, dimension reduction and visualization, cell classification, and time-trajectory inference"
- [readme] reproducibility via seed: [evidence span withheld — anchor status: external_doc]
