---
name: cross-tissue-protein-imputation-benchmarking
description: Use when cell types in training and test single-cell multi-omic data
  are not perfectly aligned and you need to test whether a trained protein-imputation
  model generalizes across tissues or across CITE-seq/REAP-seq protocols.
when_to_use_negative:
- Input expression data is not UMI-based counts (e.g., TPM or RPKM) — the article
  states prediction accuracy for such metrics is unknown and cTP-net 'can only apply
  to count-based expression input (UMI-based)'
- Cells are outside the hematopoietic lineage — cross-tissue/protocol generalization
  was only benchmarked on immune (PBMC/CBMC/BMMC) cells, so results there are unsupported
- Training and test data are from the same population and same protocol — a same-tissue
  model comparison, not a generalization benchmark, is what is being run
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0199
tools:
- name: cTP-net
  role: Multiple-branch neural network trained with Adam (lr 10e-5, 139 epochs) on
    one tissue/protocol and applied to impute proteins in the other
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises scRNA-seq count matrices (default parameters) before training/imputation;
    its contribution (~0.02 correlation) sets the baseline to beat
- name: Seurat
  role: Default LogNormalize transformation of SAVER-X denoised counts to produce
    the training/imputation input matrix
- name: Adam (optimizer)
  role: Stochastic optimization of the cTP-net objective during cross-tissue model
    training
- name: Python
  role: Environment for running cTPnet training and imputation (package available
    in Python and R)
  repo: https://github.com/zhouzilu/cTPnet
- name: R
  role: Environment for cTPnet vignette-based imputation (Seurat v2/v3 wrappers)
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:cbaa49745e9881149579ee3081455ec6f5e5c30b7c65e47db1f971145acef176
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
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cross-tissue-protein-imputation-benchmarking

## Summary

Benchmark cTP-net's ability to impute surface protein abundances when the test tissue (e.g., CITE-PBMC vs CITE-CBMC) or protocol (CITE-seq vs REAP-seq) differs from the training data, comparing per-protein Pearson correlations of imputed values against denoised RNA baselines.

## When to use

Use when cell types in training and test single-cell multi-omic data are not perfectly aligned and you need to test whether a trained protein-imputation model generalizes across tissues or across CITE-seq/REAP-seq protocols.

## When NOT to use

- Input expression data is not UMI-based counts (e.g., TPM or RPKM) — the article states prediction accuracy for such metrics is unknown and cTP-net 'can only apply to count-based expression input (UMI-based)'
- Cells are outside the hematopoietic lineage — cross-tissue/protocol generalization was only benchmarked on immune (PBMC/CBMC/BMMC) cells, so results there are unsupported
- Training and test data are from the same population and same protocol — a same-tissue model comparison, not a generalization benchmark, is what is being run

## Inputs

- CITE-PBMC paired scRNA-seq + ADT protein count matrix (Supplementary Table 1)
- CITE-CBMC paired scRNA-seq + ADT protein count matrix (Supplementary Table 1)
- REAP-PBMC paired scRNA-seq + protein barcode count matrix (Supplementary Table 1)
- Pre-trained cTP-net model file(s) or the training script training_05152020.py for refitting on the alternate tissue

## Outputs

- Per-protein cross-cell Pearson correlation table (imputed vs measured protein abundance) for each tissue/protocol transfer direction
- Comparison tables reproducing Fig. 2a and Supplementary Figs. 3b, 4a against denoised RNA baselines
- Imputed relative surface protein abundance matrix for the held-out tissue/protocol

## How to apply

Load the CITE-PBMC, CITE-CBMC, and REAP-PBMC paired transcriptome-protein datasets from Supplementary Table 1, then denoise scRNA-seq count matrices with SAVER-X (default parameters) and apply Seurat default LogNormalize. Transform surface protein ADT counts with the relative abundance (geometric mean log-ratio) transformation from Stoeckius et al. to form response vectors. Train the cTP-net multiple-branch neural network with Adam (learning rate 10e-5, 139 epochs) on CITE-CBMC and impute CITE-PBMC proteins, then reverse the roles; align feature spaces by filling genes present in training but absent in testing with zeros. Apply the CITE-seq-trained model to REAP-PBMC to test cross-protocol generalization, and compute cross-cell Pearson correlations per protein between imputed and measured abundances. Judge success when imputation correlations markedly exceed correlations of denoised RNA alone — note SAVER-X denoising alone accounts for only ~0.02 correlation gain, so cTP-net output must clearly surpass that baseline (Fig. 2a, Supplementary Figs. 3b, 4a).

## Related tools

- **cTP-net** (Multiple-branch neural network trained with Adam (lr 10e-5, 139 epochs) on one tissue/protocol and applied to impute proteins in the other) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises scRNA-seq count matrices (default parameters) before training/imputation; its contribution (~0.02 correlation) sets the baseline to beat)
- **Seurat** (Default LogNormalize transformation of SAVER-X denoised counts to produce the training/imputation input matrix)
- **Adam (optimizer)** (Stochastic optimization of the cTP-net objective during cross-tissue model training)
- **Python** (Environment for running cTPnet training and imputation (package available in Python and R)) — https://github.com/zhouzilu/cTPnet
- **R** (Environment for cTPnet vignette-based imputation (Seurat v2/v3 wrappers)) — https://github.com/zhouzilu/cTPnet

## Examples

```
cTPnet(data = pbmc_denoised_lognorm, data_type = "scRNAseq", model_file_path = "model/CITE_CBMC_trained", dprotein = gene_names_protein12)
```

## Evaluation signals

- Per-protein cross-cell Pearson correlations of imputed proteins must markedly exceed correlations of denoised RNA of the corresponding gene
- Since SAVER-X denoising alone retracts performance by 0.02 when omitted, cTP-net predictions must clearly surpass the ~0.02 denoising-only gain baseline
- Benchmark tables should reproduce the qualitative pattern of Fig. 2a and Supplementary Figs. 3b, 4a: cross-tissue (CBMC-trained → PBMC and reversed) slightly below same-population accuracy but well above RNA baseline
- Cross-protocol transfer (CITE-seq-trained model on REAP-PBMC) should show good generalization to measured protein abundances
- Feature-space alignment check: genes in training but not testing must be zero-filled before prediction, and the model must remain UMI-count-based input

## Limitations

- cTP-net can only apply to count-based expression input (UMI-based); accuracy with TPM/RPKM is unknown
- Imputation of immune-related markers on cells outside the hematopoietic lineage was not tried, so cross-tissue generalization claims are restricted to hematopoietic populations
- Cross-tissue imputation is slightly less accurate than imputation from a model trained on the same population
- Comparisons in the benchmark are limited to the same cells that were in the validation set to account for variation between subsets
- The cTP-net model imputes 24 immunophenotype markers; a 100-protein model is still in training

## Evidence

- [discussion] cross-tissue imputation baseline: "cTP-net was benchmarked on PBMC and CBMC immune cells, showing good generalization across tissues and technical protocols."
- [discussion] denoising baseline gain: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] optimization hyperparameters: "The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs"
- [discussion] feature-space alignment: "Genes that are in the training data but not in the testing data are filled with zeros."
- [discussion] input restriction: "cTP-net can only apply to count-based expression input (UMI-based)."
- [readme] tool availability: "This cTPnet package includes two analysis tools: (1) **SAVERX**, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance"
- [methods] cTPnet function call: "- `cTPnet(data, data_type, model_file_path, dprotein)`"
