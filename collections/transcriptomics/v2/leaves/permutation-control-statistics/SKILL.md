---
name: permutation-control-statistics
description: Use when evaluating whether BIRD-predicted DNase I hypersensitivity correlations
  in held-out cell types reflect real expression–accessibility coupling, by benchmarking
  prediction-truth rL and rC against permuted-model controls.
when_to_use_negative:
- When the analysis goal is within-sample cross-locus accuracy only — permuted controls
  still achieve high rL (~0.65) because locus-specific DH propensity is cell-type-independent,
  so the control does not challenge cross-locus performance
- When applying BIRD to discover new DHS locations — the article notes one limitation
  of BIRD is that it will not discover new locations of DHSs in a new expression sample,
  so permutation-based evaluation of novel loci is out of scope
- When no trained/permuted model pair exists for the expression platform being used
  (e.g. an expression platform BIRD has not been extended to)
tools:
- name: BIRD (BIRD-Permute)
  role: Runs the permuted-control prediction models to obtain null rL and rC correlations
    for held-out test cell types
  repo: github.com/WeiqiangZhou/BIRD
- name: R
  role: Required runtime for BIRD's matching/normalization scripts and for computing
    rL/rC correlations and writing performance_metrics.csv
  repo: github.com/WeiqiangZhou/BIRD
- name: BIRD_predict
  role: Executes predictions with both real and permuted model files, matching ensembl
    gene ids between model and input matrix
  repo: github.com/WeiqiangZhou/BIRD
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:ddbc4cc719a14d6b0375ff95d1492e7059ef26b1ead3b33893282f3242606711
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BIRD
  - BIRD (BIRD-Permute)
  - BIRD_bash.sh
  - BIRD_predict
  - GeneBASE
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Permutation Control Statistics

## Summary

Establishes a null baseline for BIRD DNase I hypersensitivity (DH) prediction by re-running prediction with permuted models (BIRD-Permute), quantifying how much of the observed cross-locus (rL) and cross-cell-type (rC) correlations is attributable to cell-type-independent locus-specific DH propensity rather than transcriptome information.

## When to use

Use when evaluating whether BIRD-predicted DNase I hypersensitivity correlations in held-out cell types reflect real expression–accessibility coupling, by benchmarking prediction-truth rL and rC against permuted-model controls.

## When NOT to use

- When the analysis goal is within-sample cross-locus accuracy only — permuted controls still achieve high rL (~0.65) because locus-specific DH propensity is cell-type-independent, so the control does not challenge cross-locus performance
- When applying BIRD to discover new DHS locations — the article notes one limitation of BIRD is that it will not discover new locations of DHSs in a new expression sample, so permutation-based evaluation of novel loci is out of scope
- When no trained/permuted model pair exists for the expression platform being used (e.g. an expression platform BIRD has not been extended to)

## Inputs

- Trained BIRD prediction models (e.g. Exonarray_model_file.bin or human_hg19_model.bin from WeiqiangZhou/BIRD-model)
- Permuted control models (BIRD-Permute) built with shuffled gene–DHS mappings
- Gene expression matrix (ensembl ids as first column) for the 17 held-out ENCODE test cell types
- Measured DNase-seq DH values (200bp windows, hg19) for the same test cell types

## Outputs

- Control cross-locus correlation rL from BIRD-Permute (expected mean ~0.65)
- Control cross-cell-type correlation rC from BIRD-Permute (expected mean ~-0.03)
- performance_metrics.csv with real-model rL=0.82, rC=0.50 and permuted-control rL=0.65, rC=-0.03

## How to apply

After computing cross-locus (rL) and cross-cell-type (rC) correlations between BIRD-predicted and measured DH for the 17 held-out ENCODE test cell types, repeat the prediction using permuted control models (BIRD-Permute) in which the gene–regulatory element correspondence is destroyed. Record the control correlations alongside the real-model values in performance_metrics.csv. Interpretation hinges on the asymmetry between the two metrics: permuted models retain large mean rL (~0.65, because locus-specific DH propensity is cell-type-independent), but their mean rC drops to about -0.03, so the permutation control validates the cross-cell-type signal (BIRD rC = 0.50) far more sharply than the cross-locus signal (BIRD rL = 0.82). Report all four values (0.82, 0.50, 0.65, -0.03) to make clear that rL alone is not evidence of expression-informed prediction.

## Related tools

- **BIRD (BIRD-Permute)** (Runs the permuted-control prediction models to obtain null rL and rC correlations for held-out test cell types) — github.com/WeiqiangZhou/BIRD
- **R** (Required runtime for BIRD's matching/normalization scripts and for computing rL/rC correlations and writing performance_metrics.csv) — github.com/WeiqiangZhou/BIRD
- **BIRD_predict** (Executes predictions with both real and permuted model files, matching ensembl gene ids between model and input matrix) — github.com/WeiqiangZhou/BIRD

## Evaluation signals

- Permuted-control mean rC must be near zero (recorded as -0.03), demonstrating the control abolishes cross-cell-type signal
- Permuted-control mean rL remains large (~0.65), which should be reported as a caveat, not treated as real prediction performance
- Real-model values must reach mean rL = 0.82 and mean rC = 0.50 on the 17 test cell types, with the rC gap (0.50 vs -0.03) constituting the strongest evidence of expression-informed prediction
- All four values (rL, rC for BIRD and BIRD-Permute) must be present in performance_metrics.csv
- Predicted DH values are bounded from 0 to 14 (default upper bound via the -u option), matching the training-data scale

## Limitations

- Permuted models still reach a large mean rL of ~0.65, so cross-locus correlation alone cannot demonstrate transcriptome-based prediction; only the cross-cell-type rC separates real from permuted models
- BIRD will not discover new DHS locations in a new gene expression sample, so permutation evaluation is restricted to the ~1M training loci
- Open question: whether gene expression can be similarly used to predict other functional genomic data types remains unanswered

## Evidence

- [methods] permuted controls at rL = 0.65 and rC = -0.03: [evidence span withheld — anchor status: not_found]
- [methods] Repeat prediction with BIRD-Permute: [evidence span withheld — anchor status: not_found]
- [results] permuted-control rL driven by locus propensity: [evidence span withheld — anchor status: not_found]
- [readme] predicted values bounded 0 to 14: "The predicted values are now bounded from 0 to 14. Users can use the -u option to change the upper bound when using their own prediction model."
