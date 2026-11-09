---
name: bulk-single-cell-gene-set-overlap-analysis
description: Use when scDEAL IG matrices exist at both bulk and single-cell levels
  and you need to quantify how many top contributing genes are shared between the
  two scales for sensitivity vs. resistance.
when_to_use_negative:
- The goal is drug-response label prediction only (--printgene "F") rather than critical-gene
  identification and overlap quantification
- Data are not drug-treated scRNA-seq datasets paired with a bulk GDSC/CCLE-trained
  scDEAL model (e.g. no matching checkpoint for the dataset/drug combination)
- You lack the matching checkpoint files in save/models and cannot train bulkmodel.py/scmodel.py
  with identical --encoder_h_dims/--predictor_h_dims settings
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scDEAL
  role: Framework whose bulkmodel.py and scmodel.py with --printgene "T" produce integrated-gradient
    critical gene lists at bulk and single-cell levels
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Backend on which the DaNN models and integrated-gradient interpretation are
    computed
- name: Scanpy
  role: Handles the AnnData (.h5ad) single-cell outputs that carry per-cell predictions
    used in the case-study analysis
- name: AnnData
  role: Expected output object format (.h5ad) of scDEAL from which critical-gene results
    are analyzed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:781c47216b958ae29126e45076649be6efb47e2334b7a1c443e91f18935a6e98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - PyTorch
  - Scanpy
  - scDEAL
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# bulk-single-cell-gene-set-overlap-analysis

## Summary

Compares integrated-gradient (IG) contributing gene lists from scDEAL's bulk-level model (bulkmodel.py) and single-cell-level model (scmodel.py), computing the fraction of shared genes for drug-sensitivity and drug-resistance polarities (reported means: 46% and 53%, Supplementary Table S6).

## When to use

Use when scDEAL IG matrices exist at both bulk and single-cell levels and you need to quantify how many top contributing genes are shared between the two scales for sensitivity vs. resistance.

## When NOT to use

- The goal is drug-response label prediction only (--printgene "F") rather than critical-gene identification and overlap quantification
- Data are not drug-treated scRNA-seq datasets paired with a bulk GDSC/CCLE-trained scDEAL model (e.g. no matching checkpoint for the dataset/drug combination)
- You lack the matching checkpoint files in save/models and cannot train bulkmodel.py/scmodel.py with identical --encoder_h_dims/--predictor_h_dims settings

## Inputs

- Bulk RNA-seq drug-response data from scDEAL.zip (data/ALL_expression.csv, data/ALL_label_binary_wf.csv)
- Single-cell datasets GSE110894, GSE112274, GSE117872, GSE140440, GSE149383 (data/ subdirectories)
- Pre-trained model checkpoints in save/models (e.g. integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl)
- casestudy/analysis_criticalgenes.ipynb notebook template

## Outputs

- Ranked top contributing gene lists for sensitivity and resistance at the bulk level and at the single-cell level
- Integrated gradient matrices from the printgene="T" runs
- Per-dataset, per-polarity bulk-vs-single-cell overlap fractions (~46% sensitivity, ~53% resistance, Supplementary Table S6)
- critical_genes_overlap.tsv with one overlap row per dataset per polarity

## How to apply

Load the scDEAL.zip datasets into data/ and locate pre-trained checkpoints in save/models. Run bulkmodel.py with --printgene "T" (e.g. --drug "I.BET.762", --dimreduce "DAE", --encoder_h_dims "256,128", --predictor_h_dims "128,64", --bottleneck 512, --dropout 0.1, --lr 0.5, --sampling "upsampling") to emit bulk-level critical genes via integrated gradients. Run scmodel.py with --printgene "T" and the matching --checkpoint file for each benchmark dataset (GSE110894, GSE112274, GSE117872, GSE140440, GSE149383), keeping encoder/predictor dimensions identical across both steps. Use the casestudy/analysis_criticalgenes.ipynb template to extract the integrated-gradient matrices and rank top contributing genes for sensitivity and resistance at both levels. For each dataset and each polarity, compute the fraction of shared genes between the bulk-level and single-cell-level critical gene lists.

## Related tools

- **scDEAL** (Framework whose bulkmodel.py and scmodel.py with --printgene "T" produce integrated-gradient critical gene lists at bulk and single-cell levels) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Backend on which the DaNN models and integrated-gradient interpretation are computed)
- **Scanpy** (Handles the AnnData (.h5ad) single-cell outputs that carry per-cell predictions used in the case-study analysis)
- **AnnData** (Expected output object format (.h5ad) of scDEAL from which critical-gene results are analyzed)

## Examples

```
python bulkmodel.py --drug "I.BET.762" --dimreduce "DAE" --encoder_h_dims "256,128" --predictor_h_dims "128,64" --bottleneck 512 --dropout 0.1 --lr 0.5 --sampling "upsampling" --printgene "T"
```

## Evaluation signals

- Computed bulk-vs-single-cell overlap fractions approximate the reported averages of 46% (drug sensitivity) and 53% (drug resistance), Supplementary Table S6
- critical_genes_overlap.tsv exists and contains exactly one overlap row per dataset per polarity
- Critical genes are extracted only when --printgene is set to "T"; output gene lists should be absent/empty for the "F" demo runs
- bulkmodel.py and scmodel.py were run with identical dimensions (--encoder_h_dims/bulk_h_dims "256,128"-type values, --predictor_h_dims, --bottleneck) so the IG attributions are comparable across levels
- IG gene rankings are reproducible from the matching checkpoint naming pattern (bottle_, edim_, pdim_, dropout_, lr_, sam_ fields consistent with the parameters used)

## Limitations

- Prediction accuracy and downstream gene attribution could vary depending on the collection of bulk gene expression of cell lines used for training
- A potential sample swap between Data 1 and 2 (HN120 and HN137, GSE117872) was pointed out by a recent study, affecting that dataset's overlap result
- No software version numbers are stated for tools or the environment, complicating exact reproduction of IG gene lists

## Evidence

- [abstract] Another feature of scDEAL is the integrated gradient feature interpretation to infer the signature genes of drug resistance mechanisms.: "Another feature of scDEAL is the integrated gradient feature interpretation to infer the signature genes of drug resistance mechanisms."
- [intro] scDEAL's integrated gradient interpretation infers the signature genes of drug response predictions, which improves the interpretability of the model.: [evidence span withheld — anchor status: not_found]
- [readme] For the scRNA-Seq prediction task, we provide pre-trained checkpoints for the models stored in save/models: "For the scRNA-Seq prediction task, we provide pre-trained checkpoints for the models stored in save/models"
- [readme] Remember that the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64", --bottleneck 256) in two steps.: "Remember that the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64", --bottleneck 256) in two steps."
- [methods] The expected output format of scDEAL is the AnnData object (.h5ad) applied by the scanpy package: "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad) applied by the scanpy package"
