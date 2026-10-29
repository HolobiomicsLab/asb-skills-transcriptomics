---
name: confusion-matrix-metric-computation
description: Use when a benchmark provides ground-truth labels (e.g., simulated DEG
  genes in the 40%-doublet DataDEG dataset) and you must quantify how well a method's
  binary calls match truth via TPR, TNR, and accuracy.
when_to_use_negative:
- When no ground-truth labels exist (real experimental data without simulated DEGs
  or labelled doublets) — TPR/TNR/accuracy cannot be computed and proxy metrics are
  needed instead
- When the task is doublet detection itself rather than scoring downstream DEG accuracy
  against reference labels
- When predictions are continuous scores rather than binary calls and no threshold
  is defined — use ROC/PR analysis instead of raw confusion-matrix metrics
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: Chord
  role: Produces the doublet-filtered dataset whose downstream DEG calls are scored
    with TPR/TNR/accuracy against clean-data reference results
  repo: https://github.com/13308204545/Chord
- name: DoubletFinder
  role: Competing method whose top-40%-score-filtered DEG results are scored as a
    baseline; run with tuned pK and default pN = 0.25
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Provides cxds/bcds doublet scores used to filter cells; the resulting DEG
    calls are scored with the same metrics
  repo: https://github.com/kostkalab/scds
- name: MAST
  role: Differential expression framework (with Wilcoxon rank-sum) whose DEG calls
    supply the positive/negative predictions for metric computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:45fc40b40ef646138c5c060a48d5758bf480663af4d53755866d4a35eadd1f80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletFinder
  - MAST
  - R
  - adabag
  - gbm
  - remotes
  - scds
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# confusion-matrix-metric-computation

## Summary

Compute TPR, TNR, and accuracy of binary calls (e.g., DEG or doublet classifications) against known ground-truth labels, as used in the Chord paper to score DEG detection after doublet removal on the synthetic DataDEG dataset.

## When to use

Use when a benchmark provides ground-truth labels (e.g., simulated DEG genes in the 40%-doublet DataDEG dataset) and you must quantify how well a method's binary calls match truth via TPR, TNR, and accuracy.

## When NOT to use

- When no ground-truth labels exist (real experimental data without simulated DEGs or labelled doublets) — TPR/TNR/accuracy cannot be computed and proxy metrics are needed instead
- When the task is doublet detection itself rather than scoring downstream DEG accuracy against reference labels
- When predictions are continuous scores rather than binary calls and no threshold is defined — use ROC/PR analysis instead of raw confusion-matrix metrics

## Inputs

- Binary DEG call vector per gene per condition (e.g., from Wilcoxon rank-sum and MAST tests on filtered SingleCellExperiment objects)
- Known simulated-DEG ground-truth labels for the synthetic DataDEG dataset
- Optionally, binary doublet predictions (e.g., from outname_doublets.csv) to be evaluated against labelled doublets

## Outputs

- Table of TPR, TNR, and accuracy per method/condition (Chord, DoubletFinder, scds, contaminated baseline, clean reference)

## How to apply

After each filtering condition (e.g., removing the top 40% of cells ranked by each method's doublet score on the 1,667-cell scDesign-simulated DataDEG dataset), rerun differential expression testing with Wilcoxon rank-sum and MAST on the filtered data. Convert each DEG call set into a binary positive/negative vector aligned by gene to the known simulated-DEG labels, then compute true positive rate, true negative rate, and accuracy per method. Always compute the same metrics for the unfiltered contaminated data as a baseline and for the clean no-doublet data as the reference target. Validation: the correct outcome pattern is that Chord's filtered dataset yields TPR/TNR/accuracy closest to the clean-data reference, better than the contaminated data, and better than DoubletFinder or scds (cxds/bcds) alone.

## Related tools

- **Chord** (Produces the doublet-filtered dataset whose downstream DEG calls are scored with TPR/TNR/accuracy against clean-data reference results) — https://github.com/13308204545/Chord
- **DoubletFinder** (Competing method whose top-40%-score-filtered DEG results are scored as a baseline; run with tuned pK and default pN = 0.25) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **scds** (Provides cxds/bcds doublet scores used to filter cells; the resulting DEG calls are scored with the same metrics) — https://github.com/kostkalab/scds
- **MAST** (Differential expression framework (with Wilcoxon rank-sum) whose DEG calls supply the positive/negative predictions for metric computation)

## Examples

```
R> tpr <- sum(pred_degs %in% true_degs)/length(true_degs); tnr <- sum(!(pred_degs %in% true_degs))/(n_genes-length(true_degs)); accuracy <- (tpr*n_true + tnr*n_nonDE)/n_genes
```

## Evaluation signals

- TPR, TNR, and accuracy values must fall in [0, 1] and be computed on gene sets aligned by identifier to the simulated-DEG labels
- Contaminated-data baseline should show worse TPR/TNR/accuracy than any doublet-removed condition on the 40%-doublet DataDEG data
- Chord-filtered results should yield TPR/TNR/accuracy values closest to the clean (no-doublet) reference results
- Chord, DoubletFinder, and DoubletDetection should produce closer-to-clean metrics than the weaker competing methods
- Same DEG testing protocol (Wilcoxon rank-sum and MAST) must be applied identically across all filtered and baseline conditions for metrics to be comparable

## Limitations

- Metrics require known ground-truth labels, so this evaluation is only directly applicable to synthetic data like the scDesign-simulated DataDEG dataset
- Chord is not optimal in runtime because it must integrate the results of multiple methods
- Heterotypic doublets have the greatest impact on differential expression, so accuracy patterns may differ for homotypic doublets
- Other pre-processing steps, e.g., ambient mRNA removal, are needed to improve downstream analysis accuracy but were not incorporated

## Evidence

- [abstract] Chord had higher accuracy and stability than the individual approaches on different datasets containing real and synthetic data.: "Chord had higher accuracy and stability than the individual approaches on different datasets containing real and synthetic data."
- [intro] Heterotypic doublets affect downstream DE analysis: "heterotypic doublets are considered to have more impact on downstream analyses, including dimensionality reduction, cell clustering, differential expression"
- [readme] Doublet barcodes output file used for filtering before re-running DE tests: "A:The doublets' barcodes are in the file "outname_doublets.csv""
- [readme] DoubletFinder default pN used in the benchmark: "dbf.pN  The number of generated artificial doublets, expressed as a proportion of the merged real-artificial data. Default is set to 0.25"
- [discussion] Runtime limitation of the ensemble whose outputs are being scored: "Chord requires the integration of the results of multiple methods, so it is not optimal in terms of time efficiency."
