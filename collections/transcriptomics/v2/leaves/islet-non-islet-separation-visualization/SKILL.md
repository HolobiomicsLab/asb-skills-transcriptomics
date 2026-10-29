---
name: islet-non-islet-separation-visualization
description: Use when after applying BUSseq to multi-batch scRNA-seq data (e.g., the
  four human pancreas datasets), you need a heat-map view of corrected counts over
  intrinsic genes to confirm that islet and non-islet cell populations separate cleanly
  rather than clustering by batch.
when_to_use_negative:
- Data have not been batch-corrected by BUSseq — plotting raw counts would confound
  islet/non-islet separation with batch effects
- The experimental design is completely confounded (e.g., batch 1 measures only cell
  types 1-2, batch 2 only types 3-4), where no batch-correction method is applicable
- The goal is a quantitative clustering metric (ARI, silhouette) rather than a visual
  separation check — compute those separately
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BUSseq
  role: Bayesian hierarchical model that corrects batch effects, clusters cell types,
    imputes dropouts, and identifies intrinsic genes whose corrected counts feed the
    heat map
  repo: https://github.com/songfd2018/BUSseq
- name: heatmap_data_BUSseq
  role: R function in the BUSseq package that draws the heat map of the intrinsic
    gene set on corrected/imputed values with a user-specified color key and image
    dimensions
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: BUSseq_MCMC
  role: Multi-core parallel MCMC fit of BUSseq (with BIC-based optimal K selection)
    that produces the BUSseqfits object used by the visualization
  repo: https://github.com/songfd2018/BUSseq-Rpackage
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:7885c80ad9d95d0a9d65aec6374b96dbfda0ef0f5daa2216f78952d95f2ad89f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - BUSseq_MCMC
  - LIGER
  - MNN
  - Scanorama
  - Seurat
  - ZINB-WaVE
  - heatmap_data_BUSseq
  - scVI
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# islet-non-islet-separation-visualization

## Summary

Visualize the separation of islet versus non-islet cells on BUSseq-corrected pancreas scRNA-seq counts using heatmap_data_BUSseq with the intrinsic gene set, serving as a qualitative check that batch correction preserved biological cell-type structure.

## When to use

Use when, after applying BUSseq to multi-batch scRNA-seq data (e.g., the four human pancreas datasets), you need a heat-map view of corrected counts over intrinsic genes to confirm that islet and non-islet cell populations separate cleanly rather than clustering by batch.

## When NOT to use

- Data have not been batch-corrected by BUSseq — plotting raw counts would confound islet/non-islet separation with batch effects
- The experimental design is completely confounded (e.g., batch 1 measures only cell types 1-2, batch 2 only types 3-4), where no batch-correction method is applicable
- The goal is a quantitative clustering metric (ARI, silhouette) rather than a visual separation check — compute those separately

## Inputs

- BUSseqfits object (sce_BUSseqfit) from BUSseq_MCMC on the four human pancreas scRNA-seq count matrices (GSE81076, GSE85241, GSE86473, E-MATB-5061), preprocessed to 7095 cells and 2480 highly variable genes
- Known cell-type labels (islet vs non-islet) per cell
- Intrinsic gene set from intrinsic_genes_BUSseq(sce_BUSseqfit)
- Corrected (and/or imputed) read counts from corrected_read_counts(sce_BUSseqfit)

## Outputs

- Heat map image of intrinsic-gene expression on BUSseq corrected values, with islet and non-islet cell separation visible (saved to the specified image_dir)
- Qualitative evidence supporting the ARI/silhouette-based islet vs non-islet separation quality assessment

## How to apply

First fit BUSseq with BUSseq_MCMC on the preprocessed read-count matrix (e.g., 7095 cells x 2480 highly variable genes from GSE81076, GSE85241, GSE86473, E-MATB-5061), selecting the number of cell types K by comparing BIC values across candidate K (the vignettes enumerate K from 3 to 6). Retrieve the fitted object and extract the intrinsic gene set via intrinsic_genes_BUSseq, which identifies genes that differentiate cell types, and the corrected values via corrected_read_counts (or imputed_read_counts). Call heatmap_data_BUSseq on the BUSseqfits object, passing the data_type, the intrinsic gene set, a project_name, an image_dir, a color_key_seq, and image_width/image_height, so the heat map shows islet versus non-islet separation on the corrected counts. Rationale: BUSseq simultaneously corrects batch effects, clusters cell types, imputes dropout-caused missing data, and identifies intrinsic genes, and after correction the values can be used downstream as if all cells were sequenced in a single batch. Judge correctness by whether the heat map's intrinsic-gene expression patterns group cells by cell type (islet vs non-islet) rather than by source batch, complementing the quantitative ARI and median silhouette coefficient benchmark.

## Related tools

- **BUSseq** (Bayesian hierarchical model that corrects batch effects, clusters cell types, imputes dropouts, and identifies intrinsic genes whose corrected counts feed the heat map) — https://github.com/songfd2018/BUSseq
- **heatmap_data_BUSseq** (R function in the BUSseq package that draws the heat map of the intrinsic gene set on corrected/imputed values with a user-specified color key and image dimensions) — https://github.com/songfd2018/BUSseq-Rpackage
- **BUSseq_MCMC** (Multi-core parallel MCMC fit of BUSseq (with BIC-based optimal K selection) that produces the BUSseqfits object used by the visualization) — https://github.com/songfd2018/BUSseq-Rpackage

## Examples

```
R: gene_set <- intrinsic_genes_BUSseq(sce_BUSseqfit); heatmap_data_BUSseq(sce_BUSseqfit, data_type="Corrected", gene_set=gene_set, project_name="pancreas", image_dir="figs/", color_key_seq=seq(-2,2,0.5), image_width=900, image_height=900)
```

## Evaluation signals

- The heat map groups cells by known islet vs non-islet labels rather than by source batch (GSE81076, GSE85241, GSE86473, E-MATB-5061)
- The gene set plotted is the BUSseq-identified intrinsic gene set (intrinsic_genes_BUSseq), not an arbitrary gene list
- The plotted values are BUSseq corrected_read_counts (or imputed_read_counts), consistent with downstream analysis on a single pseudo-batch
- The visual separation supports BUSseq's top benchmark standing: highest ARI (0.608) among LIGER (0.542), MNN (0.279), Scanorama (0.527), scVI (0.282), Seurat (0.287), and ZINB-WaVE (0.380)
- Optimal K used for the fit was chosen by minimum BIC over the enumerated candidate K values (e.g., K = 3 to 6 in the vignettes)

## Limitations

- MCMC inference is computationally heavy; most steps are parallelizable but runtime scales with K, iterations, and cell/gene counts (the small demo alone takes ~12 min for 4 K values)
- Multi-stage pipelines that ignore clustering uncertainty can introduce false positives and false negatives in downstream gene-level conclusions
- Requires the experimental design to satisfy identifiability conditions (e.g., reference-panel or chain-type designs); completely confounded designs admit no applicable correction method

## Evidence

- [methods] heatmap_data_BUSseq signature: "heatmap_data_BUSseq(sce_BUSseqfit, data_type, gene_set, project_name, image_dir, color_key_seq, image_width, image_height)"
- [discussion] intrinsic gene identification: "BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes that differentiate cell types."
- [readme] corrected values for downstream analysis: "After correcting the batch effects with BUSseq, the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch."
- [methods] optimal K selection via BIC: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [methods] post-correction downstream usability: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
