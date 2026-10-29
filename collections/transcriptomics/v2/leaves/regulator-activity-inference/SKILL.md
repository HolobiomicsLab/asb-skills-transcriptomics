---
name: regulator-activity-inference
description: Use when a count table or expression matrix must be summarized into per-regulator
  (TF/kinase) activity scores via signed, weighted prior-knowledge networks, or when
  multiple inference methods must be combined into a consensus.
when_to_use_negative:
- The task is pathway enrichment scoring unrelated to signed regulator activities
  over a prior-knowledge network.
- No prior-knowledge resource is available or the resource lacks mode-of-regulation/interaction
  weights that the decoupleR methods require.
- Only a single method's output is needed and no benchmarking, consensus aggregation,
  or robustness analysis is in scope.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: decoupleR
  role: Runs the unified `decouple` wrapper that applies the ensemble of activity-inference
    methods to any resource and computes the consensus score
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupler-py
  role: Python implementation of the same enrichment/activity-inference methods
  repo: https://github.com/saezlab/decoupler-py
- name: decoupleRBench
  role: Benchmark package that wraps `decouple` to evaluate inferred activities against
    perturbation experiments via AUROC/AUPRC with downsampling
  repo: https://github.com/saezlab/decoupleRBench
- name: OmniPath
  role: Meta-resource of over 100 prior-knowledge databases leveraged by decoupleR
- name: R
  role: Environment in which the benchmark scripts (run_rna_bench.R, noise and scalability
    analyses) are executed
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:60219e785c4f9382f1658d0a55d5b7915a6cdcf10e77fb1cb9b56f2b4992e241
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OmniPath
  - R
  - decoupleR
  - decoupleRBench
  - decoupler-py
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# regulator-activity-inference

## Summary

Infer transcription factor and kinase regulatory activities from omics data using decoupleR's ensemble of statistical methods (e.g. ULM, MLV, consensus scoring) applied over prior-knowledge networks such as DoRothEA and OmniPath.

## When to use

Use when a count table or expression matrix must be summarized into per-regulator (TF/kinase) activity scores via signed, weighted prior-knowledge networks, or when multiple inference methods must be combined into a consensus.

## When NOT to use

- The task is pathway enrichment scoring unrelated to signed regulator activities over a prior-knowledge network.
- No prior-knowledge resource is available or the resource lacks mode-of-regulation/interaction weights that the decoupleR methods require.
- Only a single method's output is needed and no benchmarking, consensus aggregation, or robustness analysis is in scope.

## Inputs

- Omics expression/count matrix (transcriptomics or phospho-proteomics) with matched metadata table
- Signed, weighted prior-knowledge network (DoRothEA gene regulatory network, kinase-substrate network, or OmniPath resource)
- Benchmark data: count table plus metadata from single-gene perturbation experiments (Zenodo record 5645208)

## Outputs

- Per-regulator activity score matrices, one per method (activities inferred for each regulator and experiment)
- consensus_scores.csv containing one consensus activity estimate per regulator with row identifiers matching the input activity matrices
- AUROC/AUPRC benchmark results and noise-analysis Spearman correlations per method and perturbation condition

## How to apply

Provide the transcriptomic or phospho-proteomic data together with a signed interaction network (DoRothEA for TFs, a kinase-substrate network for phospho-proteomics, or OmniPath as a meta-resource of 100+ databases) and run decoupleR's unified `decouple` wrapper, which can flexibly pair any method with any resource, including methods that use mode of regulation and interaction weights. For multi-method runs, standardize each method's activity estimates to z-scores per regulator and average them to obtain the consensus score (as specified in the Implementation section and Supplementary Note). To evaluate performance, benchmark against single-gene perturbation experiments using decoupleRBench: concatenate activities across experiments into predictor vectors and perturbation labels into response vectors, then compute AUROC and AUPRC with negative-class downsampling. For network-noise robustness, add or delete a percentage of network edges, re-infer activities, and compare perturbed vs original activities by median Spearman correlation plus one-way ANOVA with Tukey's HSD. Expect linear models (ULM, multivariate LM) and the consensus score across top methods to outperform other methods at predicting perturbed regulators.

## Related tools

- **decoupleR** (Runs the unified `decouple` wrapper that applies the ensemble of activity-inference methods to any resource and computes the consensus score) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupler-py** (Python implementation of the same enrichment/activity-inference methods) — https://github.com/saezlab/decoupler-py
- **decoupleRBench** (Benchmark package that wraps `decouple` to evaluate inferred activities against perturbation experiments via AUROC/AUPRC with downsampling) — https://github.com/saezlab/decoupleRBench
- **OmniPath** (Meta-resource of over 100 prior-knowledge databases leveraged by decoupleR)
- **R** (Environment in which the benchmark scripts (run_rna_bench.R, noise and scalability analyses) are executed)

## Examples

```
Rscript R/process/run_rna_bench.R
```

## Evaluation signals

- consensus_scores.csv must contain one consensus activity estimate per regulator with row identifiers matching the input activity matrices.
- The consensus score must rank among the top methods for recovering perturbed TFs, consistent with the reported top three of consensus, multivariate linear model and ULM.
- For noise analysis, reproduced median Spearman correlations of activities to the originals should approximate 0.84 for edge addition and 0.77 for edge deletion.
- Tukey HSD adjusted P-values must be <2.2e-16 (F = 131) for normal-addition, normal-deletion and deletion-addition comparisons in the predictability analysis.
- Benchmark evaluation must use the downsampling strategy: per permutation, an equal number of negative classes are randomly sampled, since true positives are limited by regulators covered in perturbation experiments.

## Limitations

- Activity estimates are more affected by deleting edges from the prior-knowledge network than by adding them (median Spearman 0.77 vs 0.84), so network quality directly conditions results.
- True positive classes are limited by the regulators covered in perturbation experiments, requiring downsampling of negatives in evaluation.
- Noise benchmarks are compute-intensive: run_rna_noise.R takes ~6 hours and run_php_noise.R ~2 hours.

## Evidence

- [abstract] Unified framework and package availability: "we present decoupleR, a Bioconductor and Python package containing computational methods to extract these activities within a unified framework"
- [abstract] Prior-knowledge resource: "it leverages OmniPath, a meta-resource comprising over 100 databases of prior knowledge"
- [abstract] Performance finding: "Our findings suggest that simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators."
- [readme] Benchmark evaluation procedure: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmark installation and reproducibility: "In this manuscript we have built a separate R package to benchmark decoupleR, called decoupleRBench. You can install it by running: devtools::install_github('saezlab/decoupleRBench')"
