---
name: p-value-integration-combination
description: Use when you need robust DEG detection from RNA-seq count matrices and
  no single DE method can be assumed appropriate, since methods 'behave quite differently
  depending on the properties of the data'.
when_to_use_negative:
- When a single method's assumptions are known to fit the experimental design and
  the goal is interpretability of one method's statistics rather than robust consensus
  calling
- When extrapolating the optimal vote threshold (2 for spike-in data) to other organisms,
  tissues, or experimental designs — the authors are 'hesitant to extrapolate these
  findings to a more general rule'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package hosting the DEgenes Hunter pipeline and the method-integration
    (combined-FDR, vote threshold, Naïve Bayes) functionality
  repo: https://bioconductor.org/packages/ExpHunterSuite
- name: DEgenes Hunter
  role: pipeline that runs individual DEG methods and integrates their results via
    p-value integration
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DESeq2
  role: individual DEG detection method whose p-values are integrated; most conservative
    in the Lafora experiment
- name: limma
  role: individual DEG detection method whose p-values are integrated; showed higher
    precision
- name: NOISeq
  role: individual DEG detection method whose p-values are integrated; showed high
    recall
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:3333b18846c39232e68518d2cffea66be29e2ddf6b841889e27f7979f86b075f
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# p-value-integration-combination

## Summary

Combines DEG calls from multiple differential expression methods (DESeq2, edgeR, limma, NOISeq) by integrating their p-values within the DEgenes Hunter pipeline, producing a combined-FDR ranking that is more robust than any single method across simulated datasets.

## When to use

Use when you need robust DEG detection from RNA-seq count matrices and no single DE method can be assumed appropriate, since methods 'behave quite differently depending on the properties of the data'.

## When NOT to use

- When a single method's assumptions are known to fit the experimental design and the goal is interpretability of one method's statistics rather than robust consensus calling
- When extrapolating the optimal vote threshold (2 for spike-in data) to other organisms, tissues, or experimental designs — the authors are 'hesitant to extrapolate these findings to a more general rule'

## Inputs

- Simulated or real RNA-seq count matrices (e.g. A. thaliana-derived simulated count matrices with ground-truth DEG labels)
- Table of counts produced by upstream RNA-seq initial data analysis
- Per-method significance measures (p-values / FDR) from DESeq2, edgeR, limma, NOISeq runs

## Outputs

- Combined-FDR DEG calls per gene (p-value integration across methods)
- AUC per method per simulated logFC condition (e.g. auc_comparison_fig2.csv results table)
- DEgenes Hunter HTML results report

## How to apply

Run the individual DEG detection methods (DESeq2, limma, NOISeq) on the count matrix through the DEgenes Hunter pipeline within the ExpHunter Suite, then compute combined-FDR calls by p-value integration across the individual method results. Rank genes by each method's significance measure and evaluate performance (AUC) against ground truth when available — on simulated A. thaliana-derived datasets no single method won in all scenarios, but the combined-FDR integration tended to perform well overall (Fig. 2). As a complementary or alternative integration, DEgenes Hunter also supports combining methods by vote threshold; for spike-in data the optimum minimum vote threshold was 2 in terms of F-measure. Install the suite as an R/Bioconductor package via devtools from GitHub, noting reactome.db may need an extended timeout.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package hosting the DEgenes Hunter pipeline and the method-integration (combined-FDR, vote threshold, Naïve Bayes) functionality) — https://bioconductor.org/packages/ExpHunterSuite
- **DEgenes Hunter** (pipeline that runs individual DEG methods and integrates their results via p-value integration) — https://github.com/seoanezonjic/ExpHunterSuite
- **DESeq2** (individual DEG detection method whose p-values are integrated; most conservative in the Lafora experiment)
- **limma** (individual DEG detection method whose p-values are integrated; showed higher precision)
- **NOISeq** (individual DEG detection method whose p-values are integrated; showed high recall)

## Examples

```
Rscript -e 'options(timeout=1000); devtools::install_github("seoanezonjic/ExpHunterSuite", dependencies=TRUE)'
```

## Evaluation signals

- Relative method ordering in the AUC-per-condition table matches the trends of Fig. 2, with combined-FDR tending to perform well across simulated logFC conditions
- Combined-FDR and other integration strategies lead to optimal results on simulated and spike-in experiments, outperforming the best single method on the spike-in dataset
- When using vote-threshold combination on spike-in data, F-measure peaks at a minimum vote threshold of 2
- Running launch.sh followed by get_results.sh generates an HTML results report without errors
- Conservative/precision-recall behaviour of individual methods (DESeq2 conservative, NOISeq high recall, limma higher precision) is consistent with reported patterns

## Limitations

- The optimal vote threshold (2 on spike-in data) may not generalize to other organisms, tissues, and experimental designs; the authors hesitate to extrapolate a general rule
- Comparison against a competing recommendation of much higher thresholds was based on the analysis of a single human dataset
- No single DEG detection method performs best in all simulated scenarios, so single-method results should not be trusted blindly
- More elaborate classifiers than the Naïve Bayes approach are left for future work

## Evidence

- [results] No single method finding: [evidence span withheld — anchor status: not_found]
- [intro] Data-dependent method behaviour: "some widely-used expression analysis methods can behave quite differently depending on the properties of the data, potentially over-predicting or estimating inaccurate values"
- [intro] Previous combination strategies: "Previous studies have looked at combining results of multiple DEG methods to improve DEG detection, using consensus and ranking based strategies as well as p-value integration"
- [discussion] Vote threshold optimum: "for the spike-in data the optimum threshold was 2 in terms of F-measure"
- [readme] Installation command: [evidence span withheld — anchor status: external_doc]
- [abstract] Integration performance claim: "Application to simulated and spike-in experiments shows that combining multiple methods using consensus and classifiers leads to optimal results"
