---
name: deep-transfer-learning-model-training
description: Use when predicting single-cell drug responses by transferring a bulk-trained
  model (e.g., I.BET.762 on GSE110894) and you need bulk+scRNA-seq model training
  with matching hyperparameters.
when_to_use_negative:
- Input is bulk data alone with no single-cell dataset to transfer to — bulkmodel.py
  alone does not produce single-cell predictions
- The scRNA-seq dataset is from a species other than human without restricting to
  homologous genes — cross-species transfer (human-trained to mouse data) was only
  tested by keeping homologous genes and its reliability could not be systematically
  evaluated
- No bulk-level drug response data exists for the drug of interest, since prediction
  accuracy depends on the collection of bulk gene expression of cell lines
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: scDEAL
  role: Deep transfer learning framework adapting a DaNN to predict single-cell drug
    responses from bulk and scRNA-seq data
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Deep-learning backend used to train bulk and single-cell level models (CPU
    version used for the demo)
- name: Scanpy
  role: Loads and preprocesses the scRNA-seq input data for prediction
- name: AnnData
  role: Output format (.h5ad) storing predicted sensitivity labels in adata.obs['sens_label']
- name: conda-pack
  role: Distributes the prebuilt scDEAL conda environment (scdeal.tar.gz) for running
    the training scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:a873ac4c8c8ef581a003f062ad15543d7731bf4ed6e47139fe36ec56cafb90b0
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# deep-transfer-learning-model-training

## Summary

Train scDEAL's bulk-level drug-response predictor (bulkmodel.py) and transfer it via a DaNN domain-adaptive network to single-cell data (scmodel.py), enabling drug-sensitivity prediction in scRNA-seq using large-scale bulk GDSC/CCLE training data.

## When to use

Use when predicting single-cell drug responses by transferring a bulk-trained model (e.g., I.BET.762 on GSE110894) and you need bulk+scRNA-seq model training with matching hyperparameters.

## When NOT to use

- Input is bulk data alone with no single-cell dataset to transfer to — bulkmodel.py alone does not produce single-cell predictions
- The scRNA-seq dataset is from a species other than human without restricting to homologous genes — cross-species transfer (human-trained to mouse data) was only tested by keeping homologous genes and its reliability could not be systematically evaluated
- No bulk-level drug response data exists for the drug of interest, since prediction accuracy depends on the collection of bulk gene expression of cell lines

## Inputs

- Bulk-level drug response RNA-seq expression matrix from GDSC/CCLE (e.g., data/ALL_expression.csv)
- Bulk drug-screening label table (e.g., data/ALL_label_binary_wf.csv)
- scRNA-seq dataset loaded with Scanpy/AnnData (e.g., GSE110894, 1419 MA9 leukemic cells)
- Pretrained model checkpoints (.pkl) from save/models, optional

## Outputs

- Trained bulk model checkpoint (save/bulk_pre/...)
- Single-cell DaNN model checkpoint (save/sc_pre/..._DaNN.pkl)
- AnnData object (.h5ad, e.g., GSE110894[timestamp].h5ad) with adata.obs['sens_label'] (0=resistance, 1=sensitivity)

## How to apply

First train the bulk-level predictor with bulkmodel.py (e.g., --drug 'I.BET.762', --dimreduce 'DAE', --encoder_h_dims '256,128', --predictor_h_dims '128,64', --bottleneck 512, --dropout 0.1, --lr 0.5, --sampling 'upsampling', --mod 'new'); for GSE110894 these are the suggested parameters. Then run scmodel.py on the scRNA-seq dataset (e.g., --sc_data 'GSE110894') with identical bulk_h_dims, bottleneck, predictor_h_dims, dropout, lr, and sampling settings so the DaNN can transfer the bulk model to the single-cell domain. Keep encoder/predictor dimensions identical between the two steps. You may resume from provided checkpoints via the --checkpoint option (e.g., save/sc_pre/..._DaNN.pkl) or train from scratch with --checkpoint 'False'. The output is an AnnData object with sensitivity labels stored in adata.obs['sens_label'] (0 = resistance, 1 = sensitivity). Runtime for the demo test case was about 20 minutes on the pytorch CPU version.

## Related tools

- **scDEAL** (Deep transfer learning framework adapting a DaNN to predict single-cell drug responses from bulk and scRNA-seq data) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Deep-learning backend used to train bulk and single-cell level models (CPU version used for the demo))
- **Scanpy** (Loads and preprocesses the scRNA-seq input data for prediction)
- **AnnData** (Output format (.h5ad) storing predicted sensitivity labels in adata.obs['sens_label'])
- **conda-pack** (Distributes the prebuilt scDEAL conda environment (scdeal.tar.gz) for running the training scripts)

## Examples

```
python bulkmodel.py --drug "I.BET.762" --dimreduce "DAE" --encoder_h_dims "256,128" --predictor_h_dims "128,64" --bottleneck 512 --data_name "GSE110894" --sampling "upsampling" --dropout 0.1 --lr 0.5 --printgene "F" -mod "new" --checkpoint "False"; python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --lr 0.5 --sampling "upsampling" --checkpoint "False"
```

## Evaluation signals

- On Data 6 (GSE110894, I-BET), predictions should match original labels for ~97.1% of predicted resistant and ~95.8% of predicted sensitive cells
- Predicted DEG/gene scores should correlate with ground-truth differential expression scores at R2 = 0.90 (sensitive) and 0.77 (resistant)
- Empirical permutation null test with 1000 randomizations should yield p < 0.001
- Encoder (--bulk_h_dims/--encoder_h_dims) and predictor dimension arguments must be identical between bulkmodel.py and scmodel.py runs
- Output .h5ad must contain adata.obs['sens_label'] with values 0 (resistance) and 1 (sensitivity)

## Limitations

- Prediction accuracy could vary depending on the collection of bulk gene expression of cell lines; additional bulk-level databases may improve training
- Trans-species (human-to-mouse) reliability could not be systematically evaluated due to limited drug-treated mouse benchmark scRNA-seq data
- More experimentally validated drug response scRNA-seq data would be needed to determine better model hyperparameters; no software version numbers are stated
- Training time is ~20 minutes for the demo test case on the pytorch CPU version

## Evidence

- [intro] We develop scDEAL (single-cell Drug rEsponse AnaLysis) by adapting a Domain-adaptive Neural Network (DaNN)12 to predict drug responses from bulk and scRNA-seq data.: "we develop scDEAL (single-cell Drug rEsponse AnaLysis) by adapting a Domain-adaptive Neural Network (DaNN)12 to predict drug responses from bulk and scRNA-seq data"
- [readme] Dimension consistency requirement: "Remember that the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64", --bottleneck 256) in two steps."
- [readme] Checkpoints naming: "integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl"
- [methods] Runtime: "The training time of the test case including bulk-level and single-cell-level training on the testing computer was 20 minutes on the pytorch cpu version"
- [methods] Output format: "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad) applied by the scanpy package"
