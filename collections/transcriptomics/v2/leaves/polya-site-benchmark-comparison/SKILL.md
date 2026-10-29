---
name: polya-site-benchmark-comparison
description: Use when evaluating predicted polyA-site bins from an aptardi output
  GTF (or a StringTie/Ensembl baseline transcriptome) against curated polyA-site databases
  such as PolyASite 2.0, PolyA_DB 3, or PolyA-Seq tissue-specific ground truth.
when_to_use_negative:
- Direct 3′-end sequencing data (e.g., PolyA-Seq) is available for the sample itself
  — aptardi is designed to predict sites from RNA-Seq plus DNA sequence when 3′-sequencing
  data are not available
- The analysis targets coding APA in upstream exons — aptardi only evaluates 3' terminal
  exons, so upstream events cannot be benchmarked with this skill
- Only a priori annotated polyA sites are needed (e.g., MISO/QAPA/PARQ workflows)
  — there is no de novo prediction to validate against databases
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: aptardi
  role: Predicts polyA-site bins from RNA-Seq coverage and DNA sequence via the pre-built
    HBR model (Mode 1); its output GTF supplies the predicted positive bins being
    benchmarked
  repo: https://github.com/luskry/aptardi
- name: BEDtools
  role: Intersects predicted 100-base bins with PolyASite 2.0 / PolyA_DB 3 annotations
    to classify true-positive and false-positive bins
- name: StringTie
  role: Assembles the baseline transcriptome (--rf -G with Ensembl guide) against
    which aptardi-added unannotated sites are counted and fold increase is computed
- name: RSEM
  role: Quantitates the aptardi transcriptome so transcripts without at least 1 count
    in 2/3 of samples can be removed before benchmarking tissue-specific sites
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:3318baec6f9dea4855b88a211b88d46aaaba98e114e5e9045086fa7727768311
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - BEDtools
  - HISAT2
  - RSEM
  - SAMtools
  - StringTie
  - aptardi
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# polya-site-benchmark-comparison

## Summary

Benchmarks predicted polyadenylation (polyA) sites from an aptardi-modified or assembler transcriptome against ground-truth references (PolyASite 2.0, PolyA_DB 3, Derti et al. PolyA-Seq) using BEDtools interval intersections, yielding precision/recall metrics and counts of unannotated tissue-specific sites. It establishes whether a machine-learning polyA predictor meaningfully extends the 3′-end content of a StringTie/Ensembl transcriptome.

## When to use

Use when evaluating predicted polyA-site bins from an aptardi output GTF (or a StringTie/Ensembl baseline transcriptome) against curated polyA-site databases such as PolyASite 2.0, PolyA_DB 3, or PolyA-Seq tissue-specific ground truth.

## When NOT to use

- Direct 3′-end sequencing data (e.g., PolyA-Seq) is available for the sample itself — aptardi is designed to predict sites from RNA-Seq plus DNA sequence when 3′-sequencing data are not available
- The analysis targets coding APA in upstream exons — aptardi only evaluates 3' terminal exons, so upstream events cannot be benchmarked with this skill
- Only a priori annotated polyA sites are needed (e.g., MISO/QAPA/PARQ workflows) — there is no de novo prediction to validate against databases

## Inputs

- aptardi output GTF with predicted polyA-site bins (Mode 1, HBR model.hdf5 + scale.pk)
- Sorted BAM of genome-aligned RNA-Seq reads (e.g., HISAT2-aligned, mm10 or hg38)
- StringTie GTF transcriptome (assembled with guide annotation, e.g., Ensembl mm10 with --rf -G)
- Reference polyA-site annotations: PolyASite 2.0, PolyA_DB 3, or Derti et al. tissue-specific PolyA-Seq
- RSEM transcript quantitation for the 2/3-samples expression filter

## Outputs

- Precision and recall per reference database (e.g., aptardi_evaluation_metrics.csv with per-database analyzed bins, true positives, false positives)
- Counts of unannotated tissue-specific polyA sites and fold increase over StringTie alone
- BED-format predicted positive bins (probability >= 0.5) classified as true/false positives per reference

## How to apply

