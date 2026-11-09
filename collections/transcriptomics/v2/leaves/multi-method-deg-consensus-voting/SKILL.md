---
name: multi-method-deg-consensus-voting
description: Use when a raw RNA-seq count table plus sample metadata must be screened
  for DEGs and no single method can be trusted, e.g. rare-disease model datasets like
  PMM2-CDG or Lafora disease where method choice strongly changes DEG counts.
when_to_use_negative:
- Input is already a normalized expression matrix or a pre-computed DEG list rather
  than a raw count table produced from initial RNA-seq analysis
- 'A ground-truth spike-in setting is being used: the optimum vote threshold there
  was 2 by F-measure, so the strict all-methods (3-vote) cut-off would under-call
  true positives'
- Only one method's output is available or the analysis design does not permit running
  multiple DEG methods, since the vote threshold requires at least the chosen minimum
  number of method calls
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package that orchestrates the DEG detection and consensus integration
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DEgenes Hunter
  role: Pipeline that runs the individual DEG methods and combines their calls by
    vote threshold
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DESeq2
  role: One of the combined DEG methods; most conservative on Lafora disease data
    but by far the least conservative on PMM2-CDG
- name: NOISeq
  role: Additional DEG method combined in the Lafora disease analysis; showed high
    recall
- name: limma
  role: DEG method combined in the consensus; showed higher precision
- name: Bioconductor
  role: Distribution channel for the ExpHunterSuite R/Bioconductor package
  repo: https://bioconductor.org/packages/ExpHunterSuite
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:99823d168ca01f21d61c009bc7690133b77bf6f784d2535ef82be5c93d8e1e36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seoanezonjic/ExpHunterSuite
  tools:
  - Bioconductor
  - DESeq2
  - DEgenes Hunter
  - ExpHunter Suite
  - NOISeq
  - R
  - limma
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7916666666666666
n_claims: 24
---

# Multi-method DEG consensus voting

## Summary

Combines differential expression calls from multiple DEG detection methods (DESeq2, edgeR, NOISeq, limma) run inside DEgenes Hunter by a minimum vote threshold, because individual methods behave inconsistently across datasets. Produces a per-gene consensus table with per-method calls, vote counts, and estimated logFC.

## When to use

Use when a raw RNA-seq count table plus sample metadata must be screened for DEGs and no single method can be trusted, e.g. rare-disease model datasets like PMM2-CDG or Lafora disease where method choice strongly changes DEG counts.

## When NOT to use

- Input is already a normalized expression matrix or a pre-computed DEG list rather than a raw count table produced from initial RNA-seq analysis
- A ground-truth spike-in setting is being used: the optimum vote threshold there was 2 by F-measure, so the strict all-methods (3-vote) cut-off would under-call true positives
- Only one method's output is available or the analysis design does not permit running multiple DEG methods, since the vote threshold requires at least the chosen minimum number of method calls

## Inputs

- RNA-seq raw count table (gene x sample matrix)
- Sample metadata / experimental design describing conditions (e.g. mutant vs wild-type)
- Reference annotation for functional analysis (optional, e.g. via orthologous translation)

## Outputs

- Per-gene consensus DEG table (per-method call, vote count, estimated logFC), e.g. deg_results.csv
- Consensus DEG gene list at a chosen vote threshold
- ExpHunter Suite HTML results report

## How to apply

Run the DEgenes Hunter module of ExpHunter Suite on the count matrix, executing several methods (e.g. DESeq2, edgeR, limma; NOISeq in the Lafora analysis) on the experimental design. Combine per-method DEG calls with a minimum vote threshold cut-off: the article recommends 'the use of a minimum vote threshold cut-off where possible'; in the PMM2-CDG reproduction a cut-off of 3 (all methods must agree) yields 345 genes called by all three out of 1081 detected by at least one, while the optimum threshold on the spike-in data was 2 by F-measure. Summarize a per-gene table recording each method's call, the vote count, and the estimated logFC (PMM2-CDG outlier logFCs ≈ -23.5, -8.8, -2.6). The rationale is that DE methods rest on different assumptions and can over-predict or estimate inaccurate values depending on data properties, so consensus and classifiers lead to optimal results in simulated and spike-in experiments.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package that orchestrates the DEG detection and consensus integration) — https://github.com/seoanezonjic/ExpHunterSuite
- **DEgenes Hunter** (Pipeline that runs the individual DEG methods and combines their calls by vote threshold) — https://github.com/seoanezonjic/ExpHunterSuite
- **DESeq2** (One of the combined DEG methods; most conservative on Lafora disease data but by far the least conservative on PMM2-CDG)
- **NOISeq** (Additional DEG method combined in the Lafora disease analysis; showed high recall)
- **limma** (DEG method combined in the consensus; showed higher precision)
- **Bioconductor** (Distribution channel for the ExpHunterSuite R/Bioconductor package) — https://bioconductor.org/packages/ExpHunterSuite

## Examples

```
bash launch_DEgenesHunter.sh
```

## Evaluation signals

- On the PMM2-CDG dataset, reproduce 1081 DEGs detected by at least one of the three methods and 345 detected by all three at the minimum vote threshold of 3
- Verify the consensus table records each method's call, the vote count, and the estimated logFC, and reproduce outlier logFC values of approximately -23.5, -8.8, and -2.6 (deg_results.csv)
- Check the vote threshold is applied as a minimum cut-off (a gene needs at least the threshold number of method calls), not a simple union or intersection unless the threshold equals 1 or the number of methods
- On spike-in data with known truth, confirm threshold choice against F-measure (optimum was 2 in the article) rather than assuming 3
- Confirm an ExpHunter Suite HTML report is generated when the full workflow is executed

## Limitations

- The optimal vote threshold may not generalize: the authors note it 'may differ for other organisms, tissues and experimental designs' and are hesitant to extrapolate a general rule
- Method behavior is dataset-dependent: DESeq2 was the most conservative method on the Lafora disease experiment but by far the least conservative for PMM2-CDG
- Incorrect fold change estimation by individual methods can lead to poor qPCR target selection and affects downstream logFC analyses such as GSEA
- A competing recommendation for much higher thresholds was based on the analysis of a single human dataset, so cross-study comparisons are limited

## Evidence

- [abstract] Application to simulated and spike-in experiments shows that combining multiple methods using consensus and classifiers leads to optimal results: "Application to simulated and spike-in experiments shows that combining multiple methods using consensus and classifiers leads to optimal results"
- [intro] DE methods are based on different assumptions and analysis procedures, making it impossible to know the most appropriate method for a given dataset: "DE methods are based on different assumptions and analysis procedures, making it impossible to know the most appropriate method for a given dataset"
- [discussion] we would suggest the use of a minimum vote threshold cut-off where possible: "we would suggest the use of a minimum vote threshold cut-off where possible"
- [readme] Use count tables obtained to execute DEgenes Hunter expression analysis as done in Lafora or PMM2 but changing input file.: [evidence span withheld — anchor status: external_doc]
