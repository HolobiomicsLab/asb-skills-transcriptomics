---
name: regression-model-training-on-multi-cell-type-genomic-data
description: Use when you have DNase-seq (or other genome-wide regulatory) measurements
  and matched gene expression matrices for tens of cell types and need cell-type-general
  predictive models of regulatory element activity.
when_to_use_negative:
- You need to discover new DHS locations in a new sample — BIRD will not discover
  new locations of DHSs, it only predicts within loci present in the model
- The response and predictors are low-dimensional — big data regression with cluster-level
  models is designed for ultra-high-dimensional predictors (transcriptome) and responses
  (regulome)
- Gene expression platform differs from all available training models and no matched
  training data exist to build a new model
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: BIRD
  role: Big data regression framework for training DH prediction models and predicting
    DNase-seq signal from gene expression
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD_predict
  role: Applies a trained .bin model to an input expression matrix, matching ensembl
    gene ids and bounding predictions to 0–14
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD_bash.sh
  role: Recommended bash wrapper for running BIRD prediction
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Required for loading BIRD-data .rds training matrices and running model-building
    scripts; Rscript must be executable
- name: GeneBASE
  role: Produces the exon array gene expression output files accepted by BIRD for
    training and prediction
  repo: https://github.com/WeiqiangZhou/BIRD/blob/master/exonarray_instruction.md
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:127a8557011b319761382fefff69a45207cb327a34803812a2c4b3537380fc6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD_bash.sh
  - BIRD_predict
  - GeneBASE
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# regression-model-training-on-multi-cell-type-genomic-data

## Summary

Train big-data regression models (BIRD) that predict genome-wide DNase I hypersensitivity (DH) from whole-transcriptome gene expression across many cell types, using ENCODE DNase-seq and exon array/RNA-seq training matrices and evaluating on held-out cell types via cross-locus (rL) and cross-cell-type (rC) correlations.

## When to use

Use when you have DNase-seq (or other genome-wide regulatory) measurements and matched gene expression matrices for tens of cell types and need cell-type-general predictive models of regulatory element activity.

## When NOT to use

- You need to discover new DHS locations in a new sample — BIRD will not discover new locations of DHSs, it only predicts within loci present in the model
- The response and predictors are low-dimensional — big data regression with cluster-level models is designed for ultra-high-dimensional predictors (transcriptome) and responses (regulome)
- Gene expression platform differs from all available training models and no matched training data exist to build a new model

## Inputs

- DNase-seq chromatin accessibility matrix (200bp windows, location in first three columns, one column per cell type) from BIRD-data, e.g. DNase_data_167_cells.rds
- Gene expression matrix (exon array or RNA-seq FPKM) with gene ensembl ids as the first column, one column per cell type
- Prebuilt or newly trained BIRD model file (.bin), e.g. human_hg19_model.bin or Exonarray_model_file.bin
- BIRD-Permute permuted model files for control predictions

## Outputs

- Trained locus-level and cluster-level BIRD prediction model files (.bin)
- Predicted DH matrix (log2(x+1) transformed, values bounded 0–14) per test cell type
- Matched input matrices (input_file.match.txt) after ensembl id matching
- Cross-locus rL and cross-cell-type rC metrics for BIRD and BIRD-Permute controls (e.g., recorded in performance_metrics.csv)

## How to apply

Assemble expression and DH matrices from the BIRD-data repository (e.g., DNase-seq and exon array data for 57 ENCODE samples or RNA-seq data for 167 ENCODE samples, hg19). For each DHS cluster, build locus-level and cluster-level prediction models following the build_prediction_model.md protocol, applying the robust quantile normalization (which significantly boosts performance when the input contains many tied values, e.g., single-cell RNA-seq) and bounding predicted values from 0 to 14 (upper bound adjustable via the -u option; use -l to fall back to the locus-level model if the cluster-level model is uncertain). Train on a subset of cell types (e.g., 40 ENCODE training cell types), then run BIRD_predict on held-out test cell types (e.g., 17 test cell types), letting BIRD match ensembl gene ids between the model and the input matrix (matched data output as input_file.match.txt). Evaluate with cross-locus correlation rL and cross-cell-type correlation rC against measured DH, and repeat with BIRD-Permute permuted models to obtain control baselines (expected test-set means: BIRD rL=0.82, rC=0.50; permuted controls rL=0.65, rC=-0.03).

## Related tools

- **BIRD** (Big data regression framework for training DH prediction models and predicting DNase-seq signal from gene expression) — https://github.com/WeiqiangZhou/BIRD
- **BIRD_predict** (Applies a trained .bin model to an input expression matrix, matching ensembl gene ids and bounding predictions to 0–14) — https://github.com/WeiqiangZhou/BIRD
- **BIRD_bash.sh** (Recommended bash wrapper for running BIRD prediction) — https://github.com/WeiqiangZhou/BIRD
- **R** (Required for loading BIRD-data .rds training matrices and running model-building scripts; Rscript must be executable)
- **GeneBASE** (Produces the exon array gene expression output files accepted by BIRD for training and prediction) — https://github.com/WeiqiangZhou/BIRD/blob/master/exonarray_instruction.md

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg19_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- Held-out test cell types should reach mean cross-locus rL ≈ 0.82 and cross-cell-type rC ≈ 0.50 for the ENCODE 40-train/17-test split
- BIRD-Permute controls should yield rL ≈ 0.65 (cell-type-independent DH propensity baseline) and rC ≈ -0.03; rC well above the permuted control indicates real cross-cell-type signal
- All predicted values must lie within the 0–14 bound (log2(x+1) scale)
- The matched input file input_file.match.txt should contain only ensembl ids shared with the model, with version suffixes stripped by default
- Cluster-level model predictions should be compared against the -l locus-level model output when cluster model reliability is uncertain

## Limitations

- BIRD will not discover new locations of DHSs when applied to a new gene expression sample
- Locus-specific DH propensity is partly cell-type-independent, so high cross-locus rL alone (as with permuted controls reaching 0.65) does not demonstrate cell-type-specific prediction quality
- Cluster-level models may not always work; use the -l option to fall back to the locus-level model when unsure
- Extending BIRD to other gene expression platforms remains future work; models are genome-build specific (hg19/hg38) and platform-specific (exon array vs RNA-seq)

## Evidence

- [methods] The quantile normalization function is updated to a more robust version: "The quantile normalization function is updated to a more robust version"
- [readme] The predicted values are now bounded from 0 to 14: "The predicted values are now bounded from 0 to 14. Users can use the -u option to change the upper bound"
- [readme] BIRD will first match the ensembl id: "BIRD will first match the ensembl id of the genes used for building the prediction model with the ids in the input data matrix"
- [discussion] one limitation of BIRD: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
- [readme] locus-level model fallback: "Users can also use -l option to perform prediction using the locus-level model rather than using the full model"
- [intro] BIRD handles ultra-high-dimensional prediction: "we develop a big data regression approach, BIRD (Fig. 1b), to handle the prediction problem where both predictors (i.e., transcriptome) and responses (i.e., regulome) are ultra-high-dimensional"
