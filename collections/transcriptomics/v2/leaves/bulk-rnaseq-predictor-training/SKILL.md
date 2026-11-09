---
name: bulk-rnaseq-predictor-training
description: Use when bulk RNA-seq expression with binary drug response annotations
  is available and a drug sensitivity predictor must be trained (or resumed from a
  checkpoint) before transfer to scRNA-seq data.
when_to_use_negative:
- The goal is already single-cell prediction with a bulk-trained model — use scmodel.py
  (the transfer step) instead; bulkmodel.py only trains on bulk cell-line data
- No bulk-level drug response labels or bulk expression matrix are available — the
  predictor cannot be trained without ALL_expression.csv and ALL_label_binary_wf.csv
- You need drug response prediction from bulk-only data without any scRNA-seq transfer
  — scDEAL is designed to augment scRNA-seq analysis with bulk data
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scDEAL
  role: Framework providing bulkmodel.py, the DAE-based bulk-level training script
    whose checkpoints feed single-cell transfer via scmodel.py
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Backend for the CPU-based bulk model training (demo bulk+single-cell training
    takes ~20 minutes on CPU)
- name: conda-pack
  role: Distributes the prepacked scDEALenv environment (scdeal.tar.gz) in which bulkmodel.py
    runs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:9155a97d296afe80089a7fe2df559852dab779b3a39a7d2f2c29cce83181b3da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - Anaconda
  - PyTorch
  - Scanpy
  - conda
  - conda-pack
  - pip
  - scDEAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bulk-rnaseq-predictor-training

## Summary

Train a bulk-level drug sensitivity predictor by fitting a denoising autoencoder (DAE) encoder plus predictor head on bulk RNA-seq expression (ALL_expression.csv) with binary drug-response labels (ALL_label_binary_wf.csv), producing checkpoints that scDEAL later transfers to single-cell data.

## When to use

Use when bulk RNA-seq expression with binary drug response annotations is available and a drug sensitivity predictor must be trained (or resumed from a checkpoint) before transfer to scRNA-seq data.

## When NOT to use

- The goal is already single-cell prediction with a bulk-trained model — use scmodel.py (the transfer step) instead; bulkmodel.py only trains on bulk cell-line data
- No bulk-level drug response labels or bulk expression matrix are available — the predictor cannot be trained without ALL_expression.csv and ALL_label_binary_wf.csv
- You need drug response prediction from bulk-only data without any scRNA-seq transfer — scDEAL is designed to augment scRNA-seq analysis with bulk data

## Inputs

- Bulk RNA-seq drug-response expression matrix ALL_expression.csv (from scDEAL.zip data directory)
- Binary bulk drug-response labels ALL_label_binary_wf.csv
- Drug name (e.g. I.BET.762, CISPLATIN, GEFITINIB, DOCETAXEL, ERLOTINIB)
- Optional pretrained bulk checkpoint .pkl from save/bulk_pre/

## Outputs

- Trained bulk predictor checkpoint (e.g. save/bulk_pre/integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling)
- Log/status files under save/logs/

## How to apply

Run bulkmodel.py inside the prepacked scDEALenv conda environment (unpacked from scdeal.tar.gz via conda-pack) with --dimreduce "DAE" and --bulk integrate, so the bulk encoder embeds cell-line expression prior to predictor training. Set dataset-specific hyperparameters from the README's suggested-parameter table, e.g. GSE110894/I-BET.762: bottleneck 512, encoder_h_dims "256,128", predictor_h_dims "128,64", dropout 0.1, lr 0.5, sampling "upsampling"; GSE112274/GEFITINIB: bottleneck 256, encoder_h_dims "512,256", sampling "no"; GSE117872HN120: dropout 0.3, lr 0.01, sampling "SMOTE". Choose a sampling strategy (upsampling, downsampling, or SMOTE) to balance resistant/sensitive classes, and choose either a pretrained checkpoint via --checkpoint <path> or --checkpoint "False" to train from scratch. Critically, the encoder (--encoder_h_dims/--bulk_h_dims), predictor (--predictor_h_dims), and bottleneck dimensions must be identical in the bulk step and the later scmodel.py transfer step so weights load correctly. The trained bulk model is written under save/bulk_pre/ with a name encoding data, drug, and all hyperparameters.

## Related tools

- **scDEAL** (Framework providing bulkmodel.py, the DAE-based bulk-level training script whose checkpoints feed single-cell transfer via scmodel.py) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Backend for the CPU-based bulk model training (demo bulk+single-cell training takes ~20 minutes on CPU))
- **conda-pack** (Distributes the prepacked scDEALenv environment (scdeal.tar.gz) in which bulkmodel.py runs)

## Examples

```
python bulkmodel.py --drug "I.BET.762" --dimreduce "DAE" --encoder_h_dims "256,128" --predictor_h_dims "128,64" --bottleneck 512 --data_name "GSE110894" --sampling "upsampling" --dropout 0.1 --lr 0.5 --checkpoint "False"
```

## Evaluation signals

- Checkpoint filename under save/bulk_pre/ encodes the exact hyperparameters used (data, drug, bottle, edim, pdim, dropout, lr, mod, sam)
- Bulk and single-cell runs use identical --encoder_h_dims/bulk_h_dims, --predictor_h_dims, and --bottleneck values, or the downstream scmodel.py checkpoint load fails
- Training completes within ~20 minutes on the PyTorch CPU version for the GSE110894 test case (bulk + single-cell levels)
- Downstream benchmark: the transferred model should reproduce reported averages of F1 0.892, AUROC 0.898, AP 0.944, precision 0.926, recall 0.899 across the six datasets
- For GSE110894 demo, expected per-dataset metrics are F1 0.92 / AUROC 0.92 / AP 0.97

## Limitations

- Prediction accuracy varies depending on the collection of bulk gene expression of cell lines; scDEAL's training data may need to be expanded by integrating additional bulk-level databases
- No software version numbers are stated for the tools or the conda environment, complicating exact reproducibility
- Hyperparameter optimization (e.g. grid-search) is needed per dataset; more experimentally validated drug response scRNA-seq data would be required to determine better model hyperparameters

## Evidence

- [methods] Bulk-level training with bulkmodel.py and dataset-specific sampling: "Samping method of training data for the bulk model traning. Can be upsampling, downsampling, or SMOTE. default: no"
- [readme] Encoder/predictor dimensions must match across bulk and single-cell steps: "Remember that the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64""
- [readme] Training from scratch uses --checkpoint "False": "We can also train bulkmode.py and scmodel.py from scratch with user-defined parameters by setting --checkpoint "False":"
- [intro] Bulk data comes from GDSC/CCLE-scale drug response RNA-seq: "it can use a large amount of bulk-level drug response RNA-seq information from the Genomics of Drug Sensitivity in Cancer (GDSC) database13,14 and Cancer Cell Line Encyclopedia (CCLE)15"
- [methods] Environment setup via conda-pack: "tar -xzf scDEAL.tar.gz -C scDEALenv
# Activate the environment. This adds `scDEALenv/bin` to your path
source scDEALenv/bin/activate"
