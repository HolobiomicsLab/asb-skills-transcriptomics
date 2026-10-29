---
name: differential-gene-expression-analysis
description: Use when you have a gene-level count table from an RNA-seq experiment
  (real, spike-in, or simulated) and need robust DEG calls whose method-specific assumptions
  are unknown or likely to matter.
when_to_use_negative:
- When only a single method's output is required and dataset properties (e.g. non-model
  organism count distributions) make one tool clearly appropriate — the multi-method
  integration adds overhead without benefit in that case
- When the input is already a DEG result table from all methods and the goal is only
  result integration (vote thresholds or combined-FDR), not DEG detection itself
- When the input is a raw FASTQ file — reads must first be converted and mapped to
  count tables via a clean-and-map program before DEgenes Hunter can run
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: ExpHunter Suite
  role: R/Bioconductor package providing the DEgenes Hunter pipeline, result combination
    strategies (vote thresholds, combined-FDR, Naive Bayes) and benchmarking utilities
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DEgenes Hunter
  role: Command-line pipeline that executes the individual DEG detection methods on
    the count table
  repo: https://github.com/seoanezonjic/ExpHunterSuite
- name: DESeq2
  role: Individual DEG detection method; most conservative on the Lafora disease experiment
    and a top F-measure performer (0.88) on spike-in data
- name: limma
  role: Individual DEG detection method; showed higher precision and best logFC correlation
    (mean 0.92) on spike-in data
- name: NOISeq
  role: Individual DEG detection method integrated into the pipeline; F-measure 0.74
    on spike-in data
- name: ExpHunterSuiteExamples
  role: Repository with code, launch scripts (launch_DEgenesHunter.sh, launch.sh)
    and evaluation scripts to reproduce the simulated, spike-in and rare-disease experiments
  repo: https://github.com/fmjabato/ExpHunterSuiteExamples
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-94343-w
    title: Gene expression analysis method integration and co-expression module detection
      applied to rare glucide metabolism disorders using ExpHunterSuite
  source_content_hash: sha256:1e4bd338a57c6512d22db10e72d7f813ce142b30ebb51689c63a2da15e4dc191
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seoanezonjic/ExpHunterSuite
  tools:
  - DESeq2
  - DEgenes Hunter
  - ExpHunter Suite
  - ExpHunterSuiteExamples
  - NOISeq
  - R
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# differential-gene-expression-analysis

## Summary

Detect differentially expressed genes (DEGs) from an RNA-seq count table by running multiple methods (DESeq2, limma, NOISeq) through the DEgenes Hunter pipeline in ExpHunter Suite and integrating their results via vote thresholds or combined-FDR data fusion.

## When to use

Use when you have a gene-level count table from an RNA-seq experiment (real, spike-in, or simulated) and need robust DEG calls whose method-specific assumptions are unknown or likely to matter.

## When NOT to use

- When only a single method's output is required and dataset properties (e.g. non-model organism count distributions) make one tool clearly appropriate — the multi-method integration adds overhead without benefit in that case
- When the input is already a DEG result table from all methods and the goal is only result integration (vote thresholds or combined-FDR), not DEG detection itself
- When the input is a raw FASTQ file — reads must first be converted and mapped to count tables via a clean-and-map program before DEgenes Hunter can run

## Inputs

- Gene-level RNA-seq count table (matrix of counts per gene per sample)
- Simulated count matrices with ground-truth differentially expressed / non-DE gene labels (e.g. A. thaliana-derived simulations from upstream simulation steps)
- Spike-in count tables derived from FASTQ via a clean-and-map step (synthetic mouse spike-in experiment from ExpHunterSuiteExamples)
- Known spike-in or simulated truth labels for metric computation

## Outputs

- Per-method DEG calls and significance rankings (DESeq2, limma, NOISeq)
- Combined DEG call sets from minimum-vote thresholds (1, 2, 3, 4) and the combined-FDR data-fusion criterion
- AUC-per-method-per-condition comparison table (e.g. auc_comparison_fig2.csv)
- Precision/recall/F-measure table per method and combination (e.g. deg_metrics_comparison.csv)
- Per-method logFC correlation table (spike-in benchmark; limma mean 0.92)

## How to apply

