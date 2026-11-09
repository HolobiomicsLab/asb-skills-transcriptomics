---
name: denoising-autoencoder-training
description: Use when predicting single-cell drug responses from scRNA-seq by transferring
  a model trained on bulk cell-line drug response data (GDSC/CCLE), e.g. scoring GSE110894
  AML cells for I.BET.762 sensitivity.
when_to_use_negative:
- Input lacks bulk-level drug response training labels (the transfer relies on GDSC/CCLE-style
  bulk response annotations); scDEAL cannot be trained without them
- Task is generic single-cell clustering or dimensionality reduction without a drug-response
  prediction goal — the DaNN/MMD transfer machinery is unnecessary overhead
- The scRNA-seq data is mouse and homologous gene mapping was not performed (in the
  mouse case study only homologous genes were kept before applying the human-trained
  model)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDEAL
  role: Framework implementing the DaNN-based deep transfer learning with DAE encoders;
    provides bulkmodel.py and scmodel.py entry points
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: 'Backend used to build and train the denoising-autoencoder encoders and predictor
    (demo: ~20 minutes on pytorch cpu version)'
- name: Scanpy
  role: Package used with AnnData objects that carry the per-cell predicted sens_label
    output
- name: AnnData
  role: Output container (.h5ad) holding scored cells with obs['sens_label']
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:8533e687cebdc899744c4684e3c4626a2274545b5b45cc03da59b3264c346479
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - PyTorch
  - Scanpy
  - scDEAL
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# denoising-autoencoder-training

## Summary

Trains scDEAL's denoising-autoencoder (DAE) encoders for bulk and single-cell RNA-seq (encoder hidden dims 256,128; bottleneck 512; dropout 0.1) together with a drug-response predictor, so bulk-trained drug response knowledge can be transferred to scRNA-seq cells via a DaNN/MMD loss.

## When to use

Use when predicting single-cell drug responses from scRNA-seq by transferring a model trained on bulk cell-line drug response data (GDSC/CCLE), e.g. scoring GSE110894 AML cells for I.BET.762 sensitivity.

## When NOT to use

- Input lacks bulk-level drug response training labels (the transfer relies on GDSC/CCLE-style bulk response annotations); scDEAL cannot be trained without them
- Task is generic single-cell clustering or dimensionality reduction without a drug-response prediction goal — the DaNN/MMD transfer machinery is unnecessary overhead
- The scRNA-seq data is mouse and homologous gene mapping was not performed (in the mouse case study only homologous genes were kept before applying the human-trained model)

## Inputs

- Bulk expression matrix ALL_expression.csv (GDSC/CCLE cell-line RNA-seq)
- Binary drug response labels ALL_label_binary_wf.csv
- scRNA-seq data (e.g. GSE110894 acute myeloid leukemia, I-BET-762)
- Optional pre-trained model checkpoint (.pkl) for resuming training

## Outputs

- Trained DAE encoder + predictor checkpoint (.pkl) under save/sc_pre or save/models
- Scored AnnData (.h5ad) with adata.obs['sens_label'] (0=resistance, 1=sensitivity) written to save/adata

## How to apply

Load the bulk expression matrix (ALL_expression.csv) and binary drug response labels (ALL_label_binary_wf.csv) plus the scRNA-seq data, applying single-cell filters (min_g=200 genes per cell, min_c=3 cells per gene, percent_mito=100). Instantiate in PyTorch two denoising-autoencoder encoders (hidden dims 256,128; bottleneck 512; dropout 0.1; dimreduce 'DAE') and a predictor (dims 128,64), optionally initializing from a bulk-pretrained checkpoint via scmodel.py --checkpoint. Run joint training with --mod new so cell-type cluster labels regularize each epoch, using --sampling upsampling, --lr 0.5, and the MMD transfer loss (mmd_weight 0.25, mmd_GAMMA 1000), updating Eb, Es, and P simultaneously; the loss combines maximum mean discrepancy between bulk and single-cell latent spaces with cross-entropy against bulk drug-response labels. Save the checkpoint using the naming convention (e.g. integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl under save/sc_pre), keeping encoder/predictor dimensions identical across bulkmodel.py and scmodel.py steps.

## Related tools

- **scDEAL** (Framework implementing the DaNN-based deep transfer learning with DAE encoders; provides bulkmodel.py and scmodel.py entry points) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Backend used to build and train the denoising-autoencoder encoders and predictor (demo: ~20 minutes on pytorch cpu version))
- **Scanpy** (Package used with AnnData objects that carry the per-cell predicted sens_label output)
- **AnnData** (Output container (.h5ad) holding scored cells with obs['sens_label'])

## Examples

```
python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --lr 0.5 --sampling "upsampling" -mod "new" --checkpoint "False"
```

## Evaluation signals

- scmodel.py run with --checkpoint completes and produces a readable .h5ad whose obs contains sens_label with values in {0,1}
- Encoder and predictor dimensions are identical between bulkmodel.py and scmodel.py steps (--encoder_h_dims/bulk_h_dims '256,128', --predictor_h_dims '128,64', bottleneck 512)
- Saved checkpoint filename matches the naming convention, e.g. integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl
- Training log shows MMD transfer loss (mmd_weight 0.25, mmd_GAMMA 1000) plus cross-entropy updating Eb, Es, and P jointly each epoch
- Predicted sensitivity labels reproduce expected cell-type-level drug response patterns on benchmark data (e.g. GSE110894 I.BET.762 case)

## Limitations

- Prediction accuracy could vary depending on the collection of bulk gene expression of cell lines; training data will be updated by integrating additional bulk-level databases in the future
- More experimentally validated drug response scRNA-seq data would be needed to determine better model hyperparameters
- No software version numbers stated for tools or the environment; a potential sample swap between HN120 and HN137 (Data 1&2) was noted by a recent study

## Evidence

- [full_text] Training structure with MMD and cross-entropy: [evidence span withheld — anchor status: not_found]
- [intro] DaNN adaptation: "In this work, we develop scDEAL (single-cell Drug rEsponse AnaLysis) by adapting a Domain-adaptive Neural Network (DaNN)12 to predict drug responses from bulk and scRNA-seq data."
- [readme] Checkpoint demo invocation: "python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --printgene "F" -mod "new" --lr 0.5"
- [methods] PyTorch training time: "The training time of the test case including bulk-level and single-cell-level training on the testing computer was 20 minutes on the pytorch cpu version"
- [intro] Cell-type label regularization rationale: "in order to avoid losing heterogeneity in scRNA-seq data, scDEAL includes cell cluster labels for loss function regularization in each training epoch"
