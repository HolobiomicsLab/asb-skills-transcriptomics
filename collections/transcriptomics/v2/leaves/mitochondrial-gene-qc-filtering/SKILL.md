---
name: mitochondrial-gene-qc-filtering
description: Use when preprocessing a SpatialExperiment gene-by-spot count matrix
  (e.g. Visium human DLPFC) before fitting nnSVG, where mitochondrial genes and genes
  with low UMI counts or detection in too few spots must be dropped to reduce runtime
  and improve p-value calibration.
when_to_use_negative:
- If the goal is spatial-domain marker genes (mean-level changes between domains)
  rather than continuous SVGs, this filtering context from the nnSVG SVG-detection
  workflow is not the right framework
- If counts are already normalized logcounts without a raw counts assay — filter_genes
  operates on the raw count matrix with UMI-count and percent-of-spots thresholds
- If downstream analysis requires mitochondrial genes retained (e.g. QC metric computation
  itself), since filter_mito removes them outright
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: nnSVG
  role: Provides the filter_genes module with filter_genes_ncounts, filter_genes_pcspots,
    and filter_mito parameters for mitochondrial and low-expression gene removal prior
    to SVG detection
  repo: https://bioconductor.org/packages/nnSVG
- name: Bioconductor
  role: Framework from which the nnSVG R package (and its SpatialExperiment dependency)
    is installed via BiocManager
  repo: https://bioconductor.org/packages/nnSVG
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:acf584bc60ed0114b61c2a075324e0a29053eb25cc63dd953f90ecde3d993702
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - nnSVG
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# mitochondrial-gene-qc-filtering

## Summary

Removes mitochondrial genes (and optionally low-expressed genes) from a spatially-resolved transcriptomics count matrix using the nnSVG Bioconductor package's filter_genes module before SVG detection. On the Visium human DLPFC tutorial dataset with default parameters it removed 13 mitochondrial genes and 30216 of 33525 low-expression genes, retaining 3396 of 21803 genes in the paper's reported run.

## When to use

Use when preprocessing a SpatialExperiment gene-by-spot count matrix (e.g. Visium human DLPFC) before fitting nnSVG, where mitochondrial genes and genes with low UMI counts or detection in too few spots must be dropped to reduce runtime and improve p-value calibration.

## When NOT to use

- If the goal is spatial-domain marker genes (mean-level changes between domains) rather than continuous SVGs, this filtering context from the nnSVG SVG-detection workflow is not the right framework
- If counts are already normalized logcounts without a raw counts assay — filter_genes operates on the raw count matrix with UMI-count and percent-of-spots thresholds
- If downstream analysis requires mitochondrial genes retained (e.g. QC metric computation itself), since filter_mito removes them outright

## Inputs

- SpatialExperiment Bioconductor object with a gene-by-spot raw counts assay (e.g. Visium_humanDLPFC() from STexampleData, 33538 genes x 4992 spots before QC)
- Spot-level QC already performed and spots outside tissue removed (in_tissue == 1)

## Outputs

- Filtered SpatialExperiment with mitochondrial genes and low-expressed genes removed (e.g. 33525 -> 3309 genes on the tutorial dataset after all filters)
- Filtering log messages reporting counts removed (e.g. 'removed 13 mitochondrial genes', 'removed 30216 out of 33525 genes due to low expression')
- Gene count table / verification counts (e.g. filtered_gene_counts.csv: 21803 total, 3396 passing)

## How to apply

Apply filter_genes() from the nnSVG package to a SpatialExperiment object after spot-level QC and keeping spots over tissue (in_tissue == 1). The module exposes three parameters — filter_genes_ncounts, filter_genes_pcspots, and filter_mito — controlling per-gene UMI-count thresholds, percent-of-spots detection thresholds, and mitochondrial gene removal; defaults require at least 3 counts in at least 0.5% of spatial locations (n = 19 spots on the Visium human DLPFC sample) and remove mitochondrial genes. Run it before computing logcounts with scran (computeLibraryFactors + logNormCounts), since downstream nnSVG model fitting assumes filtered, normalized logcounts. More stringent filtering reduces runtime because nnSVG fitting scales with the number of genes, and the paper notes that adaptive filtering thresholds could further improve p-value calibration for low-expressed genes. Verify the retained gene count against expected values (e.g. 3396 of 21803 genes on the Visium human DLPFC dataset as reported in the paper's workflow).

## Related tools

- **nnSVG** (Provides the filter_genes module with filter_genes_ncounts, filter_genes_pcspots, and filter_mito parameters for mitochondrial and low-expression gene removal prior to SVG detection) — https://bioconductor.org/packages/nnSVG
- **Bioconductor** (Framework from which the nnSVG R package (and its SpatialExperiment dependency) is installed via BiocManager) — https://bioconductor.org/packages/nnSVG

## Examples

```
spe <- filter_genes(spe)  # nnSVG defaults: filter_genes_ncounts, filter_genes_pcspots, filter_mito
```

## Evaluation signals

- Console output of filter_genes must report mitochondrial gene removal first (e.g. 'removed 13 mitochondrial genes' on the tutorial dataset)
- Retention invariant: 3396 of 21803 genes survive the paper's low-expression filtering on Visium human DLPFC; 33525 -> 3309 genes on the README tutorial run
- Default threshold check: genes retained have at least 3 counts in at least 0.5% (n = 19) of spatial locations
- Ordering check: filtering precedes computeLibraryFactors/logNormCounts and any nnSVG(spe) model fitting
- Filtering log should match the pattern 'removed 30216 out of 33525 genes due to low expression' when defaults are used

## Limitations

- Filtering thresholds are fixed defaults from the package documentation, not adaptive; the paper notes adaptive filtering thresholds that carefully remove low-expressed genes (and improve p-value calibration) are not yet implemented
- More aggressive filtering reduces runtime (which depends on the number of genes) but trades off against p-value calibration for low-expressed genes
- Parameter choices are deferred to the nnSVG package documentation; QC metric thresholds are only available in the external code repository rather than the manuscript text (reproducibility gap)

## Evidence

- [readme] Default filtering parameters remove mitochondrial genes and low-expressed genes: "# filter low-expressed and mitochondrial genes
# using function from nnSVG package with default filtering parameters
spe <- filter_genes(spe)"
- [readme] Concrete removal counts and threshold on the tutorial dataset: [evidence span withheld — anchor status: external_doc]
- [discussion] Runtime depends on number of genes; more stringent filtering helps: "Since runtime depends on the number of genes, this can be reduced with more stringent gene filtering"
- [discussion] Adaptive thresholds are proposed future work for p-value calibration: "This could be improved by developing adaptive filtering thresholds that carefully remove low-expressed genes, which could also improve the calibration of p-values for low-expressed genes"
