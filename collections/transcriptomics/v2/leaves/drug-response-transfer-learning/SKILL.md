---
name: drug-response-transfer-learning
description: Use when you must predict single-cell drug sensitivity/resistance labels
  and signature genes for a drug-treated scRNA-seq dataset (e.g. Cisplatin GSE117872,
  Gefitinib GSE112274, I-BET-762 GSE110894) using bulk cell-line training data.
when_to_use_negative:
- Your query data is bulk RNA-seq drug-response prediction, not single-cell — the
  skill's DaNN alignment and clustering regularization target scRNA-seq heterogeneity
- 'You need systematic trans-species evaluation: only homologous genes were kept in
  the mouse Data 6, and the article notes trans-species reliability could not be systematically
  evaluated'
- You lack the matching bulk drug-screening training data for the drug of interest,
  since prediction accuracy varies with the collection of bulk cell-line expression
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scDEAL
  role: 'Core framework: DaNN-based bulk-to-single-cell drug-response transfer, prediction,
    and integrated gradient interpretation'
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Backend used to train bulkmodel.py and scmodel.py; demo runs ~20 minutes on
    the pytorch cpu version
- name: Scanpy
  role: Processes the output AnnData objects (.h5ad) containing sens_label predictions
- name: AnnData
  role: Output container format (.h5ad) for single-cell predictions and labels
- name: conda-pack
  role: Distributes the prepacked scDEAL conda environment (scdeal.tar.gz) for reproducible
    execution
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:fed16d1c0332762fb7c8fe1ca3c32acb4bfafb1da3018d2ffa38da7ae11bb36d
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47368421052631576
n_claims: 19
---

# drug-response-transfer-learning

## Summary

scDEAL is a deep transfer learning framework that predicts single-cell drug response labels (0 = resistance, 1 = sensitivity) by training on bulk GDSC/CCLE cell-line RNA-seq (ALL_expression.csv + ALL_label_binary_wf.csv) and transferring the model to drug-treated scRNA-seq data via DaNN-based MMD alignment. It also derives drug-response signature genes through integrated gradient (IG) interpretation.

## When to use

Use when you must predict single-cell drug sensitivity/resistance labels and signature genes for a drug-treated scRNA-seq dataset (e.g. Cisplatin GSE117872, Gefitinib GSE112274, I-BET-762 GSE110894) using bulk cell-line training data.

## When NOT to use

- Your query data is bulk RNA-seq drug-response prediction, not single-cell — the skill's DaNN alignment and clustering regularization target scRNA-seq heterogeneity
- You need systematic trans-species evaluation: only homologous genes were kept in the mouse Data 6, and the article notes trans-species reliability could not be systematically evaluated
- You lack the matching bulk drug-screening training data for the drug of interest, since prediction accuracy varies with the collection of bulk cell-line expression

## Inputs

- Bulk GDSC/CCLE drug-response training files: ALL_expression.csv and ALL_label_binary_wf.csv
- Drug-treated scRNA-seq dataset (e.g. GSE117872 HN120/HN137, GSE112274, GSE140440, GSE149383, GSE110894)
- Drug name (e.g. 'I.BET.762', 'CISPLATIN')
- Optional pre-trained checkpoint .pkl from save/sc_pre or save/bulk_pre

## Outputs

- AnnData (.h5ad) object with adata.obs['sens_label'] (0 = resistance, 1 = sensitivity) saved under save/adata/data
- Trained bulk and single-cell model checkpoints (.pkl) under save/models
- Integrated gradient critical-gene lists for sensitive and resistant cells when --printgene 'T'
- Benchmark metrics: F1, AUROC, AP, precision, recall, AMI, ARI

## How to apply

