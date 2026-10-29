---
name: critical-gene-identification-from-trained-networks
description: Use when a trained deep drug-response predictor must be interpreted —
  i.e., you need ranked lists of genes contributing to sensitivity vs resistance predictions
  and want to measure bulk-vs-single-cell overlap (reported ~46% / 53%).
when_to_use_negative:
- The model has not yet been trained — IG attribution requires a converged bulk and
  single-cell checkpoint with matching encoder/predictor dimensions
- You only need drug-response labels per cell rather than gene-level mechanistic interpretation
  (run scmodel.py with --printgene "F" instead)
- Input scRNA-seq lacks homologous gene mapping to the human bulk training space (e.g.
  non-human species without homolog filtering)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scDEAL
  role: Deep transfer learning framework whose trained bulkmodel.py and scmodel.py
    networks are interpreted with --printgene "T" to emit integrated-gradient critical
    genes
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Backend on which the DaNN models and integrated-gradient attribution run (CPU
    version)
- name: Scanpy
  role: Used with AnnData .h5ad outputs to handle single-cell results during critical-gene
    analysis
- name: AnnData
  role: Expected output object format (.h5ad) holding per-cell predictions and IG
    gene contributions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:8f4301b5faa43c8dd69b2bc69fd6eefe0194e499902b323559ec8da4cda345af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - PyTorch
  - Scanpy
  - scDEAL
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# critical-gene-identification-from-trained-networks

## Summary

Extract critical genes driving drug-response predictions from trained scDEAL networks using integrated-gradient (IG) feature attribution at both the bulk (GDSC/CCLE-trained bulkmodel) and single-cell (scmodel) levels, then compare the top-contributing sensitivity/resistance gene sets across the two levels.

## When to use

Use when a trained deep drug-response predictor must be interpreted — i.e., you need ranked lists of genes contributing to sensitivity vs resistance predictions and want to measure bulk-vs-single-cell overlap (reported ~46% / 53%).

## When NOT to use

- The model has not yet been trained — IG attribution requires a converged bulk and single-cell checkpoint with matching encoder/predictor dimensions
- You only need drug-response labels per cell rather than gene-level mechanistic interpretation (run scmodel.py with --printgene "F" instead)
- Input scRNA-seq lacks homologous gene mapping to the human bulk training space (e.g. non-human species without homolog filtering)

## Inputs

- Pre-trained bulk checkpoint (save/bulk_pre/..., .pkl without DaNN suffix)
- Pre-trained single-cell checkpoint (save/sc_pre/..., .pkl with DaNN suffix)
- Bulk drug-response expression matrices (data/ALL_expression.csv, data/ALL_label_binary_wf.csv)
- Single-cell datasets extracted from scDEAL.zip into data/ (GSE110894, GSE112274, GSE117872, GSE140440, GSE149383)
- casestudy/analysis_criticalgenes.ipynb analysis template

## Outputs

- Bulk-level integrated-gradient critical gene lists ranked for sensitivity and resistance
- Single-cell-level integrated-gradient critical gene lists per dataset and polarity
- Per-dataset bulk-vs-single-cell overlap fractions (expected ~46% sensitivity, ~53% resistance)
- critical_genes_overlap.tsv with one overlap row per dataset per polarity
- AnnData outputs (.h5ad) in save/adata

## How to apply

Locate the pre-trained checkpoints in save/models (naming pattern: integrate_data_<scGSE>_drug_<drug>_bottle_<bottleneck>_edim_<encoder_dims>_pdim_<predictor_dims>_model_<encoder>_dropout_<dropout>_gene_<T/F>_lr_<lr>_mod_<version>_sam_<sampling>). Run bulkmodel.py with --printgene "T" to emit the bulk-level IG critical gene list, then run scmodel.py with --printgene "T" and the matching --checkpoint for each benchmark dataset (GSE110894, GSE112274, GSE117872, GSE140440, GSE149383). Use the casestudy/analysis_criticalgenes.ipynb template to extract the integrated-gradient matrices and rank the top genes separately for sensitivity and resistance polarity. For each dataset and polarity, compute the fraction of genes shared between the bulk-level and single-cell-level critical gene lists; expect on average ~46% overlap for drug sensitivity and ~53% for drug resistance (Supplementary Table S6). Keep encoder/predictor dimensions identical between the bulk and single-cell runs (e.g. encoder_h_dims "256,128", predictor_h_dims "128,64", bottleneck 512) so the attribution is computed on the same feature space.

## Related tools

- **scDEAL** (Deep transfer learning framework whose trained bulkmodel.py and scmodel.py networks are interpreted with --printgene "T" to emit integrated-gradient critical genes) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Backend on which the DaNN models and integrated-gradient attribution run (CPU version))
- **Scanpy** (Used with AnnData .h5ad outputs to handle single-cell results during critical-gene analysis)
- **AnnData** (Expected output object format (.h5ad) holding per-cell predictions and IG gene contributions)

## Examples

```
python bulkmodel.py --drug "I.BET.762" --dimreduce "DAE" --encoder_h_dims "256,128" --predictor_h_dims "128,64" --bottleneck 512 --data_name "GSE110894" --sampling "upsampling" --dropout 0.1 --lr 0.5 --printgene "T" -mod "new" --checkpoint "save/bulk_pre/integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling"
```

## Evaluation signals

- Bulk-vs-single-cell overlap fractions approximate 46% (drug sensitivity) and 53% (drug resistance) on average, matching Supplementary Table S6
- critical_genes_overlap.tsv contains exactly one overlap row per dataset per polarity (5 datasets × 2 polarities)
- Encoder and predictor dimensions are identical between bulkmodel.py and scmodel.py runs (e.g. edim "256,128", pdim "128,64", bottleneck 512)
- Checkpoints used carry gene_T in the filename, confirming --printgene "T" was applied
- IG matrices loaded in analysis_criticalgenes.ipynb yield non-empty ranked gene lists for both sensitivity and resistance

## Limitations

- Prediction accuracy and the resulting critical genes depend on the collection of bulk gene-expression profiles of cell lines used in training (GDSC/CCLE); scDEAL's training data will be updated with additional bulk databases
- A recent study pointed out a potential sample swap between Data 1 and 2 (HN120 and HN137), which affects downstream resistance-gene interpretation for those datasets
- No software version numbers are stated for tools or the environment, so reproduction requires the provided scdeal.tar.gz conda-pack environment

## Evidence

- [abstract] Integrated gradient feature interpretation to infer signature genes of drug resistance mechanisms: "Another feature of scDEAL is the integrated gradient feature interpretation to infer the signature genes of drug resistance mechanisms."
- [readme] Checkpoint naming with gene flag: "integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl"
- [readme] Pre-trained checkpoints location: "For the scRNA-Seq prediction task, we provide pre-trained checkpoints for the models stored in save/models"
- [readme] Dimension consistency requirement: "Remember that the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64", --bottleneck 256) in two steps."
- [intro] IG interpretability objective: [evidence span withheld — anchor status: not_found]
