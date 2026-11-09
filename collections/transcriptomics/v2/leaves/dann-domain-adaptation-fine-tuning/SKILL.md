---
name: dann-domain-adaptation-fine-tuning
description: Use when a deep model trained on bulk drug-response RNA-seq must be applied
  to drug-treated scRNA-seq data (e.g. GSE117872, GSE112274, GSE140440, GSE149383,
  GSE110894) to predict per-cell sensitivity/resistance labels.
when_to_use_negative:
- The task is bulk-level drug response prediction on cell-line data only — the DaNN
  transfer step is unnecessary; use bulkmodel.py alone.
- Input scRNA-seq data is untreated or lacks a corresponding drug for which bulk training
  labels exist (the predictor requires a named drug such as I.BET.762 or CISPLATIN).
- Mouse scRNA-seq data where only homologous genes should be kept and trans-species
  reliability has not been systematically evaluated.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scDEAL
  role: Deep transfer learning framework implementing the DaNN/DaNN-based MMD alignment
    that transfers bulk-trained drug-response labels to single cells
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Backend for the CPU-based bulkmodel.py and scmodel.py training runs (~20 minutes
    for the demo test case)
- name: Scanpy
  role: Applies/reads the AnnData (.h5ad) output objects carrying sens_label predictions
- name: AnnData
  role: Output format (.h5ad) for per-cell drug sensitivity predictions
- name: conda-pack
  role: Distributes the prepacked scDEAL conda environment (scdeal.tar.gz) needed
    to run the PyTorch scripts reproducibly
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:72d22c5c90bf85bc19db024def324f7db4ccd83c31072a8ebaff69507eb934a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - Anaconda
  - AnnData
  - PyTorch
  - Scanpy
  - conda
  - conda-pack
  - pip
  - scDEAL
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# dann-domain-adaptation-fine-tuning

## Summary

Transfers a bulk-trained drug-response predictor to single-cell RNA-seq data using DaNN-based MMD alignment in scDEAL, so sensitivity/resistance labels learned from bulk cell lines (GDSC/CCLE-derived ALL_expression.csv) are assigned per cell and evaluated against ground-truth labels.

## When to use

Use when a deep model trained on bulk drug-response RNA-seq must be applied to drug-treated scRNA-seq data (e.g. GSE117872, GSE112274, GSE140440, GSE149383, GSE110894) to predict per-cell sensitivity/resistance labels.

## When NOT to use

- The task is bulk-level drug response prediction on cell-line data only — the DaNN transfer step is unnecessary; use bulkmodel.py alone.
- Input scRNA-seq data is untreated or lacks a corresponding drug for which bulk training labels exist (the predictor requires a named drug such as I.BET.762 or CISPLATIN).
- Mouse scRNA-seq data where only homologous genes should be kept and trans-species reliability has not been systematically evaluated.

## Inputs

- Bulk drug-response training files ALL_expression.csv and ALL_label_binary_wf.csv (GDSC/CCLE-derived)
- Preprocessed drug-treated scRNA-seq datasets (e.g. GSE110894, GSE112274, GSE117872 HN120/HN137, GSE140440, GSE149383)
- Bulk model checkpoint from bulkmodel.py, or --checkpoint False to train from scratch
- Ground-truth resistant/sensitive cell labels for benchmarking

## Outputs

- AnnData (.h5ad) objects with adata.obs['sens_label'] (0 = resistance, 1 = sensitivity) in save/adata/data
- Trained DaNN single-cell model checkpoints (e.g. integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl)
- Per-dataset and average benchmark metrics: F1, AUROC, AP, precision, recall, AMI, ARI

## How to apply