Run aptardi in Mode 1 with the pre-built HBR model (model.hdf5 and scale.pk from src/aptardi/ml_scale) using default 100-base prediction bins and probability threshold 0.5 to obtain predicted polyA-site bins in the output GTF. Extract predicted positive bins (probability >= 0.5), convert them to BED-format genomic intervals keyed to analyzed 3' terminal exons, and intersect them with PolyASite 2.0 and PolyA_DB 3 annotations using BEDtools to classify true-positive and false-positive bins, then compute precision and recall per database. Before benchmarking tissue-specific discovery, quantitate the aptardi transcriptome with RSEM and apply the conservative filter removing transcripts without at least one count in 2/3 of samples, since aptardi annotates genomic regions to every overlapping isoform and filtering removes redundant false positives. For species/tissue applications (e.g., mouse brain vs liver, SRA PRJNA375882, aligned to mm10 with HISAT2 --rna-strandness RF --dta), compare aptardi-added 3' termini against the Derti et al. tissue-specific PolyA-Seq ground truth and count unannotated sites absent from both the StringTie and Ensembl (mm10) transcriptomes, reporting fold increase over StringTie alone.

## Related tools

- **aptardi** (Predicts polyA-site bins from RNA-Seq coverage and DNA sequence via the pre-built HBR model (Mode 1); its output GTF supplies the predicted positive bins being benchmarked) — https://github.com/luskry/aptardi
- **BEDtools** (Intersects predicted 100-base bins with PolyASite 2.0 / PolyA_DB 3 annotations to classify true-positive and false-positive bins)
- **StringTie** (Assembles the baseline transcriptome (--rf -G with Ensembl guide) against which aptardi-added unannotated sites are counted and fold increase is computed)
- **RSEM** (Quantitates the aptardi transcriptome so transcripts without at least 1 count in 2/3 of samples can be removed before benchmarking tissue-specific sites)

## Examples

```
aptardi --o out --f hg38.fa --r stringtie.gtf --b sorted.bam --n model.hdf5 --t scale.pk --g aptardi.gtf; bedtools intersect -a predicted_bins.bed -b PolyASite2.0.bed -u
```

## Evaluation signals

- Computed precision/recall per database match reported values within tolerance: PolyASite 2.0 precision 0.85, recall 0.18; PolyA_DB 3 precision 0.88, recall 0.22
- Per-database bin counts reproduce: PolyASite 2.0 and PolyA_DB 3 contained 569,005 and 289,998 annotated sites yielding 102,774 and 86,685 analyzed bins
- Mouse brain analysis reproduces 26 unannotated brain-specific sites (fold increase 9 vs StringTie's 3) and liver analysis 69 unannotated liver-specific sites (fold increase 17 vs StringTie's 4)
- Metrics file (aptardi_evaluation_metrics.csv) contains precision, recall, and per-database analyzed/true-positive/false-positive bin counts
- Aptardi precision should exceed a standard assembler's (reported as roughly twice, with recall improved over three-fold)

## Limitations

- aptardi predicts on 100-base bins and annotates the transcript stop site as the 3' most base in the bin, so benchmarked positions have limited resolution compared with nucleotide-level PolyA-Seq sites
- Many polyA sites in the deeper-sequenced PolyA-Seq reference were likely not detectable in lower-depth RNA-Seq, deflating recall (e.g., modest mouse tissue site counts at low sequencing depth)
- aptardi annotates a genomic polyA site to every overlapping transcript isoform, inflating redundant/false-positive aptardi transcripts unless the 2/3-samples expression filter is applied; manually engineered DNA sequence features may also not apply outside mammals

## Evidence

- [abstract] The average precision of the aptardi model is twice that of a standard transcriptome assembler.: "The average precision of the aptardi model is twice that of a standard transcriptome assembler."
- [discussion] many polyA sites reported in the PolyA-Seq data—which was sequenced at much greater depths—were likely not detectable here: "many polyA sites reported in the PolyA-Seq data—which was sequenced at much greater depths—were likely not detectable here"
- [discussion] aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin): "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
- [readme] Since aptardi identifies genomic regions to annotate polyA sites, we HIGHLY RECOMMEND filtering to remove redundant, i.e. false positive, aptardi transcripts: "Since aptardi identifies **genomic** regions to annotate polyA sites, we HIGHLY RECOMMEND filtering to remove redundant, i.e. false positive, aptardi transcripts"
- [readme] we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
