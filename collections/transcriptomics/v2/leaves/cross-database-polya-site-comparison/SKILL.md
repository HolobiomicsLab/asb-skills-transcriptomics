---
name: cross-database-polya-site-comparison
description: Use when a set of predicted polyA sites or 3'-end bins must be benchmarked
  against public polyA site annotations (PolyASite, PolyA_DB) instead of a sample-specific
  gold standard.
when_to_use_negative:
- The reference set is sample-specific 3'-end sequencing data (e.g., the HBR PolyA-Seq
  testing set) rather than an aggregated database — aptardi's own evaluation workflow
  applies directly instead
- Input transcripts are not 3' terminal exons — aptardi only evaluates 3' terminal
  exons, so coding (upstream exon) APA events cannot be assessed
- Bin size was altered from the pre-built model's 100-base bins, making intersections
  with database sites non-comparable to reported metrics
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3511
tools:
- name: aptardi
  role: Predicts polyA-site bins in the sample-specific transcriptome using the pre-built
    HBR model (model.hdf5, scale.pk) in Mode 1
  repo: https://github.com/luskry/aptardi
- name: BEDtools
  role: Intersects predicted positive bins with PolyASite 2.0 and PolyA_DB 3 annotations
    to classify true positives and false positives
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
- name: StringTie
  role: Reconstructs the input transcriptome GTF whose 3' terminal exons aptardi analyzes
- name: SAMtools
  role: Produces the sorted BAM of aligned RNA-Seq reads used by aptardi
  repo: http://www.htslib.org/download/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:eb26f8746c588a6b697f8364a8771ece01d2abeae93b5f6b56d669b4ab5ce834
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - BEDtools
  - SAMtools
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Cross-database polyA site comparison

## Summary

Evaluates predicted polyadenylation sites (e.g., aptardi output bins) against independent consolidated polyA site databases such as PolyASite 2.0 and PolyA_DB 3 using BEDtools intersections, and computes precision/recall per reference.

## When to use

Use when a set of predicted polyA sites or 3'-end bins must be benchmarked against public polyA site annotations (PolyASite, PolyA_DB) instead of a sample-specific gold standard.

## When NOT to use

- The reference set is sample-specific 3'-end sequencing data (e.g., the HBR PolyA-Seq testing set) rather than an aggregated database — aptardi's own evaluation workflow applies directly instead
- Input transcripts are not 3' terminal exons — aptardi only evaluates 3' terminal exons, so coding (upstream exon) APA events cannot be assessed
- Bin size was altered from the pre-built model's 100-base bins, making intersections with database sites non-comparable to reported metrics

## Inputs

- aptardi output GTF with predicted polyA-site bins (probability >= 0.5 positives)
- Sorted BAM of aligned RNA-Seq reads (e.g., HBR sample)
- Reference genome FASTA (hg38.fa)
- StringTie-reconstructed transcriptome GTF
- PolyASite 2.0 polyA site annotations
- PolyA_DB 3 polyA site annotations

## Outputs

- BED-format genomic intervals of predicted positive bins keyed to analyzed 3' terminal exons
- Per-database precision, recall, and bin counts (analyzed bins, true positives, false positives)
- aptardi_evaluation_metrics.csv with computed metrics per reference database

## How to apply

Run the predictor (aptardi Mode 1 with the pre-built model.hdf5 and scale.pk) on the sorted BAM, hg38 FASTA, and StringTie GTF using default parameters (100-base bins, probability threshold 0.5) to obtain predicted polyA-site bins in the output GTF. Extract predicted positive bins (probability >= 0.5) and convert them to BED-format genomic intervals keyed to the analyzed 3' terminal exons. Intersect these bins with PolyASite 2.0 annotations using BEDtools to classify true-positive and false-positive bins, then repeat against PolyA_DB 3 annotations for the alternative-reference evaluation. Compute precision, recall, and per-database bin counts (analyzed bins, true positives, false positives) for each reference database — the reference study reported 569,005 PolyASite 2.0 and 289,998 PolyA_DB 3 annotated sites yielding 102,774 and 86,685 bins respectively. Validate by comparing computed metrics with reported values (PolyASite 2.0: precision 0.85, recall 0.18; PolyA_DB 3: precision 0.88, recall 0.22) within a small numeric tolerance. Expect higher precision but lower recall against aggregated databases than against the sample-specific PolyA-Seq testing set, because databases contain sites not expressed in the analyzed sample.

## Related tools

- **aptardi** (Predicts polyA-site bins in the sample-specific transcriptome using the pre-built HBR model (model.hdf5, scale.pk) in Mode 1) — https://github.com/luskry/aptardi
- **BEDtools** (Intersects predicted positive bins with PolyASite 2.0 and PolyA_DB 3 annotations to classify true positives and false positives) — https://bedtools.readthedocs.io/en/latest/content/installation.html
- **StringTie** (Reconstructs the input transcriptome GTF whose 3' terminal exons aptardi analyzes)
- **SAMtools** (Produces the sorted BAM of aligned RNA-Seq reads used by aptardi) — http://www.htslib.org/download/

## Examples

```
aptardi --o outdir --f hg38.fa --r stringtie.gtf --b sorted.bam --n model.hdf5 --t scale.pk -g aptardi_output.gtf -p 0.5; bedtools intersect -a predicted_bins.bed -b polyasite2.bed -u
```

## Evaluation signals

- Computed precision and recall per database match reported values within a small numeric tolerance (PolyASite 2.0: precision 0.85, recall 0.18; PolyA_DB 3: precision 0.88, recall 0.22)
- Per-database bin counts are reproduced (102,774 bins from PolyASite 2.0; 86,685 bins from PolyA_DB 3)
- Every predicted positive bin carries a probability >= 0.5 and maps to an analyzed 3' terminal exon
- Precision is higher and recall lower against aggregated databases than against the sample-specific PolyA-Seq HBR testing set
- BEDtools intersection output partitions all predicted bins into disjoint true-positive and false-positive sets per reference database

## Limitations

- aptardi makes predictions on 100 base bins (annotating the transcript stop site as the 3' most base in the bin); polyA sites wiggle up to 30 nucleotides, which affects site-level matching against database annotations
- Upstream exons are not subjected to aptardi analysis, eliminating the possibility of identifying coding APA, so cross-database evaluation only covers 3' terminal exon sites
- Database-based recall is low (0.18–0.22) because aggregated databases contain many sites not expressed in the analyzed sample or undetectable at the sample's sequencing depth

## Evidence

- [discussion] aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin): "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
- [readme] we DO NOT recommend altering this argument using the pre-built model: "we DO NOT recommend altering this argument using the pre-built model since this model was built using 100 base bins"
- [discussion] upstream exons are not subjected to aptardi analysis, effectively eliminating the possibility of identifying coding APA: "upstream exons are not subjected to aptardi analysis, effectively eliminating the possibility of identifying coding APA"
- [readme] Probability threshold, predictions >= threshold are labeled transription stop site (default: 0.5, value must be constrained by (0, 1)): "Probability threshold, predictions >= threshold are labeled transription stop site (default: 0.5, value must be constrained by (0, 1))"
- [discussion] many polyA sites reported in the PolyA-Seq data—which was sequenced at much greater depths—were likely not detectable here: "many polyA sites reported in the PolyA-Seq data—which was sequenced at much greater depths—were likely not detectable here"
