---
name: genomic-interval-intersection
description: Use when predicted genomic features (e.g., aptardi polyA-site bins in
  BED format) must be validated against reference annotation intervals (PolyASite
  2.0, PolyA_DB 3) to derive TP/FP classifications and precision/recall metrics.
when_to_use_negative:
- Input prediction features are not genomic intervals (e.g., raw BAM alignments or
  unprocessed GTF exons that have not been reduced to predicted-positive 100-base
  bins)
- The evaluation target is the sample-specific PolyA-Seq testing set rather than consolidated
  polyA site databases, since a different matching protocol applies
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0769
tools:
- name: BEDtools
  role: Intersects predicted-positive polyA-site BED intervals with PolyASite 2.0
    and PolyA_DB 3 annotations to classify true-positive and false-positive bins
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
- name: aptardi
  role: Produces the predicted polyA-site bins (100-base bins, threshold 0.5) in the
    output GTF that are converted to BED intervals for intersection
  repo: https://github.com/luskry/aptardi
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:46086fabb2108701056766ae763f1db4abff4f8f701d6949f9814c37be7073f4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BEDtools
  - Bedtools
  - aptardi
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- genomic-interval-set-operations
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Genomic interval intersection

## Summary

Uses BEDtools set operations to intersect predicted polyA-site bins from an aptardi output GTF against curated polyA site databases (PolyASite 2.0, PolyA_DB 3) on hg38, classifying bins as true or false positives to compute precision and recall.

## When to use

Use when predicted genomic features (e.g., aptardi polyA-site bins in BED format) must be validated against reference annotation intervals (PolyASite 2.0, PolyA_DB 3) to derive TP/FP classifications and precision/recall metrics.

## When NOT to use

- Input prediction features are not genomic intervals (e.g., raw BAM alignments or unprocessed GTF exons that have not been reduced to predicted-positive 100-base bins)
- The evaluation target is the sample-specific PolyA-Seq testing set rather than consolidated polyA site databases, since a different matching protocol applies

## Inputs

- aptardi output GTF containing predicted polyA-site bins with probabilities
- PolyASite 2.0 polyA-site annotation on hg38 (569,005 sites)
- PolyA_DB 3 polyA-site annotation on hg38 (289,998 sites)
- Predicted-positive bins (probability >= 0.5) converted to BED-format genomic intervals keyed to 3' terminal exons

## Outputs

- Per-database true-positive and false-positive bin classifications
- Precision and recall values per reference database (PolyASite 2.0: precision 0.85, recall 0.18; PolyA_DB 3: precision 0.88, recall 0.22)
- aptardi_evaluation_metrics.csv with computed metrics and per-database bin counts

## How to apply

Run aptardi in Mode 1 with the pre-built HBR model (model.hdf5 and scale.pk) on a sorted BAM, hg38 FASTA, and StringTie GTF using default parameters (100-base prediction bins, probability threshold 0.5), then extract predicted positive bins (probability >= 0.5) from the output GTF and convert them to BED-format intervals keyed to the analyzed 3' terminal exons. Intersect these BED intervals with PolyASite 2.0 annotations using BEDtools, classifying overlapping bins as true positives and non-overlapping bins as false positives. Repeat the identical intersection against PolyA_DB 3 annotations to obtain the alternative-reference evaluation. Compute precision, recall, and per-database bin counts (analyzed bins, true positives, false positives) for each reference; the HBR evaluation expected PolyASite 2.0 databases contained 569,005 sites yielding 102,774 bins and PolyA_DB 3 contained 289,998 sites yielding 86,685 bins.

## Related tools

- **BEDtools** (Intersects predicted-positive polyA-site BED intervals with PolyASite 2.0 and PolyA_DB 3 annotations to classify true-positive and false-positive bins) — https://bedtools.readthedocs.io/en/latest/content/installation.html
- **aptardi** (Produces the predicted polyA-site bins (100-base bins, threshold 0.5) in the output GTF that are converted to BED intervals for intersection) — https://github.com/luskry/aptardi

## Examples

```
bedtools intersect -u -a predicted_positive_bins.bed -b polyasite_hg38.bed > true_positive_bins.bed
```

## Evaluation signals

- Computed precision and recall for PolyASite 2.0 match the reported values (precision 0.85, recall 0.18) within a small numeric tolerance
- Computed precision and recall for PolyA_DB 3 match the reported values (precision 0.88, recall 0.22) within a small numeric tolerance
- Per-database bin counts are reproduced: 102,774 analyzed bins against PolyASite 2.0 (569,005 sites) and 86,685 bins against PolyA_DB 3 (289,998 sites)
- All metrics and per-database counts are written to aptardi_evaluation_metrics.csv with consistent TP + FP = analyzed bins per database

## Limitations

- Recall against consolidated databases is substantially lower (0.18–0.22) than against the sample-specific PolyA-Seq testing set because database sites sequenced at much greater depths may not be detectable in shallower RNA-Seq
- aptardi makes predictions on 100-base bins, so bins may contain multiple true sites or sites wiggle up to ~30 nucleotides within a bin
- Only 3' terminal exons of input transcripts are analyzed, so upstream coding APA sites cannot be evaluated

## Evidence

- [other] Extract predicted positive bins (probability >= 0.5) from the aptardi output GTF and convert them to BED-format genomic intervals keyed to the analyzed 3' terminal exons.: [evidence span withheld — anchor status: not_found]
- [other] precision 0.85 and recall 0.18 for PolyASite 2.0, and precision 0.88 and recall 0.22 for PolyA_DB 3: [evidence span withheld — anchor status: not_found]
- [other] which contained 569,005 and 289,998 annotated sites yielding 102,774 and 86,685 bins: [evidence span withheld — anchor status: not_found]
- [other] Intersect the predicted bins with PolyASite 2.0 polyA-site annotations using BEDtools to classify true-positive and false-positive bins.: [evidence span withheld — anchor status: not_found]
- [discussion] aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin): "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base"