Install ExpHunter Suite as an R/Bioconductor package and run the DEgenes Hunter command-line pipeline (e.g. launch_DEgenesHunter.sh) on a count table to obtain per-method DEG results from DESeq2, limma and NOISeq. Because DE methods rest on different assumptions and 'some widely-used expression analysis methods can behave quite differently depending on the properties of the data', do not rely on a single method: combine results either by a minimum vote threshold cut-off (a gene is DE if at least k of the methods vote for it) or by the combined-FDR p-value integration criterion. The paper found the combined-FDR criterion behaves similarly to a vote threshold of 2, which was the F-measure optimum on simulated data, and on spike-in data both combined-FDR and threshold-2 reached F-measure 0.88 alongside DESeq2. For benchmarking, rank genes by each method's significance measure, compute AUC against ground truth labels (simulated datasets) or precision/recall/F-measure against spike-in truth, and write per-method metric tables to CSV.

## Related tools

- **ExpHunter Suite** (R/Bioconductor package providing the DEgenes Hunter pipeline, result combination strategies (vote thresholds, combined-FDR, Naive Bayes) and benchmarking utilities) — https://github.com/seoanezonjic/ExpHunterSuite
- **DEgenes Hunter** (Command-line pipeline that executes the individual DEG detection methods on the count table) — https://github.com/seoanezonjic/ExpHunterSuite
- **DESeq2** (Individual DEG detection method; most conservative on the Lafora disease experiment and a top F-measure performer (0.88) on spike-in data)
- **limma** (Individual DEG detection method; showed higher precision and best logFC correlation (mean 0.92) on spike-in data)
- **NOISeq** (Individual DEG detection method integrated into the pipeline; F-measure 0.74 on spike-in data)
- **ExpHunterSuiteExamples** (Repository with code, launch scripts (launch_DEgenesHunter.sh, launch.sh) and evaluation scripts to reproduce the simulated, spike-in and rare-disease experiments) — https://github.com/fmjabato/ExpHunterSuiteExamples

## Examples

```
Rscript -e 'options(timeout=1000); devtools::install_github("seoanezonjic/ExpHunterSuite")' && bash launch_DEgenesHunter.sh
```

## Evaluation signals

- Relative method ordering in AUC across simulated logFC conditions matches the trends of Fig. 2 (combined-FDR tends to perform well but no single method wins in all scenarios)
- Combined-FDR precision/recall/F-measure values are comparable to the minimum-vote threshold of 2, the optimum by F-measure on simulated data
- Spike-in F-measures fall within tolerance of Table 1 values: DESeq2 0.88, combined-FDR 0.88, threshold-2 0.88, limma 0.76, NOISeq 0.74
- Spike-in logFC correlation per method reproduces Table 2 (limma mean 0.92)
- All precision/recall/F-measure values in the comparison CSV are within [0,1] and per-method AUC metrics are in [0,1]

## Limitations

- The optimal minimum vote threshold cut-off may differ for other organisms, tissues and experimental designs, so the paper is 'hesitant to extrapolate these findings to a more general rule'; a competing recommendation of much higher thresholds was based on a single human dataset
- Individual methods behave inconsistently across datasets — DESeq2 was the most conservative method on the Lafora disease experiment but 'by far the least conservative' on the PMM2-CDG experiment — so single-method calls should not be trusted blindly
- The Naive Bayes classifier is trained on simulated data; more elaborate classifiers are left for future work, and incorrect fold-change estimation can degrade downstream logFC analyses such as GSEA and qPCR target selection

## Evidence

- [intro] DE methods are based on different assumptions and analysis procedures, making it impossible to know the most appropriate method for a given dataset: "DE methods are based on different assumptions and analysis procedures, making it impossible to know the most appropriate method for a given dataset"
- [intro] Rationale for combining methods rather than trusting one: "some widely-used expression analysis methods can behave quite differently depending on the properties of the data, potentially over-predicting or estimating inaccurate values"
- [discussion] Recommendation for the vote-threshold combination strategy: "we would suggest the use of a minimum vote threshold cut-off where possible"
- [discussion] Optimum threshold on spike-in data: "for the spike-in data the optimum threshold was 2 in terms of F-measure"
- [readme] Pipeline scope description from the package README: [evidence span withheld — anchor status: external_doc]
- [readme] Concrete pipeline launch step for the real-disease experiments: [evidence span withheld — anchor status: external_doc]
