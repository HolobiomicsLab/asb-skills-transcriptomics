---
name: benchmark-sensitivity-evaluation
description: Use when you need to benchmark TF activity inference methods against
  perturbation experiments (e.g. Holland et al. 2020 transcriptomics) and rank methods
  like ULM, MLM, VIPER, AUCell by sensitivity.
when_to_use_negative:
- When the goal is inferring activities on a single new dataset rather than comparing
  method performance across a perturbation benchmark collection
- When perturbation ground truth (which regulator was perturbed per experiment) is
  unavailable, since the response vector depends on perturbation labels
- When analyzing phospho-proteomic perturbations — that requires the analogous run_php_bench.R
  pipeline rather than the RNA benchmark
edam_operation: http://edamontology.org/operation_3988
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: decoupleRBench
  role: 'R package that implements the benchmark pipeline: evaluates performance of
    biological activity inference methods using perturbation experiments, building
    on decoupleR''s ''decouple'' wrapper'
  repo: https://github.com/saezlab/decoupleRBench
- name: decoupleR
  role: Computes TF activity scores via 11 statistical methods (AUCell, fast GSEA,
    GSVA, ORA, ULM, VIPER, etc.) within a unified framework
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: OmniPath
  role: Meta-resource of prior knowledge networks leveraged by decoupleR (DoRothEA
    network context)
- name: decoupleR_manuscript
  role: Reproduction repository providing get_data.R and run_rna_bench.R scripts for
    the RNA perturbation benchmark
  repo: https://github.com/saezlab/decoupleR_manuscript
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:53bc909b36abf47198e70f3f2b74cd941349d8dddd23fe50880126a27b9c0296
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/decoupleRBench
  tools:
  - OmniPath
  - R
  - decoupleR
  - decoupleRBench
  - decoupleR_manuscript
  edam_operation: http://edamontology.org/operation_3988
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# benchmark-sensitivity-evaluation

## Summary

Evaluates regulator-inference methods by their sensitivity to rank experimentally perturbed transcription factors among top regulators, aggregating per-method scores into a performance table/figure. Implements the decoupleRBench pipeline used in the decoupleR manuscript to benchmark 11 decoupleR methods on transcriptomics perturbation data.

## When to use

Use when you need to benchmark TF activity inference methods against perturbation experiments (e.g. Holland et al. 2020 transcriptomics) and rank methods like ULM, MLM, VIPER, AUCell by sensitivity.

## When NOT to use

- When the goal is inferring activities on a single new dataset rather than comparing method performance across a perturbation benchmark collection
- When perturbation ground truth (which regulator was perturbed per experiment) is unavailable, since the response vector depends on perturbation labels
- When analyzing phospho-proteomic perturbations — that requires the analogous run_php_bench.R pipeline rather than the RNA benchmark

## Inputs

- Benchmark data as a count table (Holland et al. 2020 transcriptomics perturbation compendium from Zenodo record 5645208)
- Corresponding metadata table for the perturbation experiments
- Input tibble with user-specified benchmark settings
- DoRothEA gene regulatory network (prior knowledge resource with signed/weighted interactions)

## Outputs

- Per-method sensitivity scores ranking the experimentally perturbed TF among top regulators
- AUROC and AUPRC values per method (with downsampling over negative classes)
- rna_bench_performance.csv performance table matching Supplementary Table S3
- Performance figure matching Supplementary Fig. S3

## How to apply

Install decoupleRBench via devtools::install_github('saezlab/decoupleRBench'), which builds on the decoupleR 'decouple' wrapper function and requires decoupleR to be installed. Retrieve the Holland et al. 2020 transcriptomics perturbation collection from Zenodo record 5645208 by running Rscript R/process/get_data.R. Run the RNA perturbation benchmark with Rscript R/process/run_rna_bench.R, applying the 11 decoupleR methods (AUCell, fast GSEA, GSVA, ORA, ULM, VIPER, etc.) to compute TF activity scores from the perturbation compendium using the DoRothEA gene regulatory network. For each method, activities are inferred for each regulator and experiment, then all experiments are concatenated into a response vector (whether a regulator is perturbed) and a predictor vector (regulator activities); AUROC and AUPRC are computed at different thresholds with a downsampling strategy that randomly samples an equal number of negative classes per permutation. Score each method's sensitivity by its ability to rank the experimentally perturbed TF among top regulators, and aggregate scores into rna_bench_performance.csv matching Supplementary Table S3.

## Related tools

- **decoupleRBench** (R package that implements the benchmark pipeline: evaluates performance of biological activity inference methods using perturbation experiments, building on decoupleR's 'decouple' wrapper) — https://github.com/saezlab/decoupleRBench
- **decoupleR** (Computes TF activity scores via 11 statistical methods (AUCell, fast GSEA, GSVA, ORA, ULM, VIPER, etc.) within a unified framework) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **OmniPath** (Meta-resource of prior knowledge networks leveraged by decoupleR (DoRothEA network context))
- **decoupleR_manuscript** (Reproduction repository providing get_data.R and run_rna_bench.R scripts for the RNA perturbation benchmark) — https://github.com/saezlab/decoupleR_manuscript

## Examples

```
devtools::install_github('saezlab/decoupleRBench'); Rscript R/process/get_data.R; Rscript R/process/run_rna_bench.R
```

## Evaluation signals

- Top three ranked methods in rna_bench_performance.csv are consensus, multivariate linear model and ULM, matching the published ranking (Supplementary Table S3; Fig. 1C)
- Output performance table matches Supplementary Table S3 and the figure matches Supplementary Fig. S3
- AUROC/AUPRC are computed with the documented downsampling strategy (equal number of randomly sampled negative classes per permutation)
- All 11 decoupleR methods ran successfully on the DoRothEA network over the full perturbation compendium
- Per-method scores derive from concatenated experiments forming proper response (perturbed/not) and predictor (activity) vectors

## Limitations

- True positive classes are limited by the regulators covered in the perturbation experiments, requiring the downsampling strategy for negatives
- The noise analysis is computationally heavy: run_rna_noise.R takes ~6 hours
- decoupleRBench requires the decoupleR package to be installed and familiarity with the 'decouple' wrapper basics

## Evidence

- [results] sensitivity by ability to rank perturbed TF: [evidence span withheld — anchor status: not_found]
- [readme] README evaluation strategy: [evidence span withheld — anchor status: external_doc]
- [readme] README benchmark construction: [evidence span withheld — anchor status: external_doc]
- [abstract] abstract finding: "Our findings suggest that simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators."
- [other] task workflow commands: "Rscript R/process/get_data.R
Rscript R/process/run_rna_bench.R"
