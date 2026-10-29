---
name: citeseq-paired-multimodal-data-handling
description: Use when you hold paired UMI-based scRNA-seq and ADT (protein) count
  matrices from CITE-seq or REAP-seq and need normalized RNA features plus CLR-transformed
  protein targets before surface-protein imputation with cTP-net.
when_to_use_negative:
- Expression data are TPM/RPKM or otherwise not UMI-based counts — prediction accuracy
  is unknown for such inputs (CITE-seq with TPM/RPKM was not available for testing)
- Only the transcriptome is available with no paired protein panel — there are no
  ADT counts to transform, so this paired-multimodal preprocessing does not apply
- Data are already SAVER-X-denoised and LogNormalized with protein CLR abundances
  computed — rerunning would double-transform the features
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: SAVER-X
  role: Denoises the raw UMI scRNA-seq count matrix (default parameters) to produce
    the denoised expression estimates Λ used as cTP-net training input
- name: Seurat
  role: Provides the default LogNormalize function applied to the SAVER-X denoised
    counts (scale factor 10000)
- name: cTP-net
  role: Consumes the normalized denoised RNA matrix and the CLR-transformed protein
    abundances for surface-protein imputation; R/Python package provides preprocess_seurat2/3,
    preprocess_matrix and postprocess_* helpers
  repo: https://github.com/zhouzilu/cTPnet
- name: R
  role: Language for the cTPnet R package vignettes (Seurat v2/v3 notebooks) and preprocessing
    helpers
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:b2cb72e78a8848ccda3c34b8ca724f51f9da786366d57c74c7f0158664953f58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - R
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# citeseq-paired-multimodal-data-handling

## Summary

Preprocesses paired CITE-seq/REAP-seq data by denoising the UMI scRNA-seq count matrix with SAVER-X, applying Seurat LogNormalize (scale factor 10000), and converting surface-protein ADT counts into log-ratio relative abundances (per Stoeckius et al.) suitable as response vectors for cTP-net imputation.

## When to use

Use when you hold paired UMI-based scRNA-seq and ADT (protein) count matrices from CITE-seq or REAP-seq and need normalized RNA features plus CLR-transformed protein targets before surface-protein imputation with cTP-net.

## When NOT to use

- Expression data are TPM/RPKM or otherwise not UMI-based counts — prediction accuracy is unknown for such inputs (CITE-seq with TPM/RPKM was not available for testing)
- Only the transcriptome is available with no paired protein panel — there are no ADT counts to transform, so this paired-multimodal preprocessing does not apply
- Data are already SAVER-X-denoised and LogNormalized with protein CLR abundances computed — rerunning would double-transform the features

## Inputs

- Raw UMI-based scRNA-seq count matrix (required — cTP-net only applies to UMI-based count input)
- Paired surface-protein ADT count matrix (e.g. CITE-seq/REAP-seq antibody-derived tags)
- Demo/example data such as cTPnet_demo (4000 cells) with gene_names_protein12

## Outputs

- Denoised (SAVER-X) and LogNormalize-transformed gene expression matrix used as neural-network training input
- Log-ratio-transformed relative protein abundance matrix (CLR of ADT counts) used as the network response vectors
- Combined preprocessed object (e.g. preprocessed_input.rds) holding both matrices

## How to apply

Load the raw UMI-based scRNA-seq count matrix and the paired ADT count matrix (e.g. the cTPnet_demo dataset). Remove low-quality genes (<10 counts across cells) and low-quality cells (<200 genes detected), then denoise the gene matrix with SAVER-X using default parameters to obtain denoised estimates Λ; skipping this step retracts cTP-net correlation by 0.02, more than any other parameter tweak. Apply Seurat default LogNormalize to Λ, computing X_ij = log(Λ_ij * 10000 / m_j) where m_j is cell j's total count. Transform each cell's ADT vector p_ic via the Stoeckius et al. log-ratio transform, ln(p_ic / geometric mean); training on this transformed relative abundance outperforms training on raw protein barcode counts. Save the normalized denoised expression matrix and transformed protein matrix together (e.g. preprocessed_input.rds), which then serve as training input X and response vectors for the cTP-net multiple-branch neural network.

## Related tools

- **SAVER-X** (Denoises the raw UMI scRNA-seq count matrix (default parameters) to produce the denoised expression estimates Λ used as cTP-net training input)
- **Seurat** (Provides the default LogNormalize function applied to the SAVER-X denoised counts (scale factor 10000))
- **cTP-net** (Consumes the normalized denoised RNA matrix and the CLR-transformed protein abundances for surface-protein imputation; R/Python package provides preprocess_seurat2/3, preprocess_matrix and postprocess_* helpers) — https://github.com/zhouzilu/cTPnet
- **R** (Language for the cTPnet R package vignettes (Seurat v2/v3 notebooks) and preprocessing helpers) — https://github.com/zhouzilu/cTPnet

## Examples

```
out <- preprocess_seurat3(seurat_data, dprotein)  # cTPnet R helper: Seurat v3 preprocessing of paired RNA/ADT data
```

## Evaluation signals

- preprocessed_input.rds exists, is a valid RDS file, and contains both the RNA and protein matrices with cell counts matching the input (e.g. 4000-cell demo)
- Response vectors contain no missing genes; genes present in training but missing from testing data are filled with zeros to align feature spaces
- Input expression is confirmed UMI-based counts, as required by cTP-net
- Skipping SAVER-X denoising should degrade downstream cTP-net correlation by ~0.02 relative to the denoised pipeline, confirming the denoising step was applied
- Training on the CLR-transformed relative protein abundance should yield better prediction accuracy than a control network trained on raw protein barcode counts

## Limitations

- cTP-net can only apply to count-based (UMI) expression input; accuracy for TPM/RPKM CITE-seq data is unknown
- Generalization to cell types outside the hematopoietic lineage has not been tested; current models target 24 immunophenotype markers (a 100-protein model is still in training)
- Benchmark comparisons are limited to the same cells as the validation set to account for variations between subsets; Windows users may hit a Python 'ctpnet' module import error fixed by numpy==1.19.3

## Evidence

- [discussion] LogNormalize with SAVER-X pipeline: "Denoised counts (Λ) were further transformed with Seurat default LogNormalize function"
- [discussion] CLR transform of ADT counts: "For the surface protein counts, we adopted the relative abundance transformation from Stoeckius et al."
- [discussion] SAVER-X denoising rationale: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] Transformed abundance beats raw counts: "The network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than the network trained using raw protein barcode counts."
- [readme] README pipeline: SAVER-X recommended before imputation: "(1) **SAVERX**, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance"
