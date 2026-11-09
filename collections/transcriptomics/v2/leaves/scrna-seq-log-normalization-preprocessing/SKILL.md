---
name: scrna-seq-log-normalization-preprocessing
description: Use when raw SeqFISH/scRNA-seq count matrices must be converted to AnnData
  h5ad with truth locations in .obs[['x','y']] and log-normalized (scran) before contrastive
  model training or spatial reconstruction benchmarking.
when_to_use_negative:
- The input is already a log-normalized AnnData object compatible with the CellContrast
  train/eval pipeline — re-normalizing could distort the scran-scaled values used
  by the published benchmark.
- The data are sequencing-based spot-level ST (e.g. 10X Visium) rather than single-cell-resolution
  imaging data — different predefined parameters (parameters_spot.json vs parameters_singleCell.json)
  and considerations apply.
- The goal is deconvolution or gene imputation on ST spots rather than preparing expression
  profiles for SC spatial relationship reconstruction.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scran
  role: Performs the log-normalization of SeqFISH count matrices prior to contrastive
    training and evaluation
- name: CellContrast
  role: Consumes the log-normalized AnnData h5ad (with .obs[['x','y']]) as --train_data_path
    for contrastive model training
  repo: https://github.com/HKU-BAL/CellContrast
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:48ba7f04672941726c6d51dac64953ddaa79ad9f9e41103dec2b09f7862557ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CeLEry
  - CellContrast
  - CytoSpace
  - NovoSpaRc
  - SageNet
  - Scipy
  - Tangram
  - scran
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# scrna-seq-log-normalization-preprocessing

## Summary

Preprocess single-cell and spatial transcriptomics count matrices into log-normalized AnnData objects (scran-based log-normalization) so that CellContrast contrastive training and SC-to-ST spatial reconstruction operate on comparable expression scales.

## When to use

Use when raw SeqFISH/scRNA-seq count matrices must be converted to AnnData h5ad with truth locations in .obs[['x','y']] and log-normalized (scran) before contrastive model training or spatial reconstruction benchmarking.

## When NOT to use

- The input is already a log-normalized AnnData object compatible with the CellContrast train/eval pipeline — re-normalizing could distort the scran-scaled values used by the published benchmark.
- The data are sequencing-based spot-level ST (e.g. 10X Visium) rather than single-cell-resolution imaging data — different predefined parameters (parameters_spot.json vs parameters_singleCell.json) and considerations apply.
- The goal is deconvolution or gene imputation on ST spots rather than preparing expression profiles for SC spatial relationship reconstruction.

## Inputs

- Raw SeqFISH mouse organogenesis count matrices (imaging-based ST sections, e.g. embryo 1 L1/L2, embryo 2 L2, embryo 3 L2)
- Query single-cell RNA-seq count matrix for spatial reconstruction
- Truth spatial coordinates for ST reference sections

## Outputs

- Log-normalized AnnData .h5ad files with truth locations in .obs[['x','y']]
- Preprocessed training reference (embryo 1 L1, 10,150 cells, 351 genes) ready for CellContrast train

## How to apply

Convert the raw count matrices to AnnData (h5ad) format, with truth spatial coordinates stored in .obs[['x','y']] for the ST reference section — here embryo 1 L1 (10,150 cells, 351 genes) serves as training reference and embryo 1 L2, embryo 2 L2, embryo 3 L2 as test sections. Normalize the counts with scran and apply log transformation so query SC and reference ST expression profiles are on a comparable scale, which is required for the encoder-projector contrastive model to learn meaningful cosine similarities. Keep the gene panel consistent across reference and query (the SeqFISH panels are limited to 351 predefined genes, a known constraint of imaging-based ST). Only after this log-normalized preprocessing should the CellContrast train step be launched (e.g. with --single_cell and parameters_singleCell.json for imaging-based ST), since the infoNCE contrastive pairs are built from k=80 nearest spatial neighbors of the normalized profiles.

## Related tools

- **scran** (Performs the log-normalization of SeqFISH count matrices prior to contrastive training and evaluation)
- **CellContrast** (Consumes the log-normalized AnnData h5ad (with .obs[['x','y']]) as --train_data_path for contrastive model training) — https://github.com/HKU-BAL/CellContrast

## Examples

```
python cellContrast.py train --train_data_path train_ST.h5ad --save_folder cellContrast_models/ --single_cell --parameter_file parameters/parameters_singleCell.json
```

## Evaluation signals

- AnnData objects load without error and .obs contains 'x' and 'y' columns with finite coordinates for the ST reference section.
- Expression matrix is log-scale: value range is non-negative and small (typical log-normalized single-cell range, no raw integer counts of thousands).
- Gene panel matches the 351-gene SeqFISH set consistently across train and test sections.
- Downstream CellContrast training on the preprocessed reference reaches the reported benchmark behavior (median average hit ratio near 47.32% vs 36.22% SageNet), indicating normalization did not break the signal.
- Evaluation JSD and Spearman's rho on embryo 1 L2 fall near reported values (JSD ~0.237, rho ~0.874), consistent with correctly scaled inputs.

## Limitations

- Imaging-based ST methods such as SeqFISH+ achieve single-cell resolution but their gene-profiling capabilities are limited to a few hundred predefined genes, capping the information available to the reconstruction model.
- A certain level of mismatch may exist between the query SC data and the ST reference due to limited ST resources; large gaps between ST and SC samples lead to a performance drop in global reconstruction across all methods.
- CellContrast training data must be in AnnData format with truth locations in .obs[['x','y']]; the default parameters are defined for sequencing-based ST, so imaging-based data require the --single_cell flag and parameters_singleCell.json.

## Evidence

- [methods] AnnData objects with truth locations in .obs[['x','y']]: embryo 1 L1 (10,150 cells) for training and embryo 1 L2, embryo 2 L2, embryo 3 L2 (351 genes) for testing, log-normalized with scran: [evidence span withheld — anchor status: not_found]
- [intro] Imaging-based methods, such as SeqFISH+, STARmap, and MERFISH, have achieved single-cell resolution; however, their gene-profiling capabilities are limited to a few hundred predefined genes.: "Imaging-based methods, such as SeqFISH+,2 STARmap,3 and MERFISH,4 have achieved single-cell resolution"
- [readme] CellContrast model was trained based on ST data (which should be in AnnData format, with truth locations in .obs[['x','y']]).: "trained based on ST data (which should be in [AnnData](https://anndata.readthedocs.io/en/latest/) format, with truth locations in `.obs[['x','y']])`"
- [discussion] A certain level of mismatch may exist between the query SC data and the ST reference due to limited ST resources.: "A certain level of mismatch may exist between the query SC data and the ST reference due to limited ST resources"
