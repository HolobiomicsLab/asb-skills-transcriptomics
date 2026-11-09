---
name: transcription-factor-network-analysis
description: Use when a transcriptomics (or phospho-proteomic) dataset must be summarized
  into per-regulator activity scores and the user runs multiple inference methods
  simultaneously and needs one consensus estimate per regulator.
when_to_use_negative:
- When only a single inference method is run — no aggregation across methods is needed
  and the consensus mean z-score does not apply
- When the input is raw count data without a prior knowledge network of TF-target
  interactions; decoupleR's methods require signed/weighted regulatory interactions
- When the goal is pathway/gene-set enrichment scoring rather than regulator-level
  activity inference
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: decoupleR
  role: Core Bioconductor/Python package that runs the ensemble of activity-inference
    methods and provides the 'decouple' wrapper used to compute the consensus score
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: R package that runs the perturbation benchmark pipeline over decoupleR methods,
    concatenates experiments, and computes AUROC/AUPRC with downsampling
  repo: https://github.com/saezlab/decoupleRBench
- name: OmniPath
  role: Meta-resource of prior knowledge (over 100 databases) supplying the TF-target
    interactions used as input to the inference methods
- name: R
  role: Language in which the benchmark pipeline (run_rna_bench.R) and consensus aggregation
    are executed
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: Python
  role: Alternative implementation of decoupleR (decoupler-py) available on GitHub
    for Python users
  repo: https://github.com/saezlab/decoupler-py
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:6f0cb220e7eb7df2ef8e94faa202e3d38d1abb9dbf66c6d6540827fa6fa82104
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OmniPath
  - Python
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# transcription-factor-network-analysis

## Summary

Infers transcription factor (regulator) activities from transcriptomics data by running multiple decoupleR methods on a prior knowledge network, then aggregating per-method activity estimates into a single consensus score computed as the mean z-score across methods.

## When to use

Use when a transcriptomics (or phospho-proteomic) dataset must be summarized into per-regulator activity scores and the user runs multiple inference methods simultaneously and needs one consensus estimate per regulator.

## When NOT to use

- When only a single inference method is run — no aggregation across methods is needed and the consensus mean z-score does not apply
- When the input is raw count data without a prior knowledge network of TF-target interactions; decoupleR's methods require signed/weighted regulatory interactions
- When the goal is pathway/gene-set enrichment scoring rather than regulator-level activity inference

## Inputs

- Transcriptomics perturbation benchmark data (DoRothEA-based count table plus metadata) from Zenodo record 5645208
- Prior knowledge resource (e.g. OmniPath meta-resource of over 100 databases) providing signed and weighted TF-target interactions
- Per-method regulator activity score matrices from the decoupleRBench pipeline

## Outputs

- consensus_scores.csv containing one consensus activity estimate per regulator with row identifiers matching the input activity matrices
- Per-method activity estimates for each regulator and experiment
- AUROC and AUPRC evaluation of sensitivity in recovering perturbed TFs for the consensus and each individual method

## How to apply

Retrieve the DoRothEA-based transcriptomics perturbation benchmark data (Zenodo record 5645208) and run the decoupleRBench pipeline (Rscript R/process/run_rna_bench.R), which builds on decoupleR's 'decouple' wrapper to infer activities for each regulator and experiment. For each method's activity score matrix, standardize the activity estimates to z-scores per regulator. Compute the consensus score as the mean z-score across the methods, as specified in the decoupleR Implementation section and Supplementary Note. Evaluate recovery of the perturbed transcription factors by concatenating all experiments into a response vector (regulator perturbed or not) and a predictor vector (regulator activities), then compute AUROC and AUPRC across thresholds using downsampling of negative classes. The article reports that simple linear models (ULM, MLM) and the consensus score rank among the top methods at predicting perturbed regulators.

## Related tools

- **decoupleR** (Core Bioconductor/Python package that runs the ensemble of activity-inference methods and provides the 'decouple' wrapper used to compute the consensus score) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (R package that runs the perturbation benchmark pipeline over decoupleR methods, concatenates experiments, and computes AUROC/AUPRC with downsampling) — https://github.com/saezlab/decoupleRBench
- **OmniPath** (Meta-resource of prior knowledge (over 100 databases) supplying the TF-target interactions used as input to the inference methods)
- **R** (Language in which the benchmark pipeline (run_rna_bench.R) and consensus aggregation are executed) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **Python** (Alternative implementation of decoupleR (decoupler-py) available on GitHub for Python users) — https://github.com/saezlab/decoupler-py

## Examples

```
Rscript R/process/run_rna_bench.R
```

## Evaluation signals

- The consensus method must appear among the top-ranked methods at predicting perturbed regulators on the transcriptomics benchmark — the article reports the top three as consensus, multivariate linear model, and ULM
- consensus_scores.csv must contain one consensus activity estimate per regulator, with row identifiers matching those of the input per-method activity matrices
- Per-method activity matrices must be z-scored per regulator before averaging; verify standardized distributions per regulator
- AUROC/AUPRC must be computed against a response vector marking whether a regulator is perturbed, with negative classes downsampled per permutation to match the limited true-positive class size

## Limitations

- True positive classes are limited to regulators covered by the perturbation experiments, requiring a downsampling strategy for evaluation
- The consensus score's rationale and details are deferred to the Supplementary Note rather than fully specified in the main text
- Some analyses (e.g. noise analysis, ~6 hours for run_rna_noise.R) are computationally expensive

## Evidence

- [abstract] Simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators.: [evidence span withheld — anchor status: not_found]
- [supplementary] Consensus is mean z-score across methods.: [evidence span withheld — anchor status: not_found]
- [readme] Evaluation uses AUROC/AUPRC with downsampling of negatives.: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmark pipeline is a separate R package decoupleRBench.: "In this manuscript we have built a separate R package to benchmark decoupleR, called decoupleRBench."
- [abstract] decoupleR is a Bioconductor package with multiple statistical methods in a unified framework.: "we present decoupleR, a Bioconductor package containing different statistical methods to extract these signatures within a unified framework"
