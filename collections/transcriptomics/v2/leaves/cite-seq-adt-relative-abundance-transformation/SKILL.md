---
name: cite-seq-adt-relative-abundance-transformation
description: 'Use when preparing paired transcriptome–surface protein (CITE-seq or
  REAP-seq) data as training targets for cTP-net or comparable protein-imputation
  models: raw ADT barcode counts must be converted to per-cell relative abundances
  before model training.'
when_to_use_negative:
- Input scRNA-seq expression is not UMI-based count data (e.g., TPM/RPKM) — cTP-net
  can only apply to count-based (UMI) expression input, so the paired transformation
  pipeline is not valid
- The ADT matrix has already been relative-abundance/log-ratio transformed (double
  transformation would distort the response vectors)
- No paired protein measurements or trained cTP-net model exist, and the goal is de
  novo protein abundance estimation rather than transformation of measured ADT counts
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: cTP-net
  role: Implements the transformation within its preprocessing functions (preprocess_seurat2,
    preprocess_seurat3, preprocess_matrix) and inverts it in postprocessing after
    imputation
  repo: https://github.com/zhouzilu/cTPnet
- name: Seurat
  role: Performs the LogNormalize transformation of SAVER-X denoised RNA counts that
    feeds the same training procedure alongside the transformed ADT response vectors
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:cdc34cd016c0621cc3416d70cd312e3cdeb8e25fd1410d6f2198c3f18f905764
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - Adam (optimizer)
  - Python
  - R
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cite-seq-adt-relative-abundance-transformation

## Summary

Applies the Stoeckius et al. relative abundance transformation (geometric mean log-ratio per cell) to CITE-seq/REAP-seq surface protein ADT counts, producing the training response vectors for cTP-net. The article reports this transformed relative abundance gives better prediction accuracy than training on raw protein barcode counts.

## When to use

Use when preparing paired transcriptome–surface protein (CITE-seq or REAP-seq) data as training targets for cTP-net or comparable protein-imputation models: raw ADT barcode counts must be converted to per-cell relative abundances before model training.

## When NOT to use

- Input scRNA-seq expression is not UMI-based count data (e.g., TPM/RPKM) — cTP-net can only apply to count-based (UMI) expression input, so the paired transformation pipeline is not valid
- The ADT matrix has already been relative-abundance/log-ratio transformed (double transformation would distort the response vectors)
- No paired protein measurements or trained cTP-net model exist, and the goal is de novo protein abundance estimation rather than transformation of measured ADT counts

## Inputs

- Raw surface protein ADT barcode count matrix (cells x antibodies) from CITE-seq or REAP-seq experiments
- Paired scRNA-seq UMI count matrix (optionally SAVER-X denoised and Seurat LogNormalized) aligned cell-for-cell with the ADT matrix
- Vector/list of target surface protein names (e.g., dprotein argument in cTPnet preprocessing functions)

## Outputs

- Per-cell relative abundance matrix (geometric mean log-ratio transformed ADT values) used as cTP-net training response vectors
- Post-processed predicted protein abundances after inverting the transformation (postprocess_seurat2/seurat3/matrix)

## How to apply

Take the surface protein ADT count matrix (cells x antibodies) from CITE-seq/REAP-seq data (e.g., CITE-PBMC, CITE-CBMC, REAP-PBMC in Supplementary Table 1) and apply the relative abundance (geometric mean log-ratio) transformation from Stoeckius et al. so that each cell's protein profile is expressed relative to its geometric mean, forming the training response vectors. On the RNA side, first denoise scRNA-seq counts with SAVER-X (default parameters) and apply the Seurat default LogNormalize transformation; the normalized denoised matrix X becomes the training input while the transformed ADT values become the response. In the cTPnet package this step is handled by the preprocessing functions (preprocess_seurat2, preprocess_seurat3, preprocess_matrix) with postprocess_* functions inverting it after prediction. The rationale, per the paper, is that the network trained on transformed relative protein abundance as the response vector yields better prediction accuracy than training on raw protein barcode counts.

## Related tools

- **cTP-net** (Implements the transformation within its preprocessing functions (preprocess_seurat2, preprocess_seurat3, preprocess_matrix) and inverts it in postprocessing after imputation) — https://github.com/zhouzilu/cTPnet
- **Seurat** (Performs the LogNormalize transformation of SAVER-X denoised RNA counts that feeds the same training procedure alongside the transformed ADT response vectors)

## Examples

```
preprocess_seurat2(seurat_data, dprotein)
```

## Evaluation signals

- Per-cell geometric mean of the transformed protein values is approximately zero (log-ratio property), i.e., each cell's profile is centered relative to its own abundance scale
- Downstream cTP-net model trained on the transformed response yields higher cross-cell Pearson correlations with measured proteins than a model trained on raw protein barcode counts (reported in the paper)
- Imputation correlations from the transformed-response model clearly exceed the SAVER-X denoised RNA baseline (SAVER-X alone accounts for only ~0.02 correlation gain)
- The postprocess_seurat2/seurat3/matrix function recovers predicted values on the original abundance scale, and dimensions match the input ADT matrix (cells x proteins)

## Limitations

- cTP-net (and thus this transformation pipeline) is only applicable to count-based, UMI-derived expression input; accuracy on TPM/RPKM CITE-seq data is unknown because such data were not available for testing
- Comparisons are limited to proteins measured in the training panel (24 immunophenotype markers); a model extending prediction to 100 surface proteins is still in training
- Generalization to cell types outside the hematopoietic lineage was not tested

## Evidence

- [discussion] Relative abundance transformation is adopted from Stoeckius et al.: "For the surface protein counts, we adopted the relative abundance transformation from Stoeckius et al."
- [discussion] Transformed response outperforms raw barcode counts: "The network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than the network trained using raw protein barcode counts."
- [methods] Preprocessing functions in cTPnet package implement the pipeline: "- `preprocess_seurat2(seurat_data, dprotein)`"
- [discussion] Denoising precedes the transformation pipeline: "scRNA data denoising was performed with SAVER-X using default parameters."
- [discussion] cTP-net restricted to UMI count input: "cTP-net can only apply to count-based expression input (UMI-based)."
