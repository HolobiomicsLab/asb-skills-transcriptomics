---
name: within-cell-type-scatter-visualization
description: Use when benchmarking protein imputation methods (cTP-net vs Seurat v3
  anchor transfer) on CITE-seq PBMC data and you need per-cell-type visual comparison
  of predicted vs measured ADT levels, e.g. reproducing Fig. 2b-c.
when_to_use_negative:
- Expression data is transcriptome-only without paired ADT measurements — there are
  no measured protein levels to plot against
- Expression input is TPM/RPKM rather than UMI-based counts, which cTP-net cannot
  be applied to
- Cells are not of hematopoietic lineage — generalization of imputation was not tested
  outside immune cells
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: cTP-net
  role: Source of the imputed surface protein abundances plotted against CITE-seq
    measurements; available in R and Python
  repo: https://github.com/zhouzilu/cTPnet
- name: Seurat v3
  role: Baseline anchor-based transfer-learning method whose imputed protein abundances
    are co-plotted for comparison (Fig. 2b-c)
- name: R
  role: Language used to render the comparison scatter plots alongside the cTP-net
    vignette workflow
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:14eb7fc8610551892d06b61bd7a39f4407e31e43fcb515cf8b4b2609f29ac1a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - R
  - Seurat v3
  - cTP-net
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# within-cell-type-scatter-visualization

## Summary

Renders per-cell-type scatter plots of CITE-seq-measured versus imputed (cTP-net or Seurat v3) surface protein abundance to visually assess imputation fidelity within each PBMC cell type, reproducing Fig. 2b-c of the cTP-net paper.

## When to use

Use when benchmarking protein imputation methods (cTP-net vs Seurat v3 anchor transfer) on CITE-seq PBMC data and you need per-cell-type visual comparison of predicted vs measured ADT levels, e.g. reproducing Fig. 2b-c.

## When NOT to use

- Expression data is transcriptome-only without paired ADT measurements — there are no measured protein levels to plot against
- Expression input is TPM/RPKM rather than UMI-based counts, which cTP-net cannot be applied to
- Cells are not of hematopoietic lineage — generalization of imputation was not tested outside immune cells

## Inputs

- CITE-seq-measured ADT (surface protein) abundance matrix for the holdout validation cells
- cTP-net imputed protein abundance matrix for the same validation cells
- Seurat v3 anchor-transfer imputed protein abundance matrix for the same cells
- Per-cell cell-type annotations from the CITE-PBMC paired RNA/ADT data

## Outputs

- Within-cell-type scatter plots of measured vs imputed protein abundance, one per protein and per method (Fig. 2b-c equivalents)
- Per-cell-type Pearson correlation values between measured and imputed protein levels for cTP-net and Seurat v3

## How to apply

For each surface protein target (e.g. CD4, CD8, CD127 among the 24 immunophenotype markers), subset cells by assigned cell type and scatter the CITE-seq-measured ADT abundance (x-axis) against the imputed protein abundance (y-axis) on the shared validation cells. Produce one panel per imputation method — cTP-net and the Seurat v3 anchor-based transfer predictions — on identical axes so per-cell-type correlation and dynamic-range compression are directly comparable. Restrict plotted cells to the same holdout validation set used for the cTP-net correlation heatmap to account for variations between subsets, especially in the out-of-cell-type scenario where a whole cell type is excluded from training. Judge correctness by whether the visualized within-cell-type Pearson correlation for cTP-net matches or exceeds Seurat v3 in the holdout scenario, and whether cTP-net correlations are visibly much stronger than Seurat v3 for the held-out cell type.

## Related tools

- **cTP-net** (Source of the imputed surface protein abundances plotted against CITE-seq measurements; available in R and Python) — https://github.com/zhouzilu/cTPnet
- **Seurat v3** (Baseline anchor-based transfer-learning method whose imputed protein abundances are co-plotted for comparison (Fig. 2b-c))
- **R** (Language used to render the comparison scatter plots alongside the cTP-net vignette workflow) — https://github.com/zhouzilu/cTPnet

## Examples

```
Rscript cTPnet_vignette_v3.Rmd  # after imputation, plot measured vs imputed ADT per cell type with ggplot2: ggplot(sub, aes(measured, imputed)) + geom_point() + facet_wrap(~protein)
```

## Evaluation signals

- Plotted cells must be exactly the same cells used in the cTP-net validation set, to account for variations between subsets
- Within-cell-type Pearson correlation of cTP-net predictions vs measured ADT should be slightly higher than Seurat v3 in the random holdout scenario
- In the out-of-cell-type scatter (held-out cell type), cTP-net correlations should vastly exceed Seurat v3 correlations, matching Fig. 2a heatmap entries
- Axes and cell subsets must be identical between the cTP-net and Seurat v3 panels for a fair visual comparison
- Each scatter should show measurable positive correlation for proteins like CD markers where transcript and protein levels are related

## Limitations

- Comparisons are limited to the same cells as the validation set in the holdout scheme due to subset variation
- Protein targets restricted to the ~24 immunophenotype markers of the trained cTP-net model; a 100-protein model is still in training
- cTP-net can only apply to count-based (UMI) expression input; TPM/RPKM accuracy is unknown

## Evidence

- [discussion] we limit comparisons to the same cells that were in the validation set in the holdout scheme to account for variations between subsets.: "we limit comparisons to the same cells that were in the validation set in the holdout scheme to account for variations between subsets."
- [discussion] We compared cTP-net with an anchor-based transfer-learning method developed in Seurat v3: "We compared cTP-net with an anchor-based transfer-learning method developed in Seurat v3"
- [readme] For three example proteins, cross-cell scatter and correlation of CITE-seq measured abundances vs. (1) raw RNA count, (2) SAVER-X denoised RNA level, and (3) cTP-net predicted protein abundance.: "For three example proteins, cross-cell scatter and correlation of CITE-seq measured abundances vs. (1) raw RNA count, (2) SAVER-X denoised RNA level, and (3) cTP-net predicted protein abundance."
- [discussion] instead of selecting the test set cells randomly, we partition the cells by their cell types: "instead of selecting the test set cells randomly, we partition the cells by their cell types"
