---
name: perturbation-benchmark-pipeline-execution
description: Use when the goal is to reproduce or re-run a method-ranking benchmark
  that scores how well decoupleR methods recover experimentally perturbed TFs/kinases
  from perturbation experiments, or to run the associated network-noise robustness
  analysis.
when_to_use_negative:
- Input is a single omics dataset where you only want TF/kinase activity scores rather
  than a comparative benchmark of inference methods — use decoupleR/decoupler directly
  instead of decoupleRBench
- The prior knowledge resource is not a signed, weighted network compatible with decoupleR's
  mode-of-regulation methods (e.g. unweighted gene sets without TF/regulator mapping)
- The task is differential expression or enrichment analysis on a single contrast
  rather than evaluating regulator-inference performance against known perturbation
  labels
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3525
tools:
- name: decoupleR
  role: Core method suite whose 'decouple' wrapper infers TF/kinase activity scores
    with 11 methods plus consensus, used inside the benchmark
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: Dedicated benchmarking package that concatenates experiments, builds response/predictor
    vectors, and computes AUROC/AUPRC with negative-class downsampling
  repo: https://github.com/saezlab/decoupleRBench
- name: OmniPath
  role: Meta-resource of prior knowledge networks leveraged by decoupleR as benchmark
    input resources
  repo: https://github.com/saezlab/decoupler-py
- name: R
  role: Execution environment; all benchmark, noise and figure scripts are Rscript
    calls
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:7976d36a6eaf1c3e6bf0ae50e3663c964b8c47355b8715fe966026e89c86f1c0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OmniPath
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3525
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# perturbation-benchmark-pipeline-execution

## Summary

Executes the decoupleR manuscript's perturbation benchmarks, running 11 decoupleR methods (AUCell, fast GSEA, GSVA, ORA, ULM, VIPER, etc.) plus the consensus score over transcriptomics and phospho-proteomics perturbation compendia against DoRothEA and kinase substrate networks to rank regulator-inference methods by sensitivity.

## When to use

Use when the goal is to reproduce or re-run a method-ranking benchmark that scores how well decoupleR methods recover experimentally perturbed TFs/kinases from perturbation experiments, or to run the associated network-noise robustness analysis.

## When NOT to use

- Input is a single omics dataset where you only want TF/kinase activity scores rather than a comparative benchmark of inference methods — use decoupleR/decoupler directly instead of decoupleRBench
- The prior knowledge resource is not a signed, weighted network compatible with decoupleR's mode-of-regulation methods (e.g. unweighted gene sets without TF/regulator mapping)
- The task is differential expression or enrichment analysis on a single contrast rather than evaluating regulator-inference performance against known perturbation labels

## Inputs

- Perturbation transcriptomics compendium (Holland et al. 2020) from Zenodo record 5645208
- Perturbation phospho-proteomics dataset (Hernandez-Armenta et al. 2017) from Zenodo record 5645208
- DoRothEA gene regulatory network
- Kinase substrate network
- Benchmark input tibble with user-specified settings, count table, and metadata table (decoupleRBench format)

## Outputs

- rna_bench_performance.csv — per-method sensitivity ranking matching Supplementary Table S3
- php_bench_performance.csv — per-method performance metrics for kinase inference
- Noise-analysis outputs: median Spearman correlations per condition and Tukey HSD ANOVA results reproducing Supplementary Figures S5 and S6
- Manuscript figures via Rscript R/plot/supp_fig_1.R through supp_fig_7.R

## How to apply

Install the dedicated benchmarking package with devtools::install_github('saezlab/decoupleRBench') and retrieve all benchmark data (Holland et al. 2020 transcriptomics, Hernandez-Armenta et al. 2017 phospho-proteomics, DoRothEA gene regulatory network, kinase substrate network) from Zenodo record 5645208 via 'Rscript R/process/get_data.R'. Run the RNA and phospho-proteomics perturbation benchmarks with 'Rscript R/process/run_rna_bench.R' and 'Rscript R/process/run_php_bench.R', applying the 11 decoupleR methods plus the consensus score to compute TF/kinase activity scores. Evaluate sensitivity by whether each method ranks the experimentally perturbed regulator among the top regulators, then aggregate per-method scores into performance tables (rna_bench_performance.csv, php_bench_performance.csv) and figures. For noise robustness, run run_rna_noise.R (~6 hours) and run_php_noise.R (~2 hours), which add or delete a percentage of edges from DoRothEA and the kinase substrate network, re-infer activities, and compute Spearman correlations between perturbed and original activities, followed by one-way ANOVA with Tukey's HSD comparing predictability across normal, edge-addition and edge-deletion conditions.

## Related tools

- **decoupleR** (Core method suite whose 'decouple' wrapper infers TF/kinase activity scores with 11 methods plus consensus, used inside the benchmark) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (Dedicated benchmarking package that concatenates experiments, builds response/predictor vectors, and computes AUROC/AUPRC with negative-class downsampling) — https://github.com/saezlab/decoupleRBench
- **OmniPath** (Meta-resource of prior knowledge networks leveraged by decoupleR as benchmark input resources) — https://github.com/saezlab/decoupler-py
- **R** (Execution environment; all benchmark, noise and figure scripts are Rscript calls)

## Examples

```
Rscript R/process/run_rna_bench.R
```

## Evaluation signals

- Transcriptomics benchmark: the top three ranked methods in rna_bench_performance.csv are consensus, multivariate linear model and ULM, as in Supplementary Table S3 / Fig. 1C
- Phospho-proteomics benchmark reproduces the same top-three ranking: consensus, multivariate linear model and ULM
- Noise analysis: median Spearman correlation to original activities approximates 0.84 for edge addition and 0.77 for edge deletion
- Tukey HSD adjusted P-values < 2.2e-16 for normal-addition, normal-deletion and deletion-addition comparisons with F = 131
- Per-method AUROC/AUPRC are computed with the downsampling strategy where negative classes are randomly sampled to equal size per permutation

## Limitations

- The RNA noise benchmark takes ~6 hours and the phospho-proteomics noise benchmark ~2 hours to run; scalability analysis scripts may also take a while
- True positive classes in the benchmark are limited by the regulators covered in the perturbation experiments, requiring negative-class downsampling for AUROC/AUPRC
- The R decoupleR package is deprecated; the maintained Python implementation is scverse/decoupler, which is not the version used by this R-based manuscript pipeline

## Evidence

- [readme] Benchmark pipeline: "In this manuscript we have built a separate R package to benchmark decoupleR, called decoupleRBench. You can install it by running: devtools::install_github('saezlab/decoupleRBench')"
- [readme] Data retrieval: "To retrieve all the necessary data, please run: Rscript R/process/get_data.R"
- [readme] Benchmark scripts: "# Run perturbation benchmark
Rscript R/process/run_rna_bench.R
Rscript R/process/run_php_bench.R"
- [readme] Evaluation method: [evidence span withheld — anchor status: external_doc]
- [abstract] Key finding: "Our findings suggest that simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators."
