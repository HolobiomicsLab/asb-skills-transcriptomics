---
name: imputation-correlation-analysis-vs-measured-proteins
description: Use when training and test single-cell multi-omic data are not perfectly
  cell-type aligned or come from different tissues/protocols (PBMC vs CBMC, CITE-seq
  vs REAP-seq), and per-protein imputation accuracy must be quantified against measured
  protein levels.
when_to_use_negative:
- Input expression is TPM or RPKM rather than UMI-based counts — cTP-net only applies
  to count-based (UMI) expression input, and accuracy for TPM/RPKM data is untested
- Target cells are not of the hematopoietic lineage — generalization of these immune-marker
  imputations outside hematopoietic cells was not evaluated
- No measured protein (ADT) data are available for the test set, since this skill's
  evaluation requires computing correlations against measured protein abundances
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: cTP-net
  role: Transfer-learning deep neural network that imputes surface protein abundances
    from denoised scRNA-seq data; model trained on one tissue/protocol and applied
    to the other before correlation testing
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises the scRNA-seq count matrix before imputation; required preprocessing
    step whose contribution (~0.02 correlation) defines the baseline the imputation
    must exceed
- name: Seurat
  role: Provides the default LogNormalize transformation applied to SAVER-X denoised
    counts
- name: Adam (optimizer)
  role: Optimizes the cTP-net objective with learning rate 10e-5 for 139 epochs when
    training the cross-tissue model
- name: Python
  role: Language for the shared training script and cTP-net package
  repo: https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py
- name: R
  role: Language for the cTP-net package and Seurat v2/v3 vignette workflows
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:c3348407e63fa894bd878f55a6c91cbf35d9b183e7ead7128ad6974b87dcfdd3
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# imputation-correlation-analysis-vs-measured-proteins

## Summary

Cross-cell Pearson correlation benchmarking of cTP-net-imputed surface protein abundances against measured CITE-seq/REAP-seq ADT counts, specifically to test generalization when the test tissue or protocol differs from training (e.g., train CITE-CBMC → test CITE-PBMC, train CITE-seq → test REAP-seq).

## When to use

Use when training and test single-cell multi-omic data are not perfectly cell-type aligned or come from different tissues/protocols (PBMC vs CBMC, CITE-seq vs REAP-seq), and per-protein imputation accuracy must be quantified against measured protein levels.

## When NOT to use

- Input expression is TPM or RPKM rather than UMI-based counts — cTP-net only applies to count-based (UMI) expression input, and accuracy for TPM/RPKM data is untested
- Target cells are not of the hematopoietic lineage — generalization of these immune-marker imputations outside hematopoietic cells was not evaluated
- No measured protein (ADT) data are available for the test set, since this skill's evaluation requires computing correlations against measured protein abundances

## Inputs

- Paired scRNA-seq + surface-protein (ADT) count matrices: CITE-PBMC, CITE-CBMC, REAP-PBMC (Supplementary Table 1)
- SAVER-X-denoised, LogNormalize-transformed gene expression matrix (UMI-based counts)
- ADT counts transformed to relative abundance via geometric mean log-ratio (Stoeckius et al.)
- Pre-trained cTP-net model file trained on the donor tissue/protocol

## Outputs

- Per-protein cross-cell Pearson correlation table of imputed vs measured protein abundance (Fig. 2a style)
- Cross-tissue comparison table (CITE-CBMC↔CITE-PBMC, both directions)
- Cross-protocol comparison table (CITE-seq-trained model applied to REAP-PBMC)
- Baseline correlation values for raw and SAVER-X denoised RNA of corresponding genes

## How to apply

Load the paired transcriptome–protein datasets (CITE-PBMC, CITE-CBMC, REAP-PBMC per Supplementary Table 1), denoise scRNA-seq counts with SAVER-X (default parameters), LogNormalize the denoised counts, and apply the Stoeckius et al. relative abundance (geometric mean log-ratio) transformation to ADT counts so responses are comparable across protocols. Train the cTP-net multiple-branch network with Adam (learning rate 10e-5, 139 epochs) on one tissue/protocol and impute the other; align feature spaces by filling genes present in training but absent in testing with zeros. For each protein, compute cross-cell Pearson correlation between imputed and measured relative protein abundance and reproduce the Fig. 2a and Supplementary Figs. 3b/4a comparisons. Accept the result only when imputation correlations clearly exceed denoised RNA-level correlations, remembering SAVER-X denoising alone contributes only ~0.02 correlation gain, so the cTP-net output must surpass that baseline by a wide margin.

## Related tools

- **cTP-net** (Transfer-learning deep neural network that imputes surface protein abundances from denoised scRNA-seq data; model trained on one tissue/protocol and applied to the other before correlation testing) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises the scRNA-seq count matrix before imputation; required preprocessing step whose contribution (~0.02 correlation) defines the baseline the imputation must exceed)
- **Seurat** (Provides the default LogNormalize transformation applied to SAVER-X denoised counts)
- **Adam (optimizer)** (Optimizes the cTP-net objective with learning rate 10e-5 for 139 epochs when training the cross-tissue model)
- **Python** (Language for the shared training script and cTP-net package) — https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py
- **R** (Language for the cTP-net package and Seurat v2/v3 vignette workflows) — https://github.com/zhouzilu/cTPnet

## Examples

```
R: cTPnet(data = denoised_pbmc_matrix, data_type = "Seurat3", model_file_path = "model_CITE-CBMC.rds", dprotein = gene_names_protein12)
```

## Evaluation signals

- Imputation correlations across measured proteins markedly exceed correlations of denoised RNA levels alone for the same proteins
- Imputed-vs-measured correlation gain attributable to SAVER-X denoising alone is ~0.02, so cTP-net output must clearly surpass that baseline
- Cross-tissue results (train CBMC → impute PBMC and reverse) should approach, but be slightly less accurate than, same-population imputation
- Per-protein correlation table reproduces the ordering/comparisons of Fig. 2a and Supplementary Figs. 3b and 4a
- Response vectors were produced by the geometric mean log-ratio (relative abundance) transformation, not raw ADT barcode counts, which the paper reports gives better accuracy

## Limitations

- CITE-seq data with TPM and RPKM expression metrics were not available for testing, so accuracy under those metrics is unknown
- Generalization to cell types outside the hematopoietic lineage was not tested
- cTP-net can only apply to count-based expression input (UMI-based)
- Cross-tissue imputation is slightly less accurate than imputation from a model trained on the same population
- Comparisons are limited to cells in the validation set of the holdout scheme to account for subset variation

## Evidence

- [discussion] cTP-net was benchmarked on PBMC and CBMC immune cells, showing good generalization across tissues and technical protocols.: "cTP-net was benchmarked on PBMC and CBMC immune cells, showing good generalization across tissues and technical protocols."
- [discussion] Adam training parameters: "The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs"
- [discussion] Missing-gene zero-filling rule: "Genes that are in the training data but not in the testing data are filled with zeros."
- [discussion] SAVER-X baseline contribution: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] Dataset inventory: "Supplementary Table 1 summarizes the five data sets analyzed in this study: CITE-PBMC, CITE-CBMC, REAP-PBMC, HCA-CBMC, and HCA-BMMC."
- [readme] Package purpose from README: "is a transfer learning framework to impute surface protein abundances from scRNA-seq data by learning from existing single-cell multi-omic resources"
- [methods] Quality filter thresholds: "low-quality gene (<10 counts across cells) and low-quality cells (<200 genes detected) are removed"
