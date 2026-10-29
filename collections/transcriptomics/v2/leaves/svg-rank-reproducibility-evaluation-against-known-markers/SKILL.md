---
name: svg-rank-reproducibility-evaluation-against-known-markers
description: Use when replicating or auditing an SVG-detection result on spatially-resolved
  transcriptomics data where published per-gene rankings, adjusted p-values, and recovery
  of known biological marker genes must be verified.
when_to_use_negative:
- The goal is identifying spatial domain marker genes (mean-level changes between
  domains) rather than genes varying continuously across the tissue — the paper explicitly
  focuses on SVG methods, not spatial domain marker genes
- Multiple samples must be analyzed jointly in a principled statistical way — nnSVG
  is developed for a single sample at a time and only offers practical rank averaging
  across samples
- Genes must be grouped into gene groups or metagenes — nnSVG identifies individual
  SVGs and does not perform metagene grouping
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: nnSVG
  role: Fits per-gene nearest-neighbor Gaussian process models via BRISC, runs the
    LR test, and ranks SVGs by LR_stat with padj from multiple-testing adjustment
  repo: https://bioconductor.org/packages/nnSVG
- name: STexampleData
  role: Provides the Visium_humanDLPFC SpatialExperiment used as the benchmark dataset
    for the reproduction
  repo: https://bioconductor.org/packages/STexampleData
- name: SpatialExperiment
  role: Container format whose rowData stores nnSVG results and whose colData supplies
    the in_tissue spot filter
  repo: https://bioconductor.org/packages/SpatialExperiment
- name: scran
  role: Computes library size factors (computeLibraryFactors) and logcounts normalization
    before model fitting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:87787026e1095f4dccc96f58c177edcb61257dc039df9956efe31441668e48fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - STexampleData
  - SpatialExperiment
  - nnSVG
  - scran
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# svg-rank-reproducibility-evaluation-against-known-markers

## Summary

Reproduces the nnSVG ranking of spatially variable genes (SVGs) on the Visium human DLPFC dataset and validates it against known cortical layer markers (e.g., MOBP, PCP4, SNAP25), checking the reported 2198 significant SVGs at padj <= 0.05 and recovery of 133 of 134 layer-specific marker genes.

## When to use

Use when replicating or auditing an SVG-detection result on spatially-resolved transcriptomics data where published per-gene rankings, adjusted p-values, and recovery of known biological marker genes must be verified.

## When NOT to use

- The goal is identifying spatial domain marker genes (mean-level changes between domains) rather than genes varying continuously across the tissue — the paper explicitly focuses on SVG methods, not spatial domain marker genes
- Multiple samples must be analyzed jointly in a principled statistical way — nnSVG is developed for a single sample at a time and only offers practical rank averaging across samples
- Genes must be grouped into gene groups or metagenes — nnSVG identifies individual SVGs and does not perform metagene grouping

## Inputs

- Visium_humanDLPFC SpatialExperiment object from the STexampleData Bioconductor package (33538 genes x 4992 spots, 3639 in-tissue)
- Matrix of log-transformed normalized counts (logcounts) plus a numeric matrix of spatial coordinates (alternative input format)

## Outputs

- Ranked per-gene results table in rowData of the SpatialExperiment: gene_id, gene_name, LR_stat, rank, pval, padj, prop_sv, phi, sigma.sq, tau.sq
- nnsvg_visium_dlpfc_results.csv containing gene IDs, LR statistics, adjusted p-values, and ranks
- Count of significant SVGs at padj <= 0.05 (expected: 2198)

## How to apply

Load the Visium_humanDLPFC SpatialExperiment from the STexampleData Bioconductor package, keep spots with in_tissue == 1, and run filter_genes() with default parameters (removes mitochondrial genes; retains genes with at least 3 counts in at least 0.5% of locations) to reach the ~3396-gene set, then compute logcounts via scran's computeLibraryFactors and logNormCounts. Fit per-gene nearest-neighbor Gaussian process models with nnSVG (set.seed(123)), which estimates gene-specific length scale parameters (phi) and runs the likelihood ratio (LR) test; results land in rowData with columns LR_stat, rank, pval, padj, and prop_sv. Adjust p-values for multiple testing, rank genes by LR_stat, and declare significance at padj <= 0.05. Judge success by (1) reproducing 2198 significant SVGs overall, (2) confirming MOBP, PCP4, SNAP25, HBB, IGKC, and NPY fall within the top 100/top 1000 ranks, and (3) recovering 133 of the 134 filtered layer-specific marker genes as significant. Write the ranked per-gene table (gene IDs, LR statistics, adjusted p-values, ranks) to nnsvg_visium_dlpfc_results.csv.

## Related tools

- **nnSVG** (Fits per-gene nearest-neighbor Gaussian process models via BRISC, runs the LR test, and ranks SVGs by LR_stat with padj from multiple-testing adjustment) — https://bioconductor.org/packages/nnSVG
- **STexampleData** (Provides the Visium_humanDLPFC SpatialExperiment used as the benchmark dataset for the reproduction) — https://bioconductor.org/packages/STexampleData
- **SpatialExperiment** (Container format whose rowData stores nnSVG results and whose colData supplies the in_tissue spot filter) — https://bioconductor.org/packages/SpatialExperiment
- **scran** (Computes library size factors (computeLibraryFactors) and logcounts normalization before model fitting)

## Examples

```
library(nnSVG); library(STexampleData); spe <- Visium_humanDLPFC(); spe <- spe[, colData(spe)$in_tissue == 1]; spe <- filter_genes(spe); spe <- computeLibraryFactors(spe); spe <- logNormCounts(spe); set.seed(123); spe <- nnSVG(spe, n_threads = 1)
```

## Evaluation signals

- Number of significant SVGs at padj <= 0.05 matches the published 2198
- MOBP, PCP4, SNAP25, HBB, IGKC, and NPY are all present within the top 100/top 1000 ranks (MOBP is rank 1 in the README example)
- 133 of the 134 layer-specific marker genes that pass filtering are recovered as statistically significant SVGs
- rowData contains the expected result columns: LR_stat, rank, pval, padj, prop_sv, and gene-specific length scale phi
- Output CSV nnsvg_visium_dlpfc_results.csv contains one row per filtered gene (~3396) with gene IDs, LR statistics, adjusted p-values, and ranks

## Limitations

- Some small negative values appear in estimated LR statistics, which are difficult to interpret and occur mainly for lower-ranked genes
- P-value calibration for low-expressed genes could be improved; adaptive filtering thresholds have been proposed but are not yet implemented
- Effect size (proportion of spatial variance, prop_sv) does not distinguish technical from biological variance
- Transcriptome-wide datasets with >=10,000 spatial locations require runtimes of several hours on 10 processor cores of a high-performance compute cluster

## Evidence

- [results] nnSVG recovered nearly all known layer markers: 134 of 198 layer-specific marker genes passed filtering, and 133 of these 134 were identified as statistically significant SVGs, out of 2198 significant SVGs overall.: [evidence span withheld — anchor status: not_found]
- [readme] README workflow description: [evidence span withheld — anchor status: external_doc]
- [discussion] Discussion on length scale flexibility: "thereby leading to reduced flexibility to identify SVGs with different spatial ranges in expression"
- [intro] Intro scope statement: "Here, we are interested in methods to identify SVGs, not spatial domain marker genes. Therefore, we focus on methods to identify SVGs in this work."
- [readme] README preprocessing steps: [evidence span withheld — anchor status: external_doc]
