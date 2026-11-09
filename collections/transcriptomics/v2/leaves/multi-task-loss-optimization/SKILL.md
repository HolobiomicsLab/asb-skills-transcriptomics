---
name: multi-task-loss-optimization
description: Use when transferring bulk-level GDSC/CCLE drug response labels to scRNA-seq
  data with scDEAL, i.e. when training Eb, Es, and P simultaneously so single-cell
  and bulk embeddings are harmonized (MMD) while prediction accuracy is supervised.
when_to_use_negative:
- Only bulk RNA-seq drug response data is available with no scRNA-seq target dataset
  — the MMD/harmonization term has no single-cell domain to align to; use bulkmodel.py
  training alone
- A converged single-cell checkpoint already exists and the task is only per-cell
  prediction/application — skip joint re-optimization and run scmodel.py with --checkpoint
- The drug response labels are continuous IC50 values rather than the binary sensitivity/resistance
  labels the cross-entropy loss requires
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scDEAL
  role: DaNN-adapted deep transfer learning framework whose joint multi-task loss
    (MMD + cross-entropy + cell-type regularization) is optimized by this skill
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Deep learning backend in which Eb, Es, and P are instantiated and updated
    simultaneously during the joint training epoch loop
- name: Scanpy
  role: Preprocesses the scRNA-seq input (filtering, AnnData handling) fed into the
    joint training
- name: AnnData
  role: Container format (.h5ad) for the scored per-cell predictions written out after
    training
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:43cebaa1ca7af80244b8244d305309fb1d8ca6d57d532770e35a246aa7009bb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - PyTorch
  - Scanpy
  - scDEAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multi-task-loss-optimization

## Summary

Jointly optimizes scDEAL's bulk encoder (Eb), single-cell encoder (Es), and drug-response predictor (P) in PyTorch by combining a maximum mean discrepancy (MMD/DaNN) transfer loss between bulk and single-cell embeddings with cross-entropy loss on predicted bulk drug responses, plus per-epoch cell-type label regularization.

## When to use

Use when transferring bulk-level GDSC/CCLE drug response labels to scRNA-seq data with scDEAL, i.e. when training Eb, Es, and P simultaneously so single-cell and bulk embeddings are harmonized (MMD) while prediction accuracy is supervised.

## When NOT to use

- Only bulk RNA-seq drug response data is available with no scRNA-seq target dataset — the MMD/harmonization term has no single-cell domain to align to; use bulkmodel.py training alone
- A converged single-cell checkpoint already exists and the task is only per-cell prediction/application — skip joint re-optimization and run scmodel.py with --checkpoint
- The drug response labels are continuous IC50 values rather than the binary sensitivity/resistance labels the cross-entropy loss requires

## Inputs

- Bulk expression matrix (ALL_expression.csv) with binary drug response annotations (ALL_label_binary_wf.csv) for a given drug (e.g. I.BET.762)
- Preprocessed scRNA-seq dataset (e.g. GSE110894) filtered with min_g=200 genes per cell, min_c=3 cells per gene, percent_mito=100
- Optional pretrained bulk model checkpoint (.pkl) from save/bulk_pre
- Cell cluster labels for loss regularization under --mod new

## Outputs

- Jointly trained single-cell model checkpoint (save/sc_pre/integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl)
- Scored AnnData object (.h5ad) in save/adata with adata.obs['sens_label'] (0=resistance, 1=sensitivity) per cell

## How to apply

Instantiate in PyTorch two denoising-autoencoder encoders (hidden dims 256,128; bottleneck 512; dropout 0.1; dimreduce 'DAE') and a predictor (dims 128,64), optionally initializing from a bulk-pretrained checkpoint via scmodel.py --checkpoint. Run joint training with --mod new so cell cluster labels regularize each epoch, using --sampling upsampling, --lr 0.5, and the MMD transfer loss configured with mmd_weight 0.25 and mmd_GAMMA 1000, updating Eb, Es, and P simultaneously in each step. The rationale is that the MMD term minimizes maximum mean discrepancy between the low-dimensional feature spaces of single-cell and bulk data, while the cross-entropy loss between predicted bulk cell-line drug responses and ground-truth labels keeps the predictor supervised; the cell-cluster label regularization prevents loss of scRNA-seq heterogeneity. Save the trained model using the checkpoint naming convention (e.g. integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl in save/sc_pre). Keep encoder and predictor dimensions identical across the bulkmodel.py and scmodel.py stages so checkpoints can be loaded for resuming or applying the model.

## Related tools

- **scDEAL** (DaNN-adapted deep transfer learning framework whose joint multi-task loss (MMD + cross-entropy + cell-type regularization) is optimized by this skill) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Deep learning backend in which Eb, Es, and P are instantiated and updated simultaneously during the joint training epoch loop)
- **Scanpy** (Preprocesses the scRNA-seq input (filtering, AnnData handling) fed into the joint training)
- **AnnData** (Container format (.h5ad) for the scored per-cell predictions written out after training)

## Examples

```
python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --printgene "F" -mod "new" --lr 0.5 --sampling "upsampling" --checkpoint "False"
```

## Evaluation signals

- The scmodel.py run with --checkpoint completes and saves a readable .pkl under save/sc_pre following the checkpoint naming convention
- The resulting .h5ad loads and adata.obs['sens_label'] contains only values in {0,1}
- Both training stages use identical dimensions (--bulk_h_dims/encoder_h_dims '256,128', --predictor_h_dims '128,64', bottleneck 512) so checkpoints load without shape mismatch
- Training completes within the expected ~20 minutes on the PyTorch CPU version for the demo test case
- With --mod new, cell cluster labels are consumed in each training epoch without collapsing single-cell heterogeneity

## Limitations

- Prediction accuracy could vary depending on the collection of bulk gene expression of cell lines; scDEAL training data will be updated by integrating additional bulk-level databases
- No software version numbers are stated for tools or the environment, so exact reproducibility of the loss behavior is not guaranteed
- Sampling strategy and hyperparameters (lr 0.5, mmd_weight 0.25, mmd_GAMMA 1000, upsampling) were tuned for the benchmark datasets listed in the README; other datasets may need grid-search optimization

## Evidence

- [abstract] The overall framework is trained by considering the maximum mean discrepancy between the low-dimensional feature spaces of single-cell and bulk data, together with the cross-entropy loss between predicted bulk cell-line drug responses and their ground-truth labels.: [evidence span withheld — anchor status: not_found]
- [intro] scDEAL harmonizes single-cell and bulk embeddings so that drug response labels are transferable from bulk to single cells, accounting for data-structure differences.: "scDEAL harmonizes single-cell and bulk embeddings to ensure that the drug response labels are transferable from"
- [intro] In order to avoid losing heterogeneity in scRNA-seq data, scDEAL includes cell cluster labels for loss function regularization in each training epoch.: "in order to avoid losing heterogeneity in scRNA-seq data, scDEAL includes cell cluster labels for loss function regularization in each training epoch"
- [readme] Remember that the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64", --bottleneck 256) in two steps.: "the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64""
- [methods] The training time of the test case including bulk-level and single-cell-level training on the testing computer was 20 minutes on the pytorch cpu version.: "The training time of the test case including bulk-level and single-cell-level training on the testing computer was 20 minutes on the pytorch cpu version"
