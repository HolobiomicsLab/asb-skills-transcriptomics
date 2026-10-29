---
name: deg-method-vote-threshold-evaluation
description: Use when you have simulated RNA-seq count data with known DE/non-DE labels
  and need to choose a consensus combination strategy (vote threshold vs combined-FDR)
  across DESeq2, NOISeq, and limma results.
when_to_use_negative:
- Data are real experimental datasets (e.g. Lafora or PMM2-CDG) with no ground-truth
  DE labels, so precision/recall/F-measure cannot be computed
- You need a final recommendation for a new organism, tissue, or experimental design
  — the optimal threshold of 2 is data-specific and the authors caution against extrapolating
  it as a general rule
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package providing the DEgenes Hunter pipeline and auxiliary
    scripts for simulating RNA-seq data used to generate truth labels
  repo: https://bioconductor.org/packages/ExpHunterSuite
- name: DEgenes Hunter
  role: Pipeline that produces individual DEG calls from each integrated method (DESeq2,
    NOISeq, limma) for vote-threshold and combined-FDR combination
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: R
  role: Runtime environment in which the pipeline is run modularly and metrics are
    computed
- name: Bioconductor
  role: Distribution channel for the ExpHunterSuite package
  repo: https://bioconductor.org/packages/ExpHunterSuite
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:d441454b2c14f12bf9ba0d87f97417748cc381e1aa0b39fc2af14e6fafff4d41
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - DEgenes Hunter
  - ExpHunter Suite
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# deg-method-vote-threshold-evaluation

## Summary

Evaluates how minimum-vote thresholds (1–4) for combining DEG calls from DESeq2, NOISeq, and limma in the DEgenes Hunter pipeline compare to the combined-FDR data-fusion criterion, using precision, recall, and F-measure against known truth labels on simulated RNA-seq data.

## When to use

Use when you have simulated RNA-seq count data with known DE/non-DE labels and need to choose a consensus combination strategy (vote threshold vs combined-FDR) across DESeq2, NOISeq, and limma results.

## When NOT to use

- Data are real experimental datasets (e.g. Lafora or PMM2-CDG) with no ground-truth DE labels, so precision/recall/F-measure cannot be computed
- You need a final recommendation for a new organism, tissue, or experimental design — the optimal threshold of 2 is data-specific and the authors caution against extrapolating it as a general rule

## Inputs

- Simulated RNA-seq count datasets with known differentially-expressed and non-DE gene truth labels (from upstream simulation step)
- Per-method DEG call tables from DEgenes Hunter (DESeq2, NOISeq, limma)

## Outputs

- deg_metrics_comparison.csv with precision/recall/F-measure values in [0,1] for each vote threshold and the combined-FDR criterion
- Ranked comparison of combination strategies identifying the F-measure-optimal threshold

## How to apply

Run the DEgenes Hunter pipeline from ExpHunter Suite in R on simulated count datasets to obtain per-method DEG calls from DESeq2, NOISeq, and limma. Apply minimum-vote thresholds of 1, 2, 3, and 4 — a gene is called DE if at least that many methods vote for it — and also apply the combined-FDR data-fusion criterion that integrates per-method statistics into a single call set. Compute precision, recall, and F-measure for each strategy against the simulated truth labels and write method-by-metric results to deg_metrics_comparison.csv with values in [0,1]. Raising the vote threshold increases precision at the cost of recall; the combined-FDR criterion performs comparably to a vote threshold of 2, which is the optimum by F-measure (e.g. on spike-in data).

## Related tools

- **ExpHunter Suite** (R/Bioconductor package providing the DEgenes Hunter pipeline and auxiliary scripts for simulating RNA-seq data used to generate truth labels) — https://bioconductor.org/packages/ExpHunterSuite
- **DEgenes Hunter** (Pipeline that produces individual DEG calls from each integrated method (DESeq2, NOISeq, limma) for vote-threshold and combined-FDR combination) — https://github.com/seoanezonjic/ExpHunterSuite
- **R** (Runtime environment in which the pipeline is run modularly and metrics are computed)
- **Bioconductor** (Distribution channel for the ExpHunterSuite package) — https://bioconductor.org/packages/ExpHunterSuite

## Examples

```
Rscript -e "ExpHunterSuite::install_DEgenes_hunter('install_folder')" && bash launch_DEgenesHunter.sh
```

## Evaluation signals

- All precision, recall, and F-measure values fall within [0,1]
- Combined-FDR criterion metrics are close to those of the minimum-vote threshold of 2 across all measures
- Precision increases and recall decreases monotonically as the vote threshold is raised from 1 to 4
- Vote threshold 2 attains the highest F-measure among the four thresholds (as reported for spike-in data)
- Results reproduce the Fig. 3 style reference summary of threshold-by-metric behaviour

## Limitations

- The optimal vote threshold may not generalize to other organisms, tissues, and experimental designs; the authors are hesitant to extrapolate a general rule
- A competing recommendation of much higher thresholds was based on the analysis of a single human dataset
- Different DE methods behave quite differently depending on data properties (e.g. DESeq2 was most conservative for Lafora but least conservative for PMM2-CDG), affecting vote outcomes

## Evidence

- [results] Raising the minimum-vote threshold increases precision at the cost of recall, and the combined-FDR criterion performs comparatively well: [evidence span withheld — anchor status: not_found]
- [discussion] for the spike-in data the optimum threshold was 2 in terms of F-measure: "for the spike-in data the optimum threshold was 2 in terms of F-measure"
- [discussion] we would suggest the use of a minimum vote threshold cut-off where possible: [evidence span withheld — anchor status: not_found]
- [discussion] this may differ for other organisms, tissues and experimental designs: "this may differ for other organisms, tissues and experimental designs. As such, we are hesitant to extrapolate these findings to a more general rule"
- [readme] Install and configure your R environment as is explained: "Rscript -e "ExpHunterSuite::install_DEgenes_hunter('install_folder')""
