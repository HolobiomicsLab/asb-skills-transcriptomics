---
name: transcription-factor-activity-inference
description: Use when you need to score TF activities from RNA perturbation compendia
  (e.g. Holland et al. 2020 transcriptomics data) using a prior-knowledge gene regulatory
  network like DoRothEA and rank inference methods by sensitivity.
when_to_use_negative:
- The goal is phospho-proteomic perturbation benchmarking of kinase activities rather
  than TF activity from RNA — run the phospho benchmark (run_php_bench.R) instead.
- You only need enrichment scores from single-cell or bulk omics data without a perturbation
  benchmark — the Python scverse/decoupler package (pip install decoupler) is the
  maintained implementation, not the decoupleRBench pipeline.
- Input data lacks a gene regulatory network covering the perturbed regulators — true
  positive classes are limited by regulators covered in the perturbation experiments,
  so uncovered TFs cannot be evaluated.
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: decoupleR
  role: Computes TF activity scores from the perturbation compendium using the DoRothEA
    network via the decouple wrapper function, providing 11 statistical methods including
    AUCell, fast GSEA, GSVA, ORA, ULM, VIPER and consensus.
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: R package that evaluates performance of biological activity inference methods
    using perturbation experiments, concatenating experiments into response and predictor
    vectors and computing AUROC/AUPRC with a downsampling strategy.
  repo: https://github.com/saezlab/decoupleRBench
- name: OmniPath
  role: Meta-resource of prior knowledge networks leveraged by decoupleR as the source
    of TF-target interactions.
- name: R
  role: Language in which the benchmark pipeline (get_data.R, run_rna_bench.R) is
    implemented and executed.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:bb73292c439941042521954fe2e50c8842bf962198906ebb30c77f006858fa3d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Flame
  - OmniPath
  - R
  - SCALA
  - decoupleR
  - decoupleRBench
  - g:Profiler
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- transcription-factor-activity-inference-decoupler
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# transcription-factor-activity-inference

## Summary

Infer transcription factor activities from transcriptomics perturbation data by running decoupleR's 11 statistical methods (AUCell, fast GSEA, GSVA, ORA, ULM, VIPER, mlm, consensus, etc.) against the DoRothEA gene regulatory network, then benchmark which methods best rank the experimentally perturbed TF among top regulators.

## When to use

Use when you need to score TF activities from RNA perturbation compendia (e.g. Holland et al. 2020 transcriptomics data) using a prior-knowledge gene regulatory network like DoRothEA and rank inference methods by sensitivity.

## When NOT to use

- The goal is phospho-proteomic perturbation benchmarking of kinase activities rather than TF activity from RNA — run the phospho benchmark (run_php_bench.R) instead.
- You only need enrichment scores from single-cell or bulk omics data without a perturbation benchmark — the Python scverse/decoupler package (pip install decoupler) is the maintained implementation, not the decoupleRBench pipeline.
- Input data lacks a gene regulatory network covering the perturbed regulators — true positive classes are limited by regulators covered in the perturbation experiments, so uncovered TFs cannot be evaluated.

## Inputs

- Transcriptomics perturbation count table (Holland et al. 2020 collection from Zenodo record 5645208)
- Corresponding metadata table specifying perturbed regulators per experiment
- DoRothEA gene regulatory network (prior-knowledge TF-target interactions with sign and weight)
- Input tibble with user-specified decoupleR method settings for decoupleRBench

## Outputs

- rna_bench_performance.csv: per-method performance table matching Supplementary Table S3
- Performance figure matching Supplementary Fig. S3 / Fig. 1C
- Per-method AUROC and AUPRC values over downsampled response/predictor vectors
- TF activity scores per regulator and experiment for each of the 11 decoupleR methods

## How to apply

Install decoupleRBench from GitHub with devtools::install_github('saezlab/decoupleRBench'), which builds on decoupleR's decouple wrapper function. Retrieve the benchmark data by running Rscript R/process/get_data.R, pulling the Holland et al. 2020 transcriptomics perturbation collection from Zenodo record 5645208. Run the RNA perturbation benchmark with Rscript R/process/run_rna_bench.R, applying the 11 decoupleR methods (AUCell, fast GSEA, GSVA, ORA, ULM, VIPER, mlm, consensus, etc.) to compute TF activity scores from the perturbation compendium using the DoRothEA network. Score each method's sensitivity by its ability to rank the experimentally perturbed TF among top regulators, then aggregate per-method scores into a performance table (rna_bench_performance.csv) and figure matching Supplementary Table S3 and Fig. S3. Evaluate methods with AUROC and AUPRC using decoupleRBench's downsampling strategy: experiments are concatenated into a response vector (perturbed or not) and a predictor vector (regulator activities), with equal numbers of negative classes randomly sampled per permutation because true positive classes are limited by regulators covered in the perturbation experiments. Validate that the top three ranked methods are consensus, multivariate linear model and ULM, as reported in the paper.

## Related tools

- **decoupleR** (Computes TF activity scores from the perturbation compendium using the DoRothEA network via the decouple wrapper function, providing 11 statistical methods including AUCell, fast GSEA, GSVA, ORA, ULM, VIPER and consensus.) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (R package that evaluates performance of biological activity inference methods using perturbation experiments, concatenating experiments into response and predictor vectors and computing AUROC/AUPRC with a downsampling strategy.) — https://github.com/saezlab/decoupleRBench
- **OmniPath** (Meta-resource of prior knowledge networks leveraged by decoupleR as the source of TF-target interactions.)
- **R** (Language in which the benchmark pipeline (get_data.R, run_rna_bench.R) is implemented and executed.)

## Examples

```
Rscript R/process/run_rna_bench.R
```

## Evaluation signals

- The top three ranked methods in rna_bench_performance.csv are consensus, multivariate linear model and ULM, matching Supplementary Table S3.
- The generated performance figure matches Supplementary Fig. S3 / Fig. 1C of the manuscript.
- AUROC and AUPRC are computed for every method after downsampling an equal number of negative classes per permutation.
- The benchmark reproduces the paper's finding that simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators.
- Output table and figure are reproducible end-to-end from Zenodo record 5645208 via the get_data.R / run_rna_bench.R scripts in the manuscript repository.

## Limitations

- True positive classes are limited by the regulators covered in the perturbation experiments, requiring a downsampling strategy with equal numbers of randomly sampled negative classes per permutation.
- The RNA noise analysis is computationally expensive, taking roughly 6 hours (run_rna_noise.R); the phospho noise analysis takes ~2 hours.
- decoupleRBench requires the decoupleR package to be installed and assumes familiarity with the basics of decoupleR; the R decoupler version is deprecated in favor of the Python scverse implementation.

## Evidence

- [results] Top methods ranking: [evidence span withheld — anchor status: not_found]
- [abstract] Methods evaluated: "Using decoupleR, we evaluated the performance of methods on transcriptomic and phospho-proteomic perturbation experiments."
- [readme] Downsampling strategy: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmark installation: "To install `deocupleRBench` please run:
```
devtools::install_github('saezlab/decoupleRBench')
```"
- [readme] Data retrieval: "To retrieve all the necessary data, please run:
```
Rscript R/process/get_data.R
```"
- [readme] Building on decoupleR: [evidence span withheld — anchor status: external_doc]
- [readme] AUROC and AUPRC evaluation: [evidence span withheld — anchor status: external_doc]
- [abstract] Prior knowledge leverage: "it leverages OmniPath, a meta-resource comprising over 100 databases of prior knowledge"
