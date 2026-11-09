---
name: neural-encoder-architecture-comparison
description: Use when benchmarking whether a denoising autoencoder with Leiden cell-cluster
  label regularization outperforms a plain AE or unregularized variant for single-cell
  drug sensitivity prediction.
when_to_use_negative:
- When the question is the contribution of transfer learning or bulk-database choice
  (GDSC vs CCLE vs integrated) rather than encoder architecture — use the corresponding
  ablation arms instead
- When only a single trained model's predictions are needed and no architecture comparison
  is intended
- When the input is bulk-only drug response data with no single-cell benchmark labels
  to compute F1 against
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: scDEAL
  role: Framework whose AE vs DAE encoders and --mod ori/new loss-regularization modes
    are being compared in the ablation
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Backend for training the encoder arms (CPU version supported; demo training
    ~20 minutes)
- name: Scanpy
  role: Provides Leiden cell-cluster labels used as the --mod "new" loss regularization
    signal
- name: AnnData
  role: Output format (.h5ad) carrying adata.obs["sens_label"] sensitivity calls for
    F1 computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:9ffe7d8a6948220295f06b794f5c0db237bfb2451116757b754a5a48da7877c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - PyTorch
  - Scanpy
  - conda
  - scDEAL
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# neural-encoder-architecture-comparison

## Summary

Ablation protocol for isolating the contribution of encoder architecture (AE vs DAE) and cell-type-regularized loss (--mod "new" vs "ori") in scDEAL drug-response prediction, by training matched arms and comparing averaged F1 over 50 repeats.

## When to use

Use when benchmarking whether a denoising autoencoder with Leiden cell-cluster label regularization outperforms a plain AE or unregularized variant for single-cell drug sensitivity prediction.

## When NOT to use

- When the question is the contribution of transfer learning or bulk-database choice (GDSC vs CCLE vs integrated) rather than encoder architecture — use the corresponding ablation arms instead
- When only a single trained model's predictions are needed and no architecture comparison is intended
- When the input is bulk-only drug response data with no single-cell benchmark labels to compute F1 against

## Inputs

- Benchmark scRNA-seq dataset directory (e.g. data/GSE110894, .h5ad workflow)
- Bulk training expression matrices (ALL_expression.csv, ALL_label_binary_wf.csv) from GDSC/CCLE integration
- scDEAL conda environment (scdeal.tar.gz) with PyTorch, Scanpy, AnnData

## Outputs

- Per-repeat F1 values (range 0-1) for each ablation arm (50 repeats per arm)
- AnnData objects with per-cell predictions in adata.obs["sens_label"]
- Arm-level averaged F1 statistics written to fig2_ablation_f1_summary.csv
- Relative F1 increase of DAE+regularization over AE and over DAE-alone baselines

## How to apply

Set up the scDEAL conda environment (scDEALenv via conda-pack) and load the benchmark scRNA-seq data (e.g. GSE110894 / I.BET.762) from the data directory. Train the architecture-ablation arms for 50 repeats each, comparing the AE encoder against the DAE encoder, each with --mod "ori" (no cell-type information) versus --mod "new" (Leiden cell-cluster label regularization added to the loss), keeping encoder dims "256,128", bottleneck 512, predictor dims "128,64", dropout 0.1, lr 0.5, and sampling "upsampling" fixed so only the architecture/regularization factor varies. Use --checkpoint "False" for from-scratch training so pretrained weights do not confound the comparison. Extract per-cell sensitivity calls from adata.obs["sens_label"] (0=resistance, 1=sensitivity), compute F1 against benchmark labels per repeat, and average within each arm. Rationale: DAE denoising plus cluster-label regularization preserves single-cell heterogeneity that a plain AE loses during bulk-to-single-cell transfer.

## Related tools

- **scDEAL** (Framework whose AE vs DAE encoders and --mod ori/new loss-regularization modes are being compared in the ablation) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Backend for training the encoder arms (CPU version supported; demo training ~20 minutes))
- **Scanpy** (Provides Leiden cell-cluster labels used as the --mod "new" loss regularization signal)
- **AnnData** (Output format (.h5ad) carrying adata.obs["sens_label"] sensitivity calls for F1 computation)

## Examples

```
python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --lr 0.5 --sampling "upsampling" --printgene "F" -mod "new" --checkpoint "False"
```

## Evaluation signals

- Each ablation arm contains exactly 50 repeat F1 values, all within the range 0-1
- DAE with cell-type regularization shows ~36% relative F1 increase over plain AE and ~9% over DAE alone
- Sensitivity calls in adata.obs["sens_label"] use the 0=resistance / 1=sensitivity encoding and match the benchmark label schema
- Non-varied hyperparameters (encoder dims "256,128", bottleneck 512, predictor dims "128,64", dropout 0.1, lr 0.5, sampling "upsampling") are identical across all arms
- Arm-level averages written to fig2_ablation_f1_summary.csv and relative increases computed over the correct baseline arms

## Limitations

- Prediction accuracy depends on the collection of bulk gene expression of cell lines; additional bulk databases may change the ablation outcome
- Only a limited number of experimentally validated drug-response scRNA-seq datasets exist to determine better model hyperparameters
- F1 gains are dataset- and drug-specific (e.g. GSE110894 / I.BET.762 settings); gains may not transfer to other drugs or tissues

## Evidence

- [results] Ablation studies verified each design choice: [evidence span withheld — anchor status: not_found]
- [intro] Cell cluster label regularization rationale: "in order to avoid losing heterogeneity in scRNA-seq data, scDEAL includes cell cluster labels for loss function regularization in each training epoch"
- [readme] Checkpoint-based from-scratch training: "We can also train bulkmode.py and scmodel.py from scratch with user-defined parameters by setting --checkpoint "False""
- [methods] Output AnnData format: "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad) applied by the scanpy package"
