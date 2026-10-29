---
name: bayesian-hierarchical-model-posterior-use
description: Use when you have a fitted BUSseq model object (e.g., sce_BUSseqfit from
  BUSseq_MCMC) and need corrected count data free of batch effects and dropout for
  downstream analysis, cell-type-colored t-SNE, or DE testing.
when_to_use_negative:
- The experimental design is completely confounded (e.g., batch 1 measures cell types
  1 and 2, batch 2 measures cell types 3 and 4) — no batch-correction method is applicable
- The data are bulk experiments with known subtype information, where classic methods
  such as ComBat or SVA are the appropriate choice
- Only imputed (dropout-corrected) values are needed rather than batch-corrected values
  — use imputed_read_counts(sce_BUSseqfit) instead of corrected_read_counts
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BUSseq
  role: Fits the Bayesian hierarchical model and provides corrected_read_counts(sce_BUSseqfit)
    to generate the batch-effect-corrected counts via quantile matching
  repo: https://github.com/songfd2018/BUSseq
- name: BUSseq-Rpackage
  role: R package providing the corrected_read_counts, imputed_read_counts, celltypes,
    and intrinsic_genes_BUSseq accessors used in the quantile-matching workflow
  repo: https://github.com/songfd2018/BUSseq-Rpackage
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:f8c8abf72d0ff6ecaca6e212f952cb78f6598d55f9323204e12c40c984f6f490
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - BUSseq-Rpackage
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# bayesian-hierarchical-model-posterior-use

## Summary

Reconstructs BUSseq's quantile-matching module, which uses the fitted Bayesian hierarchical model posterior (cell-type means, batch location effects, cell-specific size factors, overdispersion, and dropout terms) to output batch-effect-corrected read counts usable as if all cells were sequenced in a single batch.

## When to use

Use when you have a fitted BUSseq model object (e.g., sce_BUSseqfit from BUSseq_MCMC) and need corrected count data free of batch effects and dropout for downstream analysis, cell-type-colored t-SNE, or DE testing.

## When NOT to use

- The experimental design is completely confounded (e.g., batch 1 measures cell types 1 and 2, batch 2 measures cell types 3 and 4) — no batch-correction method is applicable
- The data are bulk experiments with known subtype information, where classic methods such as ComBat or SVA are the appropriate choice
- Only imputed (dropout-corrected) values are needed rather than batch-corrected values — use imputed_read_counts(sce_BUSseqfit) instead of corrected_read_counts

## Inputs

- Fitted BUSseq model object (e.g., BUSseqfits_example / sce_BUSseqfit) from BUSseq_MCMC posterior inference
- Raw G-by-N scRNA-seq read count matrix (e.g., count_data_[study]_v[version].txt) with dim_[study]_v[version].txt dimensions file
- Metadata file (metadata_[study]_v[version].txt) with batch and cell type labels

## Outputs

- Corrected read count matrix (e.g., corrected_read_counts.csv) usable for single-batch-style downstream analysis
- t-SNE embedding of corrected counts colored by cell type and batch (e.g., tsne_corrected_counts.png)

## How to apply

Load the fitted BUSseq model object (e.g., BUSseqfits_example) together with the raw read-count matrices in R. For each gene-cell pair, compute the negative-binomial CDF values of the observed count under the full fitted model — incorporating fitted cell-type means, batch location effects, cell-specific sequencing depth, overdispersion, and dropout terms. Draw u uniformly between those CDF values and map u to the quantile of the batch-effect-free negative-binomial distribution whose mean depends only on cell type, obtaining the corrected count via corrected_read_counts(); write the matrix (e.g., corrected_read_counts.csv). Retrieve inferred cell labels with celltypes(sce_BUSseqfit) and run t-SNE on the corrected matrix, coloring cells by cell type and by batch, to validate that cells group by type with no separation by batch. The rationale is that the corrected values can then be used for downstream analysis as if all cells were sequenced in a single batch, with no further cell-specific normalization.

## Related tools

- **BUSseq** (Fits the Bayesian hierarchical model and provides corrected_read_counts(sce_BUSseqfit) to generate the batch-effect-corrected counts via quantile matching) — https://github.com/songfd2018/BUSseq
- **BUSseq-Rpackage** (R package providing the corrected_read_counts, imputed_read_counts, celltypes, and intrinsic_genes_BUSseq accessors used in the quantile-matching workflow) — https://github.com/songfd2018/BUSseq-Rpackage

## Examples

```
corrected_counts <- corrected_read_counts(sce_BUSseqfit); celltypes(sce_BUSseqfit)
```

## Evaluation signals

- In the t-SNE embedding of the corrected count matrix, cells group by cell type and show no separation by batch, consistent with values usable as if measured in a single batch
- Corrected counts require no further cell-specific normalization before downstream analysis
- The number of cell types K used for posterior inference is selected by minimum BIC when the true K is unknown (e.g., enumerate K from 3 to 6)
- MCMC sampling is reproducible given the same seed (-s flag) and the same number of iterations and burn-ins
- Output matrix dimensions match the input G-by-N read count matrix

## Limitations

- MCMC algorithms are well-known for heavy computation load, although most steps of the BUSseq MCMC are parallelizable (Supplementary Note 12; a GPU CUDA version exists in Anguscgm/BUSseq_gpu)
- Completely confounded designs (different cell types measured in disjoint batches) admit no applicable correction method
- As the clustering results may be prone to errors in practice, errors can propagate and lead to false positives and false negatives in downstream steps
- The C++ source codes support Linux operating systems (tested on Ubuntu 18.04 and CentOS 7); R scripts are tested on R 3.6.0

## Evidence

- [readme] After correcting the batch effects with BUSseq, the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch.: "After correcting the batch effects with BUSseq, the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch."
- [readme] BUSseq is able to simultaneously correct batch effects, clusters cell types, and takes care of the count data nature, the overdispersion, the dropout events, and the cell-specific sequencing depth of scRNA-seq data.: "BUSseq is able to simultaneously correct batch effects, clusters cell types, and takes care of the count data nature, the overdispersion, the dropout events, and the cell-specific sequencing depth of"
- [methods] the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
- [methods] corrected_read_counts accessor from the R package vignette filter steps: "- `corrected_read_counts(sce_BUSseqfit)`"
- [methods] Optimal cell-type number selection criterion for the posterior fit: "To select the optimal *K*, we need to compare the BIC values for different *K*s."
