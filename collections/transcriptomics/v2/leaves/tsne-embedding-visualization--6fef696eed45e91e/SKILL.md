---
name: tsne-embedding-visualization
description: Use when you have BUSseq-fitted corrected read counts (from corrected_read_counts)
  and need to validate that the batch-effect correction succeeded before downstream
  analysis.
when_to_use_negative:
- The data are raw, uncorrected read counts still containing batch effects — the article
  draws t-SNE on uncorrected data only to illustrate the problem, not as validation
- The cell type labels have not been inferred (BUSseq MCMC posterior inference not
  yet run), since coloring by cell type requires celltypes(sce_BUSseqfit)
- The analysis goal is quantitative clustering or differential expression rather than
  qualitative visual assessment of batch-effect removal
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: BUSseq
  role: Provides the corrected read count matrix (corrected_read_counts) and inferred
    cell type labels (celltypes) that are embedded and visualized by t-SNE
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: BUSseq-1.0
  role: Demo workflow generates t-SNE plots of count data colored by batch labels
    and cell type labels in R, including for uncorrected synthetic data
  repo: https://github.com/songfd2018/BUSseq-1.0
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:82dce0a9b4f143a8e43e33b1a6932ea8ed3c85cb0346b476d52202fa9d03f290
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq-Rpackage
  tools:
  - BUSseq
  - BUSseq-1.0
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# tsne-embedding-visualization

## Summary

Run t-SNE on the batch-effect-corrected read count matrix produced by BUSseq's quantile-matching procedure, coloring cells by both inferred cell type and batch label, to visually confirm that corrected values group by cell type with no batch separation.

## When to use

Use when you have BUSseq-fitted corrected read counts (from corrected_read_counts) and need to validate that the batch-effect correction succeeded before downstream analysis.

## When NOT to use

- The data are raw, uncorrected read counts still containing batch effects — the article draws t-SNE on uncorrected data only to illustrate the problem, not as validation
- The cell type labels have not been inferred (BUSseq MCMC posterior inference not yet run), since coloring by cell type requires celltypes(sce_BUSseqfit)
- The analysis goal is quantitative clustering or differential expression rather than qualitative visual assessment of batch-effect removal

## Inputs

- BUSseq fitted model object (e.g. sce_BUSseqfit / BUSseqfits_example)
- Corrected read count matrix from corrected_read_counts(sce_BUSseqfit)
- Inferred cell type labels from celltypes(sce_BUSseqfit)
- Batch labels for each cell

## Outputs

- t-SNE embedding of the corrected count matrix colored by cell type (e.g. tsne_corrected_counts.png)
- t-SNE embedding colored by batch label
- Visual confirmation that cells group by cell type with no separation by batch

## How to apply

Load the fitted BUSseq model object (e.g. sce_BUSseqfit / BUSseqfits_example) and compute the corrected count matrix with corrected_read_counts(sce_BUSseqfit). Retrieve the inferred cell labels via celltypes(sce_BUSseqfit). Run t-SNE on the corrected count matrix and generate two scatter-colored versions of the embedding: one colored by cell type and one by batch. The rationale is that BUSseq's quantile-matching yields adjusted counts free of batch effects and dropout events, so cells should cluster by biological cell type rather than by sequencing batch. The result is saved as an image (e.g. tsne_corrected_counts.png) as visual validation evidence.

## Related tools

- **BUSseq** (Provides the corrected read count matrix (corrected_read_counts) and inferred cell type labels (celltypes) that are embedded and visualized by t-SNE) — https://github.com/songfd2018/BUSseq-Rpackage
- **BUSseq-1.0** (Demo workflow generates t-SNE plots of count data colored by batch labels and cell type labels in R, including for uncorrected synthetic data) — https://github.com/songfd2018/BUSseq-1.0

## Examples

```
R> corrected <- corrected_read_counts(sce_BUSseqfit); cell_types <- celltypes(sce_BUSseqfit); tsne <- Rtsne(t(corrected)); plot(tsne$Y, col = cell_types); # save as tsne_corrected_counts.png
```

## Evaluation signals

- In the t-SNE embedding colored by cell type, cells of the same inferred cell type form coherent clusters
- In the t-SNE embedding colored by batch, cells from different batches are intermixed with no separation by batch
- The embedding is computed on corrected_read_counts output, not the raw count matrix
- The corrected counts require no further cell-specific normalization before embedding, as expected from the quantile-matching procedure
- The plot image (e.g. tsne_corrected_counts.png) is produced and stored alongside the corrected count matrix (corrected_read_counts.csv)

## Limitations

- t-SNE is a qualitative visual check; the article notes BUSseq itself is an MCMC-based method whose steps are parallelizable but computationally heavy
- Completely confounded designs (e.g. batch 1 measures cell types 1 and 2, batch 2 measures cell types 3 and 4) admit no method, so even a correct embedding cannot recover cell-type structure in that setting
- In the demo, t-SNE on uncorrected data colored by batch and cell type is shown to illustrate the raw batch effect — do not confuse that with validation of the correction

## Evidence

- [methods] the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
- [readme] README: t-SNE plot generation: [evidence span withheld — anchor status: external_doc]
- [discussion] discussion: BUSseq capabilities: "BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes that differentiate cell types."
- [methods] methods: downstream analysis step: [evidence span withheld — anchor status: not_found]