First train the bulk-level model with bulkmodel.py (dimreduce DAE, drug-specific bottleneck/encoder_h_dims/predictor_h_dims, e.g. GSE110894: bottleneck 512, encoder_h_dims 256,128, predictor_h_dims 128,64, dropout 0.1, lr 0.5, sampling upsampling; GSE112274: bottleneck 256, encoder_h_dims 512,256, sampling no). Then run scmodel.py with identical architecture dimensions, which harmonizes single-cell and bulk embeddings via DaNN-based MMD alignment (defaults mmd_weight 0.25, mmd_GAMMA 1000) and uses Leiden clustering labels (resolution 0.3) for loss regularization so heterogeneity is preserved. Store the AnnData output (.h5ad) with adata.obs['sens_label'] (0 = resistance, 1 = sensitivity) under save/adata/data. Compute F1, AUROC, AP, precision, recall, AMI, ARI against reported resistant/sensitive cell labels; expect averages around F1 0.892, AUROC 0.898, AP 0.944.

## Related tools

- **scDEAL** (Deep transfer learning framework implementing the DaNN/DaNN-based MMD alignment that transfers bulk-trained drug-response labels to single cells) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Backend for the CPU-based bulkmodel.py and scmodel.py training runs (~20 minutes for the demo test case))
- **Scanpy** (Applies/reads the AnnData (.h5ad) output objects carrying sens_label predictions)
- **AnnData** (Output format (.h5ad) for per-cell drug sensitivity predictions)
- **conda-pack** (Distributes the prepacked scDEAL conda environment (scdeal.tar.gz) needed to run the PyTorch scripts reproducibly)

## Examples

```
python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --lr 0.5 --sampling "upsampling" --mod "new" --checkpoint "False"
```

## Evaluation signals

- Predicted sens_label values are strictly binary 0/1 in adata.obs of the saved .h5ad
- Benchmark metrics fall near the reported values (e.g. Data 1: F1 0.92, AUROC 0.92, AP 0.97); six-dataset averages F1 0.892, AUROC 0.898, AP 0.944, precision 0.926, recall 0.899, AMI 0.528, ARI 0.608
- Encoder and predictor dimensions passed to scmodel.py exactly match the bulkmodel.py run (--encoder_h_dims/--bulk_h_dims identical, e.g. 256,128 with bottleneck 512 and predictor 128,64)
- Model checkpoint file exists under save/sc_pre following the documented naming pattern (bulk data + scRNA dataset + drug + dims + dropout + lr + mod + sampling + DaNN.pkl)
- Demo run completes in roughly 20 minutes on the PyTorch CPU version for the GSE110894 test case

## Limitations

- Prediction accuracy varies depending on the collection of bulk gene expression of cell lines; the GDSC/CCLE training set may need future expansion.
- A potential sample swap was reported between Data 1 and 2 (HN120 and HN137 of GSE117872), affecting interpretation of those benchmark results.
- Trans-species (human-to-mouse) reliability could not be systematically evaluated due to the limited number of drug-treated mouse benchmark scRNA-seq datasets.
- More experimentally validated drug-response scRNA-seq data are needed to determine better model hyperparameters; no software version numbers are stated for the environment.

## Evidence

- [intro] In this work, we develop scDEAL (single-cell Drug rEsponse AnaLysis) by adapting a Domain-adaptive Neural Network (DaNN)12 to predict drug responses from bulk and scRNA-seq data.: "In this work, we develop scDEAL (single-cell Drug rEsponse AnaLysis) by adapting a Domain-adaptive Neural Network (DaNN)12 to predict drug responses from bulk and scRNA-seq data."
- [intro] scDEAL harmonizes single-cell and bulk embeddings: "scDEAL harmonizes single-cell and bulk embeddings to ensure that the drug response labels are transferable from"
- [readme] checkpoint naming and MMD/Leiden defaults: "integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl"
- [readme] The training time of the test case including bulk-level and single-cell-level training on the testing computer was 20 minutes on the pytorch cpu version: "The training time of the test case including bulk-level and single-cell-level training on the testing computer was 20 minutes on the pytorch cpu version"
- [methods] dataset-to-drug mapping in methods: "Data 1&2  |     Sharma,   et al.     |     Cisplatin    |     GSE117872"
- [discussion] sample swap caveat in discussion: "a potential sample swap between Data 1 and 2 (HN120 and HN137)49"
