---
name: scrnaseq-accession-data-retrieval
description: Use when a benchmark requires the original Yan, Goolam, or Deng embryo
  scRNA-seq count matrices and only a GEO/ArrayExpress accession (e.g. GSE36552) is
  known.
when_to_use_negative:
- The analysis uses a built-in package dataset such as the Goolam data shipped with
  scDHA (data('Goolam')), making a GEO/ArrayExpress download redundant
- The task requires processed or normalized expression matrices rather than raw count
  matrices retrieved from the source repository
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDHA
  role: Downstream consumer of the retrieved and preprocessed count matrices; also
    bundles a Goolam example dataset loadable via data('Goolam')
  repo: https://github.com/duct317/scDHA
- name: Monocle
  role: Competing trajectory inference method run on the same preprocessed downloaded
    datasets
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: CIDR
  role: Clustering-through-imputation package from the same ecosystem; not part of
    the retrieval step itself
  repo: https://github.com/VCCRI/CIDR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:3b841af2c119d378b877761fd2bad133540b67b3e115fcd9d8e5ddbf422f8369
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scDHA
  tools:
  - CIDR
  - Monocle
  - SCANPY
  - Slingshot
  - TSCAN
  - scDHA
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# scrnaseq-accession-data-retrieval

## Summary

Retrieve raw single-cell RNA-seq count matrices for the three mouse embryo development data sets (Yan, Goolam, Deng) from GEO/ArrayExpress using their accession numbers (GSE36552, E-MTAB-3321, GSE45719), so that downstream pseudo-time benchmarking can be reproduced.

## When to use

Use when a benchmark requires the original Yan, Goolam, or Deng embryo scRNA-seq count matrices and only a GEO/ArrayExpress accession (e.g. GSE36552) is known.

## When NOT to use

- The analysis uses a built-in package dataset such as the Goolam data shipped with scDHA (data('Goolam')), making a GEO/ArrayExpress download redundant
- The task requires processed or normalized expression matrices rather than raw count matrices retrieved from the source repository

## Inputs

- GEO/ArrayExpress accession identifiers: GSE36552 (Yan), E-MTAB-3321 (Goolam), GSE45719 (Deng)
- Repository metadata (cell-level embryonic stage/capture-time labels accompanying each accession)

## Outputs

- Per-dataset gene-by-cell raw count matrices
- Per-cell embryonic stage labels used as ground-truth capture time for R-squared computation

## How to apply

Map each dataset name in the benchmark to its repository accession: GSE36552 for Yan, E-MTAB-3321 for Goolam, and GSE45719 for Deng. Download the raw count matrices from GEO (for GSE accessions) or ArrayExpress (for E-MTAB accessions), keeping the per-cell embryonic stage labels since they serve as the known capture time for R-squared evaluation. Preprocess each count matrix (log-transform, e.g. log2(counts + 1) as in the scDHA Goolam example) before embedding cells into the scDHA low-dimensional space. Validate retrieval by confirming cell counts and stage labels match the published studies before running scDHA.pt, Monocle, TSCAN, Slingshot, or SCANPY trajectory inference.

## Related tools

- **scDHA** (Downstream consumer of the retrieved and preprocessed count matrices; also bundles a Goolam example dataset loadable via data('Goolam')) — https://github.com/duct317/scDHA
- **Monocle** (Competing trajectory inference method run on the same preprocessed downloaded datasets) — https://github.com/cole-trapnell-lab/monocle3
- **CIDR** (Clustering-through-imputation package from the same ecosystem; not part of the retrieval step itself) — https://github.com/VCCRI/CIDR

## Examples

```
library(scDHA); data('Goolam'); data <- t(Goolam$data); label <- as.character(Goolam$label); data <- log2(data + 1)
```

## Evaluation signals

- The downloaded matrices match the cell numbers of the published Yan, Goolam, and Deng embryo studies
- Per-cell stage labels are present and align with the known capture order of embryonic development stages
- Applying log2(counts + 1) preprocessing yields finite, non-negative expression values suitable for the scDHA encoder
- Downstream R-squared computation between inferred pseudo-time and capture stage reproduces the paper-reported values (scDHA.pt 0.93 vs Monocle 0.84 on Yan)

## Limitations

- The article text itself does not include a code or data availability statement within the discussion; accessions come from the task workflow, so accession-to-dataset mapping should be double-checked against the paper's supplementary materials
- ArrayExpress accessions (E-MTAB-3321) require a different retrieval route than GEO GSE accessions

## Evidence

- [results] On the Yan data set, scDHA's inferred pseudo-time better explains developmental stages than Monocle, with a higher R-squared value (0.93 vs 0.84): [evidence span withheld — anchor status: not_found]
- [readme] README Goolam example load and preprocessing: "Load Goolam dataset: `data('Goolam'); data <- t(Goolam$data); label <- as.character(Goolam$label)`"
- [readme] README log transform step: "Log transform the data: `data <- log2(data + 1)`"
