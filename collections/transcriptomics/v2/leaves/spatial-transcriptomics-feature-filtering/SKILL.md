---
name: spatial-transcriptomics-feature-filtering
description: Use when constructing a SpatialDDLS object from paired scRNA-seq (SingleCellExperiment)
  and spatial transcriptomics (SpatialExperiment) data and you must define the feature
  set via logFC-based per-cluster filtering before simulating mixed transcriptional
  profiles.
when_to_use_negative:
- The inputs are already a reduced/pre-filtered feature table or a trained SpatialDDLS
  model is being applied to new spatial data — in that case use the existing gene
  set rather than re-filtering
- There is no paired scRNA-seq reference from which per-cluster logFC statistics can
  be computed
- The goal is deconvolution evaluation metrics (PCC/CCC) rather than defining the
  feature space; those come later via trainDeconvModel and calculateEvalMetrics
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SpatialDDLS
  role: Provides createSpatialDDLSobject and the logFC-based gene filtering parameters
    (sc.filt.genes.cluster, sc.n.genes.per.cluster, sc.min.mean.counts) that implement
    this skill
  repo: https://github.com/diegommcc/SpatialDDLS
- name: SpatialDDLSdata
  role: Supplies the MouseDLN.SCE and MouseDLN.ST example objects on which the filtering
    thresholds are demonstrated
  repo: https://github.com/diegommcc/SpatialDDLSdata
- name: SingleCellExperiment
  role: Container class for the scRNA-seq reference whose genes are filtered per cluster
- name: SpatialExperiment
  role: Container class for the spatial transcriptomics data used to determine shared
    genes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:509097842c26fe668a9cfefc1d750877744680ed4d093e31d8a93d591cfdcc0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/diegommcc/SpatialDDLS
  tools:
  - R
  - SingleCellExperiment
  - SpatialDDLS
  - SpatialDDLSdata
  - SpatialExperiment
  - ggplot2
  - ggpubr
  - keras
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# spatial-transcriptomics-feature-filtering

## Summary

Gene/feature selection step of the SpatialDDLS workflow: when building a SpatialDDLS object from paired scRNA-seq and 10x Visium spatial data, genes are filtered per cluster by logFC and mean counts, mitochondrial and zero-expression genes are removed, and the shared gene set between the single-cell and spatial assays is fixed for downstream profile simulation and NN training.

## When to use

Use when constructing a SpatialDDLS object from paired scRNA-seq (SingleCellExperiment) and spatial transcriptomics (SpatialExperiment) data and you must define the feature set via logFC-based per-cluster filtering before simulating mixed transcriptional profiles.

## When NOT to use

- The inputs are already a reduced/pre-filtered feature table or a trained SpatialDDLS model is being applied to new spatial data — in that case use the existing gene set rather than re-filtering
- There is no paired scRNA-seq reference from which per-cluster logFC statistics can be computed
- The goal is deconvolution evaluation metrics (PCC/CCC) rather than defining the feature space; those come later via trainDeconvModel and calculateEvalMetrics

## Inputs

- SingleCellExperiment with scRNA-seq data (e.g. MouseDLN.SCE from the SpatialDDLSdata package)
- SpatialExperiment with spatial transcriptomics data (e.g. MouseDLN.ST, 10x Visium murine lymph node slides)
- Filtering parameters: sc.filt.genes.cluster, sc.n.genes.per.cluster, sc.min.mean.counts

## Outputs

- SpatialDDLS object with logFC-filtered, mitochondria- and zero-expression-gene-removed feature set (1041 genes in the vignette run)
- Shared gene set between single-cell and spatial assays (10941 genes pre-logFC-filtering)
- Console log counts for each filtering stage (logFC-filtered genes, removed mitochondrial genes, genes without expression, shared genes)

## How to apply

Call createSpatialDDLSobject with sc.filt.genes.cluster = TRUE, sc.n.genes.per.cluster = 150, and sc.min.mean.counts = 2; these two parameters — sc.n.genes.per.cluster and sc.min.mean.counts — control which genes are kept per reference cell cluster. The logFC-based filtering on the mouse lymph node data yields 1041 genes after removing 11 mitochondrial genes, and 16 genes without expression in any cell are also dropped. The pipeline then intersects genes across assays, reporting 10941 shared genes between the single-cell and spatial transcriptomics datasets in the vignette. The retained per-cluster informative genes (rather than all genes) are the features on which mixed profiles are later simulated and on which the fully connected NN is trained, so this filtering step determines the input dimensionality of the model. Verify each filtering step by the console log counts emitted by createSpatialDDLSobject rather than assuming the final gene number.

## Related tools

- **SpatialDDLS** (Provides createSpatialDDLSobject and the logFC-based gene filtering parameters (sc.filt.genes.cluster, sc.n.genes.per.cluster, sc.min.mean.counts) that implement this skill) — https://github.com/diegommcc/SpatialDDLS
- **SpatialDDLSdata** (Supplies the MouseDLN.SCE and MouseDLN.ST example objects on which the filtering thresholds are demonstrated) — https://github.com/diegommcc/SpatialDDLSdata
- **SingleCellExperiment** (Container class for the scRNA-seq reference whose genes are filtered per cluster)
- **SpatialExperiment** (Container class for the spatial transcriptomics data used to determine shared genes)

## Examples

```
mouseDLN.SDDLS <- createSpatialDDLSobject(sc.data = MouseDLN.SCE, st.data = MouseDLN.ST, sc.filt.genes.cluster = TRUE, sc.n.genes.per.cluster = 150, sc.min.mean.counts = 2)
```

## Evaluation signals

- createSpatialDDLSobject log should report 'Number of genes after filtering based on logFC: 1041' for the vignette parameterization
- Log should report 'Number of removed mitochondrial genes: 11' after mitochondrial gene removal
- Log should report removal of genes without expression in any cell (16 genes in the vignette run)
- Log should report 'Number of shared genes between single-cell and spatial transcriptomics datasets: 10941' before logFC filtering
- Downstream NN input dimensionality must equal the final filtered shared-gene count; if simulated profiles or trainDeconvModel fail on feature mismatch, the filtering output was not propagated correctly

## Limitations

- The final gene count is parameter-dependent: sc.n.genes.per.cluster (150) and sc.min.mean.counts (2) directly control retention, so results are not comparable across settings
- Mitochondrial genes (11 in the vignette) and genes without expression in any cell (16) are removed by design, so filtered counts will not match raw feature counts
- The shared-gene step depends on platform overlap (10x Visium spatial vs 10x Chromium single-cell); low overlap shrinks the feature set

## Evidence

- [methods] Filtering parameters: "specially `sc.n.genes.per.cluster` and `sc.min.mean.counts` parameters"
- [methods] LogFC filtering yield: "=== Number of genes after filtering based on logFC: 1041"
- [methods] Mitochondrial removal: "=== Number of removed mitochondrial genes: 11"
- [methods] Zero-expression genes: "- Removing 16 genes without expression in any cell"
- [methods] Shared gene intersection: "=== Number of shared genes between single-cell and spatial transcriptomics datasets: 10941"
- [readme] Pipeline context: "The package takes advantage of single-cell RNA sequencing (scRNA-seq) data to simulate mixed transcriptional profiles with known cell composition"