Clone the OSU-BMBL/scDEAL repository, download scDEAL.zip into the root 'data' directory (containing ALL_expression.csv, ALL_label_binary_wf.csv and the six GEO datasets), and activate the conda-pack environment from scdeal.tar.gz. Train the bulk-level model from scratch with bulkmodel.py using per-dataset hyperparameters from the README table (e.g. GSE110894/I-BET.762: bottleneck 512, encoder_h_dims 256,128, predictor_h_dims 128,64, DAE, dropout 0.1, lr 0.5, upsampling; GSE117872HN120/CISPLATIN: dropout 0.3, lr 0.01, SMOTE). Then run scmodel.py with identical architecture dimensions to transfer the bulk model to the scRNA-seq data using DaNN-based MMD alignment (mmd_weight 0.25, mmd_GAMMA 1000, Leiden resolution 0.3, mod new); set --printgene 'T' to also output integrated gradient critical-gene scores. The encoder/predictor dimensions must be identical across the bulk and single-cell steps. Results are stored as AnnData (.h5ad) objects with adata.obs['sens_label'] under save/adata/data, and metrics (F1, AUROC, AP, precision, recall, AMI, ARI) are computed against reported sensitive/resistant cell labels.

## Related tools

- **scDEAL** (Core framework: DaNN-based bulk-to-single-cell drug-response transfer, prediction, and integrated gradient interpretation) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Backend used to train bulkmodel.py and scmodel.py; demo runs ~20 minutes on the pytorch cpu version)
- **Scanpy** (Processes the output AnnData objects (.h5ad) containing sens_label predictions)
- **AnnData** (Output container format (.h5ad) for single-cell predictions and labels)
- **conda-pack** (Distributes the prepacked scDEAL conda environment (scdeal.tar.gz) for reproducible execution)

## Examples

```
python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --lr 0.5 --sampling "upsampling" --printgene "F" -mod "new" --checkpoint "False"
```

## Evaluation signals

- Output .h5ad exists with adata.obs['sens_label'] encoding 0 = resistance and 1 = sensitivity
- Predicted labels reproduce reported benchmark values, e.g. Data 1 F1 0.92 / AUROC 0.92 / AP 0.97, and averages of F1 0.892, AUROC 0.898, AP 0.944, precision 0.926, recall 0.899, AMI 0.528, ARI 0.608
- Encoder/predictor dimensions (--encoder_h_dims/--bulk_h_dims, --predictor_h_dims, --bottleneck) are identical between bulkmodel.py and scmodel.py runs
- Checkpoint filename follows the documented convention, e.g. integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl
- With --printgene 'T', critical-gene lists (936 sensitive, 868 resistant in Data 1) reproduce GO terms such as 'DNA repair' (adj. p = 0.039) and 'cell division' (adj. p = 0.003)

## Limitations

- Prediction accuracy could vary depending on the collection of bulk gene expression of cell lines; training data will be updated by integrating additional bulk-level databases
- A potential sample swap between Data 1 and 2 (HN120 and HN137) was pointed out by a recent study
- Trans-species (human-to-mouse) reliability could not be systematically evaluated due to limited drug-treated mouse benchmark scRNA-seq data; more experimentally validated data would be needed to determine better hyperparameters

## Evidence

- [abstract] scDEAL abstract claim: "We propose scDEAL, a deep transfer learning framework for cancer drug response prediction at the single-cell level by integrating large-scale bulk cell-line data."
- [readme] Identical dimensions across steps: "Remember that the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64", --bottleneck 256) in two steps."
- [readme] Demo prediction behavior: "This step calls the scDEAL model and predicts the sensitivity of I.BET.762 of the input scRNA-Seq data from GSE110984."
- [methods] Output format: "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad) applied by the scanpy package"
- [discussion] Accuracy depends on bulk training data: "The accuracy of the prediction results in scDEAL could vary, depending on the collection of bulk gene expression of cell lines."
- [discussion] Potential sample swap caveat: "Noted that a recent study pointed out a potential sample swap between Data 1 and 2 (HN120 and HN137)49."
