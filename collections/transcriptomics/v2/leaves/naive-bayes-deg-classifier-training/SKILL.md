---
name: naive-bayes-deg-classifier-training
description: Use when combining differential expression results from multiple DEG
  methods (e.g., DESeq2, NOISeq, limma) into a single robust DEG call, and a labelled
  training set can be generated from simulated data modelled on the target experiment.
when_to_use_negative:
- When a single DEG method is sufficient and validated for the dataset — e.g., DESeq2
  alone was already the best-performing method on the Lafora disease spike-in comparison
- When no ground-truth labels or simulation framework are available to train the classifier,
  since the model must be trained on simulated data modelled on the target experiment
- When the goal is functional enrichment or co-expression analysis rather than DEG
  detection and result integration
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package providing the DEG method combination workflow, including
    the Naïve Bayes classifier trained on simulated data
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: R/Bioconductor
  role: Runtime environment in which the classifier is trained and the model is serialized
    as .rds
  repo: https://bioconductor.org/packages/ExpHunterSuite
- name: DESeq2
  role: One of the per-method p-value sources combined as classifier features; also
    the benchmark single method (F-measure ~0.88 on spike-in data)
- name: Autoflow
  role: Pipeline executor used by the ExpHunterSuiteExamples scripts that generate
    the synthetic results for training and evaluation
  repo: https://github.com/seoanezonjic/autoflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:65433c154359a430487f4730b0ef46990d2821e685dca8b019c8aced187cd6e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seoanezonjic/ExpHunterSuite
  tools:
  - Autoflow
  - DESeq2
  - ExpHunter Suite
  - R
  - R/Bioconductor
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# naive-bayes-deg-classifier-training

## Summary

Train a Naïve Bayes classifier that integrates per-gene p-values from multiple DEG detection methods into DEG/non-DEG calls, following the ExpHunter Suite approach of learning from simulated RNA-seq data that mimics the target dataset's characteristics.

## When to use

Use when combining differential expression results from multiple DEG methods (e.g., DESeq2, NOISeq, limma) into a single robust DEG call, and a labelled training set can be generated from simulated data modelled on the target experiment.

## When NOT to use

- When a single DEG method is sufficient and validated for the dataset — e.g., DESeq2 alone was already the best-performing method on the Lafora disease spike-in comparison
- When no ground-truth labels or simulation framework are available to train the classifier, since the model must be trained on simulated data modelled on the target experiment
- When the goal is functional enrichment or co-expression analysis rather than DEG detection and result integration

## Inputs

- Labelled simulation results: per-gene vectors of p-values from multiple DEG detection methods with DEG/non-DEG ground-truth labels
- Count tables from the target RNA-seq experiment (e.g., spike-in dataset) processed with DEgenes Hunter to obtain per-method p-values
- Characteristics of the real dataset (e.g., Lafora disease dataset) used to parameterize the simulations

## Outputs

- Serialized fitted Naïve Bayes model (naive_bayes_model.rds)
- Integrated DEG calls (DEG/non-DEG predictions) for the target dataset
- Performance metrics, e.g., F-measure on the spike-in dataset (~0.88, comparable to vote threshold of 2 and DESeq2)

## How to apply

First generate labelled simulated RNA-seq datasets whose characteristics (e.g., those of the Lafora disease dataset) provide per-gene vectors of p-values from multiple DEG detection methods with known DEG/non-DEG ground-truth labels. Assemble the per-method p-values as classifier features and the DEG/non-DEG labels as response classes, then fit a Naïve Bayes classifier within R/Bioconductor using the ExpHunter Suite. Serialize the fitted model (e.g., to naive_bayes_model.rds) so it can be applied to real experiments. Apply the trained classifier to the target dataset to produce integrated DEG calls, and benchmark against simpler combination strategies such as a vote threshold cut-off (optimum threshold 2 by F-measure on the spike-in data). Rationale: combining methods via a learned classifier achieved results as good as the best single and combination methods on the spike-in dataset, and no previous work had applied machine learning classifiers to this combination problem.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package providing the DEG method combination workflow, including the Naïve Bayes classifier trained on simulated data) — https://github.com/seoanezonjic/ExpHunterSuite
- **R/Bioconductor** (Runtime environment in which the classifier is trained and the model is serialized as .rds) — https://bioconductor.org/packages/ExpHunterSuite
- **DESeq2** (One of the per-method p-value sources combined as classifier features; also the benchmark single method (F-measure ~0.88 on spike-in data))
- **Autoflow** (Pipeline executor used by the ExpHunterSuiteExamples scripts that generate the synthetic results for training and evaluation) — https://github.com/seoanezonjic/autoflow

## Examples

```
Rscript -e 'options(timeout=1000); devtools::install_github("seoanezonjic/ExpHunterSuite")'
```

## Evaluation signals

- Spike-in F-measure of the trained classifier is ~0.88 and comparable to the vote threshold of 2 and to DESeq2
- Classifier performance matches the Naïve Bayes row of Table 1 (comparable to the best performing single and combination methods)
- Serialized model file naive_bayes_model.rds is produced and can be reloaded to score held-out simulated or spike-in genes
- Feature matrix has one p-value column per DEG detection method and response labels match the simulation ground truth (DEG/non-DEG)
- HTML results report is generated when running the launch.sh/get_results.sh workflow from ExpHunterSuiteExamples

## Limitations

- The Naïve Bayes model is trained on simulations derived from the characteristics of a specific dataset (Lafora disease); performance may not generalize to other organisms, tissues, or experimental designs
- More elaborate classifiers than Naïve Bayes were left for future work, so this is the simplest learned combination strategy evaluated
- The optimal vote threshold comparison (threshold 2) is itself dataset-specific; the authors were hesitant to extrapolate it as a general rule

## Evidence

- [abstract] Naïve Bayes classifier trained on simulated data: "result integration via data-fusion, threshold based methods, and a Naïve Bayes classifier trained on simulated data"
- [intro] Novel methodology combining DEG results including a Naïve Bayes classifier: "We provide novel methodology to combine results, including a Naïve Bayes classifier approach, that lead to robust DEG detection"
- [discussion] Naïve Bayes performs as well as best single and combination methods on spike-in data: [evidence span withheld — anchor status: not_found]
- [discussion] Spike-in optimum vote threshold was 2 by F-measure: "for the spike-in data the optimum threshold was 2 in terms of F-measure"
- [readme] Installing ExpHunterSuite with extended R timeout for large dependency reactome.db: [evidence span withheld — anchor status: external_doc]
