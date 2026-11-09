---
name: bulk-single-cell-rnaseq-integration
description: Use when predicting single-cell drug sensitivity/resistance labels from
  scRNA-seq (h5ad/AnnData) by leveraging bulk-level drug-response training data, e.g.
  reproducing the GSE110894 I-BET case study with scDEAL.
when_to_use_negative:
- Input is already a per-cell drug response label table with no bulk training data
  to transfer from — no domain adaptation is possible
- The scRNA-seq species shares no homologous genes with the human-trained model (the
  article only kept homologous genes for mouse Data 6); trans-species reliability
  was not systematically evaluated
- The task is bulk-only drug response prediction without single-cell resolution —
  scDEAL's DaNN transfer step is unnecessary
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scDEAL
  role: Deep transfer learning framework (bulkmodel.py + scmodel.py with DaNN) that
    harmonizes bulk and scRNA-seq embeddings to predict per-cell drug response
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Backend for training the DAE encoder and DaNN predictor (CPU version used
    in the demo, ~20 minutes runtime)
- name: Scanpy
  role: Loads and preprocesses the GSE110894 scRNA-seq data and handles the AnnData
    output
- name: AnnData
  role: Output format (.h5ad) storing predictions in adata.obs['sens_label']
- name: conda-pack
  role: Distributes the reproducible scDEALenv environment (scdeal.tar.gz) for running
    the pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:b494f713b620d88f963ee9898e9aded30c81fd4e597fc54e6e3c34496c65ead1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - PyTorch
  - Scanpy
  - conda-pack
  - scDEAL
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# bulk-single-cell-rnaseq-integration

## Summary

Transfer a drug-response predictor trained on bulk RNA-seq (GDSC/CCLE-derived, e.g. ALL_expression.csv) to single-cell data via scDEAL's Domain-adaptive Neural Network (DaNN), so that I-BET.762 (or similar) sensitivity labels and DEG scores can be predicted per cell in scRNA-seq datasets such as GSE110894.

## When to use

Use when predicting single-cell drug sensitivity/resistance labels from scRNA-seq (h5ad/AnnData) by leveraging bulk-level drug-response training data, e.g. reproducing the GSE110894 I-BET case study with scDEAL.

## When NOT to use

- Input is already a per-cell drug response label table with no bulk training data to transfer from — no domain adaptation is possible
- The scRNA-seq species shares no homologous genes with the human-trained model (the article only kept homologous genes for mouse Data 6); trans-species reliability was not systematically evaluated
- The task is bulk-only drug response prediction without single-cell resolution — scDEAL's DaNN transfer step is unnecessary

## Inputs

- Bulk expression matrix (e.g. ALL_expression.csv) with GDSC/CCLE-derived drug response labels (ALL_label_binary_wf.csv)
- Single-cell scRNA-seq dataset (GSE110894, 1419 MA9 leukemic cells) as Scanpy/AnnData
- Pre-trained bulk/sc checkpoint .pkl (optional, e.g. from save/models)
- Ground-truth drug-screening label table for validation (sensitive/resistant cell groups)

## Outputs

- AnnData object (GSE110894[timestamp].h5ad) with sensitivity predictions in adata.obs['sens_label'] (0 = resistance, 1 = sensitivity)
- Trained bulk and single-cell model checkpoints in save/models
- Label match percentages, DEG-score correlations (R2), and permutation-test p-values

## How to apply

Train the bulk-level predictor with bulkmodel.py on ALL_expression.csv and ALL_label_binary_wf.csv using drug I.BET.762, DAE encoder, bottleneck 512, encoder_h_dims 256,128, predictor_h_dims 128,64, dropout 0.1, lr 0.5, and upsampling; the encoder/predictor dimensions must be identical between the bulk and single-cell steps. Then run scmodel.py on the GSE110894 scRNA-seq AnnData with matching hyperparameters, transferring the bulk model through the DaNN domain-adaptive network (optionally resuming from the checkpoint integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl). For mouse data (Data 6, Mus musculus), keep only homologous genes since the model is still trained with human cancer cell lines. Export predictions as an AnnData .h5ad with adata.obs['sens_label'] (0 = resistance, 1 = sensitivity). Correct application is judged by label match rates within ground-truth resistant/sensitive groups and by correlating predicted DEG/gene scores against ground-truth differential expression scores with a 1000-permutation null test.

## Related tools

- **scDEAL** (Deep transfer learning framework (bulkmodel.py + scmodel.py with DaNN) that harmonizes bulk and scRNA-seq embeddings to predict per-cell drug response) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Backend for training the DAE encoder and DaNN predictor (CPU version used in the demo, ~20 minutes runtime))
- **Scanpy** (Loads and preprocesses the GSE110894 scRNA-seq data and handles the AnnData output)
- **AnnData** (Output format (.h5ad) storing predictions in adata.obs['sens_label'])
- **conda-pack** (Distributes the reproducible scDEALenv environment (scdeal.tar.gz) for running the pipeline)

## Examples

```
python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --lr 0.5 --sampling "upsampling" -mod "new" --checkpoint "False"
```

## Evaluation signals

- Label match rates reproduce Fig. 3: 97.1% agreement within predicted drug-resistant cells and 95.8% within predicted drug-sensitive cells on GSE110894
- DEG-score correlations reproduce Fig. 3: R2 = 0.90 (sensitive) and 0.77 (resistant) between predicted and ground-truth differential expression scores
- Empirical permutation null test with n=1000 randomizations yields p < 0.001
- Bulk and single-cell runs use identical dimensions: bulk_h_dims/encoder_h_dims 256,128; predictor_h_dims 128,64; bottleneck 512
- Output h5ad exists with adata.obs['sens_label'] populated and encoded as 0 = resistance, 1 = sensitivity

## Limitations

- Prediction accuracy depends on the collection of bulk gene expression of cell lines; training data from additional bulk databases is planned but not integrated
- Trans-species (human-to-mouse) reliability could not be systematically evaluated due to limited drug-treated mouse benchmark scRNA-seq data
- Only homologous genes are kept in mouse data, discarding non-homologous genes; a potential sample swap between Data 1 and 2 (HN120 and HN137) was reported
- No software version numbers are stated for tools or the environment

## Evidence

- [abstract] We propose scDEAL, a deep transfer learning framework for cancer drug response prediction at the single-cell level by integrating large-scale bulk cell-line data.: "We propose scDEAL, a deep transfer learning framework for cancer drug response prediction at the single-cell level by integrating large-scale bulk cell-line data."
- [intro] scDEAL harmonizes single-cell and bulk embeddings to ensure drug response labels are transferable: "scDEAL harmonizes single-cell and bulk embeddings to ensure that the drug response labels are transferable from"
- [discussion] only homologous genes were kept in the mouse data and the scDEAL model was still trained with human cancer cell lines: "only homologous genes were kept in the mouse data and the scDEAL model was still trained with human cancer cell lines."
- [methods] The expected output format of scDEAL is the AnnData object (.h5ad): "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad)"
- [readme] scmodel.py checkpoint usage with GSE110894 and I.BET.762: "python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64""
