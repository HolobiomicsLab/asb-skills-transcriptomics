---
name: spatial-transcriptomics-gene-filtering
description: Use when preparing spot-level count matrices (e.g. a SpatialExperiment
  with raw counts) for SVG detection or spatial clustering, where low-expression and
  mitochondrial genes must be removed to reduce runtime and improve p-value calibration.
when_to_use_negative:
- Input is already a filtered, normalized feature table or logcounts matrix — filter_genes
  expects raw counts and re-running it would not match the intended pipeline
- The goal is spatial-domain marker detection (mean-level changes between annotated
  domains) rather than continuous SVG detection, where different preprocessing choices
  may apply
- Single-cell data without spatial coordinates, where the percent-of-spots threshold
  is not meaningful
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: nnSVG
  role: Provides the filter_genes module (filter_genes_ncounts, filter_genes_pcspots,
    filter_mito) used for per-gene QC prior to NNGP model fitting
  repo: https://bioconductor.org/packages/nnSVG
- name: Bioconductor
  role: Framework hosting nnSVG and the SpatialExperiment/STexampleData packages used
    to load and hold the count data
  repo: https://bioconductor.org/packages/nnSVG
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_002
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:964409ec98b574c4f3d95cfcbbeae509bfa2cd50f8f7069dc6a3fc5cdf0a4c87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - SPARK-X
  - nnSVG
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# spatial-transcriptomics-gene-filtering

## Summary

Per-gene quality-control filtering for spatially-resolved transcriptomics data using the nnSVG package's filter_genes module, which removes mitochondrial genes and low-expressed genes via UMI-count and percent-of-spots thresholds before SVG detection.

## When to use

Use when preparing spot-level count matrices (e.g. a SpatialExperiment with raw counts) for SVG detection or spatial clustering, where low-expression and mitochondrial genes must be removed to reduce runtime and improve p-value calibration.

## When NOT to use

- Input is already a filtered, normalized feature table or logcounts matrix — filter_genes expects raw counts and re-running it would not match the intended pipeline
- The goal is spatial-domain marker detection (mean-level changes between annotated domains) rather than continuous SVG detection, where different preprocessing choices may apply
- Single-cell data without spatial coordinates, where the percent-of-spots threshold is not meaningful

## Inputs

- SpatialExperiment Bioconductor object with raw gene-by-spot count matrix (e.g. Visium_humanDLPFC from STexampleData: 33,538 genes x 4,992 spots before in-tissue filtering)
- Alternatively, a raw count matrix plus spatial coordinates matrix
- Filtering parameters: filter_genes_ncounts, filter_genes_pcspots, filter_mito

## Outputs

- Filtered SpatialExperiment retaining only genes passing the count/spots thresholds with mitochondrial genes removed (e.g. 3,396 of 21,803 genes on Visium human DLPFC)
- Gene count summary (total genes, mitochondrial genes removed, genes removed due to low expression, genes retained)
- Filtered gene count table (e.g. filtered_gene_counts.csv)

## How to apply

Apply the filter_genes function from the nnSVG Bioconductor package to a SpatialExperiment object containing raw counts (after keeping only spots over tissue, e.g. colData(spe)$in_tissue == 1). The module exposes three parameters: filter_genes_ncounts (default requiring at least 3 UMI counts per gene), filter_genes_pcspots (default requiring detection in at least 0.5% of spatial locations, e.g. n = 19 spots on the 3639-spot Visium human DLPFC sample), and filter_mito (remove mitochondrial genes, 13 removed on the DLPFC example). The rationale is that filtering lowly expressed genes both reduces nnSVG runtime (which scales with the number of genes) and improves p-value calibration for low-expressed genes, though the paper notes adaptive filtering thresholds are not yet implemented. On the Visium human DLPFC dataset this step retains 3396 of 21,803 genes in the published analysis (30,216 of 33,525 removed in the README example with default parameters). Verify by counting genes before and after filtering and confirming agreement with the published retained-gene count before proceeding to logcounts normalization (scran computeLibraryFactors + logNormCounts) and nnSVG fitting.

## Related tools

- **nnSVG** (Provides the filter_genes module (filter_genes_ncounts, filter_genes_pcspots, filter_mito) used for per-gene QC prior to NNGP model fitting) — https://bioconductor.org/packages/nnSVG
- **Bioconductor** (Framework hosting nnSVG and the SpatialExperiment/STexampleData packages used to load and hold the count data) — https://bioconductor.org/packages/nnSVG

## Examples

```
spe <- filter_genes(spe)
```

## Evaluation signals

- Gene retention count matches the published result: 3396 of 21803 genes retained on the Visium human DLPFC sample (21803 total, 3396 passing)
- README reference run with default parameters removes 13 mitochondrial genes and 30216 of 33525 genes due to low expression after in-tissue spot filtering
- The removal log lines report the exact thresholds applied ('at least 3 counts in at least 0.5% (n = 19) of spatial locations')
- All genes in the filtered object should satisfy the thresholds (>= 3 UMI counts in >= 0.5% of spots; no mitochondrial genes present)
- Stricter filtering should reduce nnSVG runtime, since runtime depends on the number of genes

## Limitations

- P-value calibration for low-expressed genes could be improved; the proposed fix — adaptive filtering thresholds that carefully remove low-expressed genes — is not yet implemented
- The default thresholds (3 UMI counts, 0.5% of spots) are fixed, not adaptive, so overly permissive filtering may leave poorly calibrated genes in the test
- More stringent gene filtering reduces runtime but changes the set of genes available for SVG ranking, affecting comparability with published runtimes and results

## Evidence

- [readme] Default thresholds and module parameters on the DLPFC example: [evidence span withheld — anchor status: external_doc]
- [readme] Filtering function from the nnSVG package with default parameters: "# filter low-expressed and mitochondrial genes
# using function from nnSVG package with default filtering parameters
spe <- filter_genes(spe)"
- [discussion] Rationale: gene filtering reduces runtime: "Since runtime depends on the number of genes, this can be reduced with more stringent gene filtering"
- [discussion] Proposed but unimplemented improvement to filtering: "This could be improved by developing adaptive filtering thresholds that carefully remove low-expressed genes, which could also improve the calibration of p-values for low-expressed genes"
