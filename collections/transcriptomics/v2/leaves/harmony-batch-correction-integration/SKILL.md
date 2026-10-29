---
name: harmony-batch-correction-integration
description: Use when scRNA-seq count data from multiple batches/platforms (e.
when_to_use_negative:
- Single-table data with no batch/sample structure — use corral (single-table CA)
  for dimensionality reduction instead of corralm multi-table integration
- Batches cannot be restricted to a shared feature space (no intersecting gene set),
  since multi-table CA and SV comparison require a common feature space
- Complex-tissue datasets requiring independent validation of cell identities — the
  article notes benchmarking ground truth here came from simple pseudo-cell mixtures
  or pools of distinct cell types
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: corralm
  role: Multi-table correspondence analysis for alignment and batch integration of
    multiple tables, producing the cell embeddings evaluated by Harmony and SV
  repo: github.com/laurenhsu1/corral
- name: corral
  role: R/Bioconductor package implementing CA/CA-FT for single-cell count data that
    interfaces with SingleCellExperiment classes; provides the corralm integration
    call
  repo: github.com/laurenhsu1/corral
- name: Harmony
  role: Batch correction run on both corralm and PCA embeddings to obtain batch-corrected
    representations for SV comparison
- name: R
  role: Language in which the corral/corralm integration and SV assessment pipeline
    is executed
- name: Bioconductor
  role: Ecosystem providing SingleCellExperiment classes the pipeline uses for the
    multi-table structure
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:f092f3e53b40ab906803a2b30706a877fb0d2cb77ad1cba9adc852d3e752c597
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - Harmony
  - R
  - corral
  - corralm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# harmony-batch-correction-integration

## Summary

Integrates multi-batch single-cell RNA-seq data by combining multi-table correspondence analysis (corralm) with Harmony batch correction, and quantifies integration success using the scaled variance (SV) batch-mixing metric. Grounded in the SCMixology three-platform benchmark where corralm-based embeddings achieved SV near 1 while PCA embeddings fed to Harmony did not.

## When to use

Use when scRNA-seq count data from multiple batches/platforms (e.g., SingleCellExperiment tables for 10X, Celseq2, Dropseq) must be integrated across batches and the batch-mixing quality of embeddings must be quantified, particularly when comparing CA-based (corralm) vs PCA-based embeddings as Harmony input.

## When NOT to use

- Single-table data with no batch/sample structure — use corral (single-table CA) for dimensionality reduction instead of corralm multi-table integration
- Batches cannot be restricted to a shared feature space (no intersecting gene set), since multi-table CA and SV comparison require a common feature space
- Complex-tissue datasets requiring independent validation of cell identities — the article notes benchmarking ground truth here came from simple pseudo-cell mixtures or pools of distinct cell types

## Inputs

- Multi-batch scRNA-seq count matrices (e.g., SCMixology CellBench batches: 10X, Celseq2, Dropseq) as SingleCellExperiment objects
- Gene-intersected count matrices sharing a common feature space across batches
- Cell embeddings from corralm (multi-table CA) and from PCA on the same feature-intersected data

## Outputs

- Harmony batch-corrected embeddings built on corralm and on PCA embeddings
- Scaled variance (SV) batch-mixing scores per method (corralm, corralm+Harmony, PCA+Harmony)
- Comparison table written to sv_integration_results.csv (method, SV score per batch/condition)

## How to apply

Load the multi-batch scRNA-seq counts (e.g., CellBench SCMixology: 10X, Celseq2, Dropseq) into a SingleCellExperiment-based multi-table structure in R/Bioconductor and restrict all batches to the intersecting set of genes so they share a common feature space. Run corralm to compute integrated cell embeddings across the batches via multi-table correspondence analysis, which decomposes chi-squared (Pearson) residuals rather than log-transformed counts. As a baseline, compute PCA embeddings of the same gene-intersected data (the typical log(x+1) + PCA pipeline). Run Harmony batch correction on both the corralm embeddings and the PCA embeddings to obtain batch-corrected representations. Compute the scaled variance (SV) metric for corralm, corralm+Harmony, and PCA+Harmony embeddings: SV near 1 indicates successful cross-batch integration, while SV values far from 1 indicate unsuccessful integration — as observed for Harmony with PCA embeddings on SCMixology in this study.

## Related tools

- **corralm** (Multi-table correspondence analysis for alignment and batch integration of multiple tables, producing the cell embeddings evaluated by Harmony and SV) — github.com/laurenhsu1/corral
- **corral** (R/Bioconductor package implementing CA/CA-FT for single-cell count data that interfaces with SingleCellExperiment classes; provides the corralm integration call) — github.com/laurenhsu1/corral
- **Harmony** (Batch correction run on both corralm and PCA embeddings to obtain batch-corrected representations for SV comparison)
- **R** (Language in which the corral/corralm integration and SV assessment pipeline is executed)
- **Bioconductor** (Ecosystem providing SingleCellExperiment classes the pipeline uses for the multi-table structure)

## Examples

```
BiocManager::install("laurenhsu1/corral", dependencies = TRUE)
```

## Evaluation signals

- SV values close to 1 for each batch/condition indicate successful cross-batch integration; SV far from 1 (as seen for Harmony with PCA embeddings) indicates integration failure
- All batches must be restricted to the same intersecting gene set before embedding — verify identical feature space across tables
- The SV comparison table contains one row per method (corralm, corralm+Harmony, PCA+Harmony) with per-batch/condition SV scores
- Results table is written as sv_integration_results.csv with method and SV columns
- corralm and Harmony-with-corralm should both show good batch integration and outperform Harmony with PCA, per the reported finding

## Limitations

- No tool or package versions were reported for corral or its dependencies, so reproducibility requires pinning versions yourself
- Benchmarking datasets like SCMixology and Zhengmix are well-defined, simple pseudo-cell mixtures or pools of distinct cell types and do not reflect real-world tissue complexity
- Ground-truth cell type labels came from original dataset publications and were not independently validated; the article does not provide accession IDs or links for SCMixology
- SV plots in the study showed corralm and Harmony-with-corralm both achieved good integration, so SV alone may not discriminate between these two options

## Evidence

- [readme] corralm for alignment and batch integration of multiple tables: "2. `corralm` for alignment and batch integration of multiple tables"
- [abstract] Switching from PCA to CA is a simple pipeline substitution: "Switching from PCA to CA is achieved through a simple pipeline substitution and improves dimension reduction of scRNAseq datasets."
- [intro] CA computed via SVD of Pearson residuals, avoiding log-transformation: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [discussion] Benchmark datasets limited to simple mixtures, not complex tissue: "most high-quality, “ground-truth” benchmarking datasets are derived from simple “pseudo”-cell mixtures, or from pools of distinct cell types"
- [abstract] corral interfaces directly with Bioconductor single-cell classes: "We implement CA for scRNAseq data in corral, an R/Bioconductor package which interfaces directly with single cell classes in Bioconductor."
