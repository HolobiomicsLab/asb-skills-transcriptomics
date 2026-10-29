---
name: z-score-normalization-across-methods
description: Use when decoupleR has been run with multiple methods simultaneously
  and their per-regulator activity estimates must be aggregated into one consensus
  score, or when benchmarking whether the consensus recovers perturbed regulators
  better than individual methods.
when_to_use_negative:
- Only a single decoupleR method was run — there is nothing to aggregate across methods,
  so use the method's raw activity scores directly
- Input is a raw count table or perturbation metadata rather than per-method regulator
  activity estimates — these are upstream decoupleRBench inputs, not the z-score aggregation
  step
- The goal is comparing methods on prior-knowledge resources without a perturbation
  benchmark — consensus ranking evaluation requires perturbation ground truth (perturbed
  vs non-perturbed regulators)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: decoupleR
  role: Provides the unified framework whose 'decouple' wrapper infers per-method
    regulator activities that are z-scored and averaged into the consensus score
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: Benchmark package that runs decoupleR methods on perturbation experiments
    and concatenates activity estimates into response/predictor vectors for AUROC/AUPRC
    evaluation of the consensus
  repo: https://github.com/saezlab/decoupleRBench
- name: R
  role: Language in which the benchmark pipeline (run_rna_bench.R) and z-score/mean
    consensus computation are executed
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:655477db81da0b3cee123a475e61b1986886c7117074778b2be622fb8427a4b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# z-score-normalization-across-methods

## Summary

Computes the decoupleR consensus score by standardizing each method's regulator activity estimates to per-regulator z-scores and averaging them across methods, aggregating multi-method output into a single ranked consensus activity per regulator.

## When to use

Use when decoupleR has been run with multiple methods simultaneously and their per-regulator activity estimates must be aggregated into one consensus score, or when benchmarking whether the consensus recovers perturbed regulators better than individual methods.

## When NOT to use

- Only a single decoupleR method was run — there is nothing to aggregate across methods, so use the method's raw activity scores directly
- Input is a raw count table or perturbation metadata rather than per-method regulator activity estimates — these are upstream decoupleRBench inputs, not the z-score aggregation step
- The goal is comparing methods on prior-knowledge resources without a perturbation benchmark — consensus ranking evaluation requires perturbation ground truth (perturbed vs non-perturbed regulators)

## Inputs

- Per-method regulator activity score matrices produced by the decoupleRBench pipeline (transcriptomics benchmark)
- DoRothEA-based TF activity estimates from single-gene perturbation experiments (Zenodo record 5645208)
- Benchmark count table and corresponding metadata table (input tibble for decoupleRBench)

## Outputs

- consensus_scores.csv with one consensus activity estimate per regulator, row identifiers matching the input activity matrices
- Per-regulator z-scored activity matrices (one per method)
- Method/consensus sensitivity ranking (AUROC and AUPRC with downsampled negatives)

## How to apply

Retrieve the transcriptomics perturbation benchmark data (DoRothEA-based TF activity estimates from single-gene perturbation experiments) from Zenodo record 5645208, then run the decoupleRBench pipeline via 'Rscript R/process/run_rna_bench.R' to obtain per-method activity score matrices. Standardize each method's activity estimates to z-scores per regulator so methods with different score scales become comparable. Compute the consensus score as the mean z-score across methods, as specified in the decoupleR Implementation section and Supplementary Note. Evaluate recovery of perturbed TFs (sensitivity, AUROC/AUPRC with the downsampling strategy for limited positive classes) for the consensus and each method, including the ULM and multivariate linear model baselines; the paper reports the top three as consensus, multivariate linear model and ULM.

## Related tools

- **decoupleR** (Provides the unified framework whose 'decouple' wrapper infers per-method regulator activities that are z-scored and averaged into the consensus score) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (Benchmark package that runs decoupleR methods on perturbation experiments and concatenates activity estimates into response/predictor vectors for AUROC/AUPRC evaluation of the consensus) — https://github.com/saezlab/decoupleRBench
- **R** (Language in which the benchmark pipeline (run_rna_bench.R) and z-score/mean consensus computation are executed) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html

## Examples

```
Rscript R/process/run_rna_bench.R
```

## Evaluation signals

- consensus_scores.csv contains exactly one consensus activity estimate per regulator, with row identifiers matching the input per-method activity matrices
- On the transcriptomics benchmark, the consensus method ranks among the top methods at predicting perturbed regulators — reported top three are consensus, multivariate linear model and ULM
- Each method's estimates are z-scored per regulator before averaging; consensus values are means of z-scores across the methods run simultaneously
- AUROC/AUPRC evaluation uses the downsampling strategy with an equal number of randomly sampled negatives per permutation, since true positive classes are limited to perturbation-covered regulators

## Limitations

- True positive classes in the benchmark are limited by the regulators covered in the perturbation experiments, requiring downsampling of negatives
- The consensus averages whatever methods are run simultaneously, so its quality depends on which decoupleR methods are included
- The RNA noise analysis takes about 6 hours and the phospho-proteomic noise analysis about 2 hours, so full benchmark reproduction is time-consuming

## Evidence

- [abstract] Simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators.: "Our findings suggest that simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators."
- [abstract] Zenodo data record for benchmark inputs: "and the data in Zenodo (https://zenodo.org/record/5645208)"
- [readme] Benchmark evaluation concatenates experiments into response/predictor vectors: [evidence span withheld — anchor status: external_doc]
- [readme] decoupleRBench inputs are a count table plus metadata: [evidence span withheld — anchor status: external_doc]
