---
name: auc-ranking-metric-computation
description: Use when evaluating DEG detection methods on simulated expression datasets
  that carry ground-truth DE labels, and you need per-method AUC over ranked significance
  scores to compare detection performance.
when_to_use_negative:
- Real experimental datasets without ground truth (e.g. the Lafora disease or PMM2-CDG
  experiments), where AUC against known DE labels cannot be computed and precision/recall
  behaviour of each method must instead be assessed qualitatively
- Spike-in benchmark datasets where performance is judged by F-measure of a vote threshold
  cut-off (optimum threshold 2 for the spike-in data) rather than by simulated-data
  AUC
- When the goal is producing final DEG calls for downstream functional enrichment
  or co-expression analysis rather than benchmarking detection methods
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package providing the DEgenes Hunter pipeline and auxiliary
    scripts for assessing performance and simulating RNA-seq data used to generate
    the ground-truth simulated datasets
  repo: https://bioconductor.org/packages/ExpHunterSuite
- name: DEgenes Hunter
  role: Pipeline within ExpHunter Suite used to run DESeq2, limma and NOISeq on each
    simulated dataset before p-value integration into combined-FDR calls
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DESeq2
  role: Individual DEG detection method whose ranked significance measure is scored
    by AUC against the simulated ground truth
- name: limma
  role: Individual DEG detection method whose ranked significance measure is scored
    by AUC against the simulated ground truth
- name: NOISeq
  role: Individual DEG detection method whose ranked significance measure is scored
    by AUC against the simulated ground truth
- name: R
  role: Environment in which the AUC computation, p-value integration and results-table
    assembly are run modularly
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:6926166853b8c78577973e5125cdb472d556d253dac5ad01343a8e4aa4504e71
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - DEgenes Hunter
  - ExpHunter Suite
  - NOISeq
  - R
  - limma
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# auc-ranking-metric-computation

## Summary

Benchmark skill that ranks genes by each DEG method's significance measure and computes AUC against simulated ground-truth DE labels, comparing DESeq2, edgeR, limma, NOISeq and the combined-FDR integration across simulated logFC conditions.

## When to use

Use when evaluating DEG detection methods on simulated expression datasets that carry ground-truth DE labels, and you need per-method AUC over ranked significance scores to compare detection performance.

## When NOT to use

- Real experimental datasets without ground truth (e.g. the Lafora disease or PMM2-CDG experiments), where AUC against known DE labels cannot be computed and precision/recall behaviour of each method must instead be assessed qualitatively
- Spike-in benchmark datasets where performance is judged by F-measure of a vote threshold cut-off (optimum threshold 2 for the spike-in data) rather than by simulated-data AUC
- When the goal is producing final DEG calls for downstream functional enrichment or co-expression analysis rather than benchmarking detection methods

## Inputs

- A. thaliana-derived simulated RNA-seq count matrices with varying logFC conditions (upstream task_001 simulation output)
- Ground-truth differentially expressed gene labels for each simulated dataset
- Per-method significance outputs (p-values / FDR) from DESeq2, limma, NOISeq run via DEgenes Hunter
- Combined-FDR p-value-integration results across individual methods

## Outputs

- auc_comparison_fig2.csv — AUC per method per simulated logFC condition in a single results table
- Relative method ranking (trends matching Fig. 2) showing combined-FDR integration performance vs. individual methods

## How to apply

Load the A. thaliana-derived simulated count matrices together with their ground-truth differentially expressed gene labels from the upstream simulation output, then run the individual DEG detection methods DESeq2, limma and NOISeq on each simulated dataset through the DEgenes Hunter pipeline within the ExpHunter Suite. Compute the combined-FDR calls by p-value integration across the individual method results. Rank genes by each method's significance measure and compute the AUC against the simulated ground truth for every simulated logFC condition. Assemble the AUC per method per condition into a single results table saved as auc_comparison_fig2.csv, and check that the relative method ordering matches the trends of Fig. 2 — the rationale being that no single method performs best in all simulated scenarios, but the combined-FDR integration tends to perform well in general.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package providing the DEgenes Hunter pipeline and auxiliary scripts for assessing performance and simulating RNA-seq data used to generate the ground-truth simulated datasets) — https://bioconductor.org/packages/ExpHunterSuite
- **DEgenes Hunter** (Pipeline within ExpHunter Suite used to run DESeq2, limma and NOISeq on each simulated dataset before p-value integration into combined-FDR calls) — https://github.com/seoanezonjic/ExpHunterSuite
- **DESeq2** (Individual DEG detection method whose ranked significance measure is scored by AUC against the simulated ground truth)
- **limma** (Individual DEG detection method whose ranked significance measure is scored by AUC against the simulated ground truth)
- **NOISeq** (Individual DEG detection method whose ranked significance measure is scored by AUC against the simulated ground truth)
- **R** (Environment in which the AUC computation, p-value integration and results-table assembly are run modularly)

## Examples

```
bash launch.sh && bash get_results.sh  # run ExpHunterSuiteExamples synthetic experiments, then generate the HTML results report
```

## Evaluation signals

- auc_comparison_fig2.csv contains one AUC value per method (DESeq2, limma, NOISeq, combined-FDR) per simulated logFC condition
- No single DEG method achieves the top AUC in every simulated scenario — a single method dominating all conditions indicates a pipeline error
- The combined-FDR integration method ranks near the top across most simulated conditions, matching the general trend of Fig. 2
- AUC values fall in the 0–1 range and are non-trivially above 0.5 for realistic simulated logFC conditions
- Relative method ordering in the assembled table is consistent with the Fig. 2 trends before publication

## Limitations

- Conclusions are tied to the A. thaliana-derived simulation design; the article notes that method behaviour and thresholds 'may differ for other organisms, tissues and experimental designs' and the authors are 'hesitant to extrapolate these findings to a more general rule'
- AUC evaluates ranking quality only — it does not assess logFC estimation accuracy, whose 'incorrect estimation could lead to poor target selection' and affect downstream analyses such as GSEA
- Only DESeq2, limma and NOISeq are run individually through the pipeline in this comparison, so the AUC table does not cover every combination method (e.g. the Naïve Bayes classifier or vote thresholds)

## Evidence

- [intro] auxiliary scripts for assessing performance and simulating RNA-seq data: "the ExpHunter Suite, implemented as an R/Bioconductor package including auxiliary scripts for assessing performance and simulating RNA-seq data"
- [abstract] combining methods is optimal in simulations: "Application to simulated and spike-in experiments shows that combining multiple methods using consensus and classifiers leads to optimal results"
- [intro] methods behave differently by data properties: "some widely-used expression analysis methods can behave quite differently depending on the properties of the data, potentially over-predicting or estimating inaccurate values"
- [readme] code and reports repository: [evidence span withheld — anchor status: external_doc]
- [readme] citation of the ExpHunterSuite paper: "Gene expression analysis method integration and co-expression module detection applied to rare glucide metabolism disorders using ExpHunterSuite"
