---
name: visium-dlpfc-data-processing
description: Use when input is a Visium spatial transcriptomics dataset (e.g. Visium_humanDLPFC
  from STexampleData) loaded as a SpatialExperiment, and gene filtering/normalization
  is needed before running nnSVG.
when_to_use_negative:
- The analysis goal is identifying spatial domain marker genes (mean-level changes
  between domains) rather than SVGs with continuous spatial variation
- Spot-level quality control has not yet been performed and must be executed first
  — on this example dataset it is already done
- The input is already a normalized/filtered feature table; reapplying filter_genes
  default thresholds would not be appropriate
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: nnSVG
  role: Bioconductor package providing the filter_genes function (parameters filter_genes_ncounts,
    filter_genes_pcspots, filter_mito) used to remove mitochondrial and low-expressed
    genes before SVG detection
  repo: https://bioconductor.org/packages/nnSVG
- name: STexampleData
  role: Supplies the Visium_humanDLPFC() example dataset as a SpatialExperiment object
  repo: https://bioconductor.org/packages/nnSVG
- name: scran
  role: Computes library size factors (computeLibraryFactors) and log-transformed
    normalized counts (logNormCounts) after gene filtering
  repo: https://bioconductor.org/packages/nnSVG
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:3b5d9b3e28eb85a055096f5a9135b8cae0a871fbd656a5557fe254c8ab38d3f8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - STexampleData
  - nnSVG
  - scran
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# visium-dlpfc-data-processing

## Summary

Preprocessing pipeline for the Visium human DLPFC spatial transcriptomics dataset prior to nnSVG SVG detection: tissue-spot subsetting, removal of mitochondrial and low-expressed genes with filter_genes (default thresholds of >=3 counts in >=0.5% of spots), and log-transformed normalization via scran library-size factors.

## When to use

Use when input is a Visium spatial transcriptomics dataset (e.g. Visium_humanDLPFC from STexampleData) loaded as a SpatialExperiment, and gene filtering/normalization is needed before running nnSVG.

## When NOT to use

- The analysis goal is identifying spatial domain marker genes (mean-level changes between domains) rather than SVGs with continuous spatial variation
- Spot-level quality control has not yet been performed and must be executed first — on this example dataset it is already done
- The input is already a normalized/filtered feature table; reapplying filter_genes default thresholds would not be appropriate

## Inputs

- Visium human DLPFC raw gene-by-spot count matrix as a Bioconductor SpatialExperiment object (via STexampleData::Visium_humanDLPFC())
- Spatial coordinates for spots (pxl_col_in_fullres, pxl_row_in_fullres)
- colData flag in_tissue used to keep spots over tissue

## Outputs

- Filtered SpatialExperiment with low-expressed and mitochondrial genes removed (e.g. 3,396 of 21,803 genes retained in the paper's workflow)
- 'counts' and 'logcounts' assays after computeLibraryFactors and logNormCounts
- Gene count verification table (e.g. filtered_gene_counts.csv with 21803 total, 3396 passing)

## How to apply

Load the Visium human DLPFC sample with Visium_humanDLPFC() from STexampleData (33,538 genes x 4,992 spots) into a SpatialExperiment. Subset to spots over tissue using colData(spe)$in_tissue == 1 (4,992 -> 3,639 spots); spot-level QC is already performed on this dataset. Run filter_genes(spe) from the nnSVG package with default parameters (filter_genes_ncounts, filter_genes_pcspots, filter_mito), which removes mitochondrial genes and retains genes with at least 3 counts in at least 0.5% (n = 19) of spatial locations — on the DLPFC data this retains 3,396 of 21,803 genes in the paper's workflow. Then compute library-size factors with computeLibraryFactors and log-transformed normalized counts with logNormCounts from scran, producing 'counts' and 'logcounts' assays. Filtering is also a runtime lever: because nnSVG runtime depends on the number of genes, more stringent filtering reduces it, and it must be applied before nnSVG since the model fits gene-specific length scales per gene.

## Related tools

- **nnSVG** (Bioconductor package providing the filter_genes function (parameters filter_genes_ncounts, filter_genes_pcspots, filter_mito) used to remove mitochondrial and low-expressed genes before SVG detection) — https://bioconductor.org/packages/nnSVG
- **STexampleData** (Supplies the Visium_humanDLPFC() example dataset as a SpatialExperiment object) — https://bioconductor.org/packages/nnSVG
- **scran** (Computes library size factors (computeLibraryFactors) and log-transformed normalized counts (logNormCounts) after gene filtering) — https://bioconductor.org/packages/nnSVG

## Examples

```
spe <- Visium_humanDLPFC(); spe <- spe[, colData(spe)$in_tissue == 1]; spe <- filter_genes(spe); spe <- computeLibraryFactors(spe); spe <- logNormCounts(spe)
```

## Evaluation signals

- In-tissue subsetting reduces 4,992 total spots to 3,639 spots over tissue (colData(spe)$in_tissue == 1)
- Default filter_genes removes mitochondrial genes (13 removed in the tutorial) and 30,216 of 33,525 genes due to low expression
- Applying the paper's thresholds to the Visium human DLPFC dataset retains 3,396 of 21,803 genes
- assayNames(spe) returns exactly c('counts', 'logcounts') after computeLibraryFactors and logNormCounts
- Runtime of the downstream nnSVG fit decreases with more stringent gene filtering, since runtime depends on the number of genes

## Limitations

- Runtime of nnSVG scales with the number of genes and locations; transcriptome-wide datasets with >=10,000 spatial locations require several hours on 10 processor cores of a high-performance compute cluster
- P-value calibration for low-expressed genes could be improved; adaptive filtering thresholds that would remove low-expressed genes more carefully are not yet implemented
- Parameter choices and QC metric thresholds are deferred to the nnSVG package documentation and external code repository rather than stated in the manuscript (reproducibility gap)

## Evidence

- [readme] Default gene-filtering thresholds: [evidence span withheld — anchor status: external_doc]
- [readme] Filtering scope: "# filter low-expressed and mitochondrial genes
# using function from nnSVG package with default filtering parameters
spe <- filter_genes(spe)"
- [discussion] Runtime rationale for filtering: "Since runtime depends on the number of genes, this can be reduced with more stringent gene filtering"
- [abstract] Scalability of downstream method: "scales linearly with the number of spatial locations"
