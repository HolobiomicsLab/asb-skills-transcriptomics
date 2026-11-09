---
name: consensus-ensemble-scoring
description: Use when multiple decoupleR methods (e.g., ULM, MLV, weighted mean) have
  been run simultaneously on the same network and transcriptomic/phospho-proteomic
  data and a single regulator-level activity estimate per regulator is needed.
when_to_use_negative:
- Only a single inference method is being run — the consensus mean z-score requires
  multiple methods' activity estimates to aggregate
- The goal is method benchmarking of raw method scores rather than producing a final
  consensus estimate (use the decoupleRBench evaluation pipeline directly instead)
- Inputs are omics count data prior to activity inference — consensus scoring operates
  on the per-method activity score matrices, not on raw transcriptomics counts
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: decoupleR
  role: Provides the unified framework and the consensus score (mean z-score across
    methods' activity estimates); run via its decouple wrapper
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: Benchmark pipeline that infers activities per method and regulator/experiment
    and computes AUROC/AUPRC with downsampling to evaluate the consensus score
  repo: https://github.com/saezlab/decoupleRBench
- name: R
  role: Language for running the benchmark scripts (Rscript R/process/run_rna_bench.R)
    and decoupleR's Bioconductor implementation
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: Python
  role: decoupler-py implementation of the same ensemble of enrichment methods for
    Python workflows
  repo: https://github.com/saezlab/decoupler-py
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:98ab7bf1655509a9bdc754660f328ce7817d81e1f65059189c5d972536d92438
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# consensus-ensemble-scoring

## Summary

Compute a single consensus regulator activity score in decoupleR by aggregating multiple methods' activity estimates via a mean z-score per regulator, which the decoupleR benchmark found among the top-performing approaches for predicting perturbed regulators.

## When to use

Use when multiple decoupleR methods (e.g., ULM, MLV, weighted mean) have been run simultaneously on the same network and transcriptomic/phospho-proteomic data and a single regulator-level activity estimate per regulator is needed.

## When NOT to use

- Only a single inference method is being run — the consensus mean z-score requires multiple methods' activity estimates to aggregate
- The goal is method benchmarking of raw method scores rather than producing a final consensus estimate (use the decoupleRBench evaluation pipeline directly instead)
- Inputs are omics count data prior to activity inference — consensus scoring operates on the per-method activity score matrices, not on raw transcriptomics counts

## Inputs

- Per-method regulator activity score matrices from the decoupleR decouple wrapper (one matrix per method, shared regulator identifiers)
- Transcriptomics perturbation benchmark data from Zenodo record 5645208 (DoRothEA-based TF activity estimates from single-gene perturbation experiments)
- Count table and metadata table used as benchmark input to decoupleRBench

## Outputs

- consensus_scores.csv containing one consensus activity estimate per regulator, with row identifiers matching the input per-method activity matrices
- AUROC/AUPRC benchmark results for the consensus score versus individual methods

## How to apply

Run each activity-inference method through decoupleR's unified `decouple` wrapper (as orchestrated by decoupleRBench) so that per-method regulator activity score matrices are produced with matched row identifiers. Standardize each method's activity estimates to z-scores per regulator, then compute the consensus score as the mean z-score across methods, following the Implementation section and Supplementary Note of the manuscript. Evaluate the consensus against individual methods using the perturbation benchmark: concatenate experiments into a response vector (regulator perturbed or not) and predictor vector (regulator activities), then calculate AUROC and AUPRC with the downsampling strategy where equal numbers of negative classes are randomly sampled per permutation. Rationale: the decoupleR benchmark on DoRothEA-based single-gene TF perturbation data showed the consensus score ranks among the top three, alongside the multivariate linear model and ULM baselines.

## Related tools

- **decoupleR** (Provides the unified framework and the consensus score (mean z-score across methods' activity estimates); run via its decouple wrapper) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (Benchmark pipeline that infers activities per method and regulator/experiment and computes AUROC/AUPRC with downsampling to evaluate the consensus score) — https://github.com/saezlab/decoupleRBench
- **R** (Language for running the benchmark scripts (Rscript R/process/run_rna_bench.R) and decoupleR's Bioconductor implementation) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **Python** (decoupler-py implementation of the same ensemble of enrichment methods for Python workflows) — https://github.com/saezlab/decoupler-py

## Examples

```
Rscript R/process/run_rna_bench.R
```

## Evaluation signals

- consensus_scores.csv contains exactly one consensus activity estimate per regulator, and row identifiers match the input per-method activity matrices
- The consensus method appears among the top-ranked methods at predicting perturbed regulators on the transcriptomics benchmark, consistent with the reported top three of consensus, multivariate linear model, and ULM
- Each method's activity estimates were standardized to z-scores per regulator before averaging — verify no method contributes on a non-standardized scale
- AUROC and AUPRC computed with the downsampling strategy: for each permutation an equal number of negative classes are randomly sampled
- Consensus estimates are reproducible via the manuscript reproduction scripts run against Zenodo record 5645208 data

## Limitations

- Consensus score quality depends on the constituent methods; the reported advantage holds for simple linear models and the consensus across top methods on transcriptomic and phospho-proteomic perturbation experiments, not for arbitrary method combinations
- True positive classes in the benchmark are limited by the regulators covered in the perturbation experiments, requiring the downsampling evaluation strategy
- The multi-method consensus is documented in the Supplementary Note; users should consult it for details beyond the Implementation section

## Evidence

- [abstract] Simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators.: [evidence span withheld — anchor status: not_found]
- [abstract] decoupleR is a Bioconductor and Python package with a unified framework for extracting biological activities.: "we present decoupleR, a Bioconductor and Python package containing computational methods to extract these activities within a unified framework"
- [readme] decoupleRBench is a separate R package built to benchmark decoupleR.: "In this manuscript we have built a separate R package to benchmark decoupleR, called decoupleRBench."
- [readme] Evaluation uses AUROC/AUPRC with a downsampling strategy for limited positive classes.: [evidence span withheld — anchor status: external_doc]
