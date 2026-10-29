---
name: precision-recall-metric-computation
description: Use when a set of predicted genomic intervals (e.g., polyA-site bins
  from aptardi output GTF at probability >= 0.5) must be scored for precision and
  recall against annotated reference databases such as PolyASite 2.0 or PolyA_DB 3.
when_to_use_negative:
- Predictions have not yet been thresholded — aptardi bins below the probability threshold
  (default 0.5) must be excluded before computing true/false positives
- The evaluation target is the original sample-specific PolyA-Seq testing set rather
  than an aggregated database reference (PolyASite 2.0 / PolyA_DB 3)
- No interval-overlap ground truth exists (e.g., expression-only filtering tasks like
  removing transcripts with <1 count in 2/3 of samples)
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3512
tools:
- name: aptardi
  role: Produces the predicted polyA-site bins (100-base bins, probability threshold
    0.5) in its output GTF that are scored for precision and recall
  repo: https://github.com/luskry/aptardi
- name: BEDtools
  role: Intersects predicted positive bins (converted to BED intervals) with PolyASite
    2.0 and PolyA_DB 3 annotations to classify true and false positives
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
- name: StringTie
  role: Supplies the input transcriptome GTF whose 3' terminal exons define the bins
    evaluated by aptardi and keyed in the BED intervals
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:2be50d90cc07b606ac08369f11b7abab8142766d29404317ebc7e1faac099200
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - BEDtools
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# precision-recall-metric-computation

## Summary

Computes precision and recall for predicted genomic intervals (e.g., aptardi polyA-site bins) against a ground-truth reference using interval intersection (BEDtools) and per-database counting, as done when evaluating aptardi against PolyASite 2.0 and PolyA_DB 3.

## When to use

Use when a set of predicted genomic intervals (e.g., polyA-site bins from aptardi output GTF at probability >= 0.5) must be scored for precision and recall against annotated reference databases such as PolyASite 2.0 or PolyA_DB 3.

## When NOT to use

- Predictions have not yet been thresholded — aptardi bins below the probability threshold (default 0.5) must be excluded before computing true/false positives
- The evaluation target is the original sample-specific PolyA-Seq testing set rather than an aggregated database reference (PolyASite 2.0 / PolyA_DB 3)
- No interval-overlap ground truth exists (e.g., expression-only filtering tasks like removing transcripts with <1 count in 2/3 of samples)

## Inputs

- aptardi output GTF containing predicted polyA-site bins with probabilities
- PolyASite 2.0 polyA-site annotation (569,005 sites)
- PolyA_DB 3 annotation (289,998 sites)
- BED-format predicted positive intervals keyed to analyzed 3' terminal exons

## Outputs

- Per-database precision and recall values (e.g., aptardi_evaluation_metrics.csv)
- Per-database bin counts: analyzed bins, true positives, false positives
- Classification of predicted bins as true-positive or false-positive per reference database

## How to apply

Extract predicted positive bins from the tool output (for aptardi, bins with probability >= the default 0.5 threshold in the output GTF) and convert them to BED-format intervals keyed to the analyzed 3' terminal exons. Intersect these BED intervals with each reference annotation (PolyASite 2.0, PolyA_DB 3) using BEDtools to classify true-positive and false-positive bins, and record per-database counts of analyzed bins, true positives, and false positives (the HBR evaluation analyzed 102,774 bins against PolyASite 2.0's 569,005 sites and 86,685 bins against PolyA_DB 3's 289,998 sites). Compute precision = TP / (TP + FP) and recall = TP / (TP + reference positives) for each database. Validate by checking the metrics reproduce reported values within a small numeric tolerance (HBR aptardi: precision 0.85, recall 0.18 vs PolyASite 2.0; precision 0.88, recall 0.22 vs PolyA_DB 3) and that per-database bin counts are reproduced; write metrics to a CSV such as aptardi_evaluation_metrics.csv.

## Related tools

- **aptardi** (Produces the predicted polyA-site bins (100-base bins, probability threshold 0.5) in its output GTF that are scored for precision and recall) — https://github.com/luskry/aptardi
- **BEDtools** (Intersects predicted positive bins (converted to BED intervals) with PolyASite 2.0 and PolyA_DB 3 annotations to classify true and false positives) — https://bedtools.readthedocs.io/en/latest/content/installation.html
- **StringTie** (Supplies the input transcriptome GTF whose 3' terminal exons define the bins evaluated by aptardi and keyed in the BED intervals)

## Examples

```
bedtools intersect -u -a predicted_positive_bins.bed -b polyasite_2.0.bed > true_positive_bins.bed; bedtools intersect -v -a predicted_positive_bins.bed -b polyasite_2.0.bed > false_positive_bins.bed
```

## Evaluation signals

- Precision and recall per database match reported values within a small numeric tolerance: precision 0.85 / recall 0.18 for PolyASite 2.0 and precision 0.88 / recall 0.22 for PolyA_DB 3 on HBR
- Analyzed-bin counts are reproduced: 102,774 bins against PolyASite 2.0 and 86,685 bins against PolyA_DB 3
- Every predicted bin is classified as exactly one of true positive or false positive per database (TP + FP equals the number of predicted positive bins)
- Only bins with probability >= 0.5 from the aptardi output GTF are counted as predictions
- All metrics and per-database counts are written to aptardi_evaluation_metrics.csv

## Limitations

- aptardi predicts on 100 base bins — a resolution limitation — so precision/recall tolerate only sites within the annotated bin (polyA sites wiggle up to 30 nucleotides and the stop site is annotated as the 3'-most base of the bin)
- Recall against aggregated databases (0.18 and 0.22) is much lower than against the sample-specific PolyA-Seq testing set because database sites not expressed in the sample cannot be detected
- Only 3' terminal exons are analyzed, so coding APA events are excluded from the evaluation universe

## Evidence

- [other] precision 0.85 and recall 0.18 for PolyASite 2.0, and precision 0.88 and recall 0.22 for PolyA_DB 3: [evidence span withheld — anchor status: not_found]
- [other] Intersect the predicted bins with PolyASite 2.0 polyA-site annotations using BEDtools: [evidence span withheld — anchor status: not_found]
- [abstract] The average precision of the aptardi model is twice that of a standard transcriptome assembler.: "The average precision of the aptardi model is twice that of a standard transcriptome assembler."
- [discussion] aptardi makes predictions on 100 base bins: "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
- [readme] Probability threshold default: "--p/-p <float>			Probability threshold, predictions >= threshold are labeled transription stop site (default: 0.5, value must be constrained by (0, 1))"
