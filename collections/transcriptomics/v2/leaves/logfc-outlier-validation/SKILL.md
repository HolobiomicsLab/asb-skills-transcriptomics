---
name: logfc-outlier-validation
description: Use when a multi-method DEG consensus table (DESeq2, edgeR, limma) contains
  outlier or extreme logFC estimates that must be verified before downstream use such
  as qPCR target selection or GSEA.
when_to_use_negative:
- When you only need vote-count-based consensus DEG membership and do not intend to
  use logFC values downstream (e.g. for GSEA or qPCR target selection)
- On simulated or spike-in datasets where the goal is choosing an optimal vote threshold
  (e.g. threshold 2 by F-measure on spike-in data) rather than validating biological
  fold-change estimates
- When the input is already a validated, per-gene logFC table rather than raw per-method
  DEG results
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package whose DEgenes Hunter pipeline executes the DESeq2,
    edgeR, and limma DEG methods and merges their results for logFC validation
  repo: https://bioconductor.org/packages/ExpHunterSuite
- name: DEgenes Hunter
  role: Pipeline module producing the per-gene consensus table (method call, vote
    count, logFC) that the outlier logFC values are validated against
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DESeq2
  role: DEG method in the suite; on the PMM2-CDG experiment it was by far the least
    conservative, so its logFC estimates are a key comparison point
- name: limma
  role: DEG method in the suite used as the reference estimate when reconciling outlier
    logFC values across methods
- name: ExpHunterSuiteExamples
  role: Repository providing the PMM2-CDG count data, metadata, and launch_DEgenesHunter.sh
    used to regenerate the results
  repo: https://github.com/fmjabato/ExpHunterSuiteExamples
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:fa001142425f5161b6273c22d60a23910a46124ae58c048fed702acc7cc5d3e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - DEgenes Hunter
  - ExpHunter Suite
  - ExpHunterSuiteExamples
  - R
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# logfc-outlier-validation

## Summary

Validates log2 fold-change estimates from multi-method differential expression consensus on the PMM2-CDG RNA-seq data by checking outlier genes against expected logFC values (~-23.5, -8.8, -2.6), since incorrect fold-change estimation undermines target selection and downstream analyses such as GSEA.

## When to use

Use when a multi-method DEG consensus table (DESeq2, edgeR, limma) contains outlier or extreme logFC estimates that must be verified before downstream use such as qPCR target selection or GSEA.

## When NOT to use

- When you only need vote-count-based consensus DEG membership and do not intend to use logFC values downstream (e.g. for GSEA or qPCR target selection)
- On simulated or spike-in datasets where the goal is choosing an optimal vote threshold (e.g. threshold 2 by F-measure on spike-in data) rather than validating biological fold-change estimates
- When the input is already a validated, per-gene logFC table rather than raw per-method DEG results

## Inputs

- PMM2-CDG RNA-seq count matrix and associated metadata from github.com/fmjabato/ExpHunterSuiteExamples
- Per-method DEG detection results (DESeq2, edgeR, limma) from the DEgenes Hunter run
- Consensus DEG table recording each method's call, vote count, and estimated logFC per gene

## Outputs

- deg_results.csv with per-gene method calls, vote counts, and validated logFC estimates
- Consensus counts: 1081 DEGs detected by at least one method, 345 detected by all three
- List of outlier genes with verified logFC values (~-23.5, ~-8.8, ~-2.6)

## How to apply

After running the DEgenes Hunter module of ExpHunter Suite on the PMM2-CDG count matrix, combine per-method DEG calls with a minimum vote threshold cut-off of 3 (a gene must be called DE by all three methods) and record each method's call, vote count, and estimated logFC in a per-gene table. Because widely-used expression analysis methods can over-predict or estimate inaccurate logFC values depending on data properties, inspect the consensus table for outlier genes and confirm their logFC values reproduce approximately -23.5, -8.8, and -2.6. Note that on this dataset DESeq2 was by far the least conservative method, so logFC divergence across methods is expected and should be resolved before trusting estimates. Save the validated results as deg_results.csv. This check matters because incorrect fold-change estimation can lead to poor qPCR target selection and distorts downstream logFC-based analyses such as GSEA.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package whose DEgenes Hunter pipeline executes the DESeq2, edgeR, and limma DEG methods and merges their results for logFC validation) — https://bioconductor.org/packages/ExpHunterSuite
- **DEgenes Hunter** (Pipeline module producing the per-gene consensus table (method call, vote count, logFC) that the outlier logFC values are validated against) — https://github.com/seoanezonjic/ExpHunterSuite
- **DESeq2** (DEG method in the suite; on the PMM2-CDG experiment it was by far the least conservative, so its logFC estimates are a key comparison point)
- **limma** (DEG method in the suite used as the reference estimate when reconciling outlier logFC values across methods)
- **ExpHunterSuiteExamples** (Repository providing the PMM2-CDG count data, metadata, and launch_DEgenesHunter.sh used to regenerate the results) — https://github.com/fmjabato/ExpHunterSuiteExamples

## Examples

```
bash launch_DEgenesHunter.sh
```

## Evaluation signals

- The consensus table confirms 1081 DEGs detected by at least one method and 345 detected by all three methods (vote threshold = 3)
- Outlier genes reproduce logFC values of approximately -23.5, -8.8, and -2.6
- Every gene in the results table carries a per-method DEG call, a vote count between 0 and 3, and an estimated logFC
- Results are reproducible by re-running launch_DEgenesHunter.sh from the ExpHunterSuiteExamples PMM2 folder
- deg_results.csv is produced with the expected per-gene schema

## Limitations

- Optimal vote thresholds do not generalize: the article notes the optimum threshold (2 for spike-in data) 'may differ for other organisms, tissues and experimental designs' and the authors are 'hesitant to extrapolate these findings to a more general rule'
- Incorrect logFC estimation can propagate into downstream analyses such as GSEA and lead to poor qPCR target selection
- Methods behave differently depending on data properties (DESeq2 was most conservative on Lafora but least conservative on PMM2-CDG), so logFC agreement across methods cannot be assumed

## Evidence

- [discussion] Incorrect fold-change estimation affects qPCR target selection and logFC-based downstream analyses: "its incorrect estimation could lead to poor target selection. This also has important implications for downstream analyses that use logFC, such as the popular GSEA method"
- [intro] Methods can estimate inaccurate logFC values depending on data properties: "some widely-used expression analysis methods can behave quite differently depending on the properties of the data, potentially over-predicting or estimating inaccurate values"
- [discussion] DESeq2 was far less conservative on the PMM2-CDG experiment, driving cross-method logFC divergence: "for the PMM2-CDG experiment DESeq2 was by far the least conservative"
- [discussion] Consensus DEG calls are combined via a minimum vote threshold cut-off: "we would suggest the use of a minimum vote threshold cut-off where possible"
- [readme] The PMM2 experiment is replicated by executing the DEgenes Hunter launcher script: [evidence span withheld — anchor status: external_doc]
