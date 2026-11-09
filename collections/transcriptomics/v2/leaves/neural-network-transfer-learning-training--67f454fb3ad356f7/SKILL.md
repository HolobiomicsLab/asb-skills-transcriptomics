---
name: neural-network-transfer-learning-training
description: Use when you have paired transcriptome–surface-protein training data
  (e.g., CITE-PBMC, CITE-CBMC, REAP-PBMC) and want to train or fine-tune a model that
  imputes immunophenotype protein levels for transcriptome-only single-cell data.
when_to_use_negative:
- Input expression data is not UMI-based count data (e.g., TPM or RPKM metrics) —
  the article states prediction accuracy is unknown for such data and cTP-net 'can
  only apply to count-based expression input (UMI-based)'
- Target cell types are outside the hematopoietic lineage — the authors note they
  have not tried imputing these immune-related markers on non-hematopoietic cells
- The task is pure imputation with an existing pretrained 12/24-protein model rather
  than training a new model — use the cTPnet(data, data_type, model_file_path, dprotein)
  inference path instead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: cTP-net
  role: The multiple-branch neural network being trained; maps normalized denoised
    expression matrix X to relative surface protein abundance
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises the scRNA-seq count matrix (default parameters) before training;
    omitting it costs ~0.02 correlation
- name: Seurat
  role: LogNormalize transformation of SAVER-X denoised counts to produce the training
    input matrix
- name: Adam (optimizer)
  role: Stochastic optimizer for the training objective at learning rate 10e-5 for
    139 epochs
- name: Python
  role: Language of the shared training script (training_05152020.py); cTPnet package
    is also available in R
  repo: https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:a6de3dac1c76f4061ba6bb08dfdbef79d9be6ddf0c229b61890505eff2f81dcb
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# neural-network-transfer-learning-training

## Summary

Train a cTP-net multiple-branch deep neural network that maps denoised scRNA-seq transcriptomes to relative surface protein abundances (ADT counts), enabling transfer of protein-level information from paired CITE-seq/REAP-seq training data to transcriptome-only test data.

## When to use

Use when you have paired transcriptome–surface-protein training data (e.g., CITE-PBMC, CITE-CBMC, REAP-PBMC) and want to train or fine-tune a model that imputes immunophenotype protein levels for transcriptome-only single-cell data.

## When NOT to use

- Input expression data is not UMI-based count data (e.g., TPM or RPKM metrics) — the article states prediction accuracy is unknown for such data and cTP-net 'can only apply to count-based expression input (UMI-based)'
- Target cell types are outside the hematopoietic lineage — the authors note they have not tried imputing these immune-related markers on non-hematopoietic cells
- The task is pure imputation with an existing pretrained 12/24-protein model rather than training a new model — use the cTPnet(data, data_type, model_file_path, dprotein) inference path instead

## Inputs

- Paired scRNA-seq + surface protein (ADT) count matrix from a CITE-seq or REAP-seq dataset (e.g., CITE-PBMC, CITE-CBMC, REAP-PBMC, HCA-CBMC, HCA-BMMC per Supplementary Table 1)
- UMI-based scRNA-seq count matrix (gene x cell) for the training input
- Surface protein barcode count matrix for computing Stoeckius relative abundance response vectors
- Surface protein marker panel definition (e.g., the 24 immunophenotype markers; gene_names_protein12 in the cTPnet package)

## Outputs

- Trained cTP-net multiple-branch neural network model file (e.g., model_file_path artifact)
- Imputed surface protein relative abundance vector/matrix per cell for test data
- Cross-cell Pearson correlations of imputed vs. measured protein abundance per protein (Fig. 2a, Supplementary Figs. 3b, 4a style tables)

## How to apply

First remove low-quality genes (<10 counts across cells) and low-quality cells (<200 genes detected), then denoise the scRNA-seq count matrix with SAVER-X using default parameters and transform the denoised counts (Λ) with the Seurat default LogNormalize function — this denoising step accounts for ~0.02 of the final correlation gain and must not be skipped. Apply the relative abundance (geometric mean log-ratio) transformation from Stoeckius et al. to the surface protein ADT counts to form the training response vectors, since training on transformed relative abundance outperforms training on raw protein barcode counts. Split cells 90/10 into training and testing sets (or partition by cell type for out-of-cell-type prediction), select the optimal model on testing error, and optimize the objective stochastically with Adam at learning rate 10e-5 for 139 epochs. Align feature spaces between train and test data by filling genes present in training but absent from testing with zeros, and remember that the network supports only UMI-based count expression input.

## Related tools

- **cTP-net** (The multiple-branch neural network being trained; maps normalized denoised expression matrix X to relative surface protein abundance) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises the scRNA-seq count matrix (default parameters) before training; omitting it costs ~0.02 correlation)
- **Seurat** (LogNormalize transformation of SAVER-X denoised counts to produce the training input matrix)
- **Adam (optimizer)** (Stochastic optimizer for the training objective at learning rate 10e-5 for 139 epochs)
- **Python** (Language of the shared training script (training_05152020.py); cTPnet package is also available in R) — https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py

## Examples

```
python extdata/training_05152020.py  # shared cTP-net training script (Adam lr=10e-5, 139 epochs); for inference in R: cTPnet(data, data_type, model_file_path, dprotein)
```

## Evaluation signals

- Hold out 10% of cells as a testing set and confirm the selected model minimizes testing error on that split
- Imputed protein correlations with measured ADT levels must clearly exceed denoised-RNA-only baselines (SAVER-X denoising alone contributes only ~0.02 correlation gain over raw RNA)
- Cross-tissue transfer (train CITE-CBMC → impute CITE-PBMC and vice versa) should markedly beat denoised RNA correlation, though slightly under same-population-trained accuracy (Fig. 2a, Supplementary Figs. 3b, 4a)
- Feature-space alignment check: after zero-filling missing genes, training and test matrices must share identical gene dimensions (note the README dimension bug when object length ≠ 1992)
- Training on Stoeckius relative abundance response vectors should yield better validation accuracy than a model trained on raw protein barcode counts

## Limitations

- cTP-net only applies to UMI-based count expression input; CITE-seq data with TPM/RPKM metrics was not available for testing so accuracy is unknown
- Generalization to cell types outside the hematopoietic lineage was not tested
- Cross-tissue/cross-protocol imputation is slightly less accurate than a model trained on the same population
- Benchmark comparisons are limited to the same cells in the validation set due to variation between subsets
- Windows is not recommended (numpy must be pinned to 1.19.3 to avoid ModuleNotFoundError for 'ctpnet'); a 100-protein model was still in training at publication time

## Evidence

- [discussion] Adam optimizer and epoch parameters: "The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs"
- [discussion] 90/10 train-test split and model selection: "we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error."
- [discussion] Feature-space alignment by zero-filling: "Genes that are in the training data but not in the testing data are filled with zeros."
- [discussion] Importance of SAVER-X denoising before training: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [readme] Shared training script availability: "due to additional interets in training your own cTP-net model in the computational biology field, we have decided to share our training python code"
- [discussion] UMI-only input restriction: "cTP-net can only apply to count-based expression input (UMI-based)."
