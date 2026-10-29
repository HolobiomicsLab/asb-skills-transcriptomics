---
name: bsj-overlap-relation-index-computation
description: Use when evaluating how overlapping BSJ sites from CircAST output (or
  similar circRNA lists) affect full-length assembly sensitivity/precision and FPKM/TPM
  quantification accuracy, e.g. in simulation benchmarking.
when_to_use_negative:
- Input consists of raw FASTQ/back-splice junction reads rather than an assembled
  circRNA transcript table — RI analysis requires the CircAST output with genomic
  coordinates and FPKM/TPM columns.
- The circRNAs under study are intronic or intergenic — the current CircAST pipeline
  only assembles exon-annotated circular transcripts, so RI stratification of such
  loci is not supported.
- Evaluating quantification in non-RNase R-treated samples, where remnant linear mRNA
  can generate false transcripts and confound the accuracy metrics.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: CircAST
  role: Assembles full-length circular transcripts and estimates FPKM/TPM expression
    whose accuracy is stratified by the computed relation index; run downstream of
    BSJ detection (CIRCexplorer2/CIRI2/UROBORUS) on tophat2-sorted SAM input
  repo: https://github.com/xiaofengsong/CircAST
- name: pandas
  role: Loads the CircAST output table and implements the BSJ overlap-relation grouping
    and RI-bin stratification for the metrics computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:5a93e9d1eee28120edcdd85a2a98626583498a9ea59dbf0a7c52e715a3d53e6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CircAST
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bsj-overlap-relation-index-computation

## Summary

Computation of the relation index (RI) — the cardinality of a back-splice junction (BSJ) site's equivalence class under genomic-overlap relations — to stratify CircAST reconstruction and quantification accuracy on simulated circRNA data. It explains why assembly performance degrades when multiple circRNAs share overlapping genomic regions.

## When to use

Use when evaluating how overlapping BSJ sites from CircAST output (or similar circRNA lists) affect full-length assembly sensitivity/precision and FPKM/TPM quantification accuracy, e.g. in simulation benchmarking.

## When NOT to use

- Input consists of raw FASTQ/back-splice junction reads rather than an assembled circRNA transcript table — RI analysis requires the CircAST output with genomic coordinates and FPKM/TPM columns.
- The circRNAs under study are intronic or intergenic — the current CircAST pipeline only assembles exon-annotated circular transcripts, so RI stratification of such loci is not supported.
- Evaluating quantification in non-RNase R-treated samples, where remnant linear mRNA can generate false transcripts and confound the accuracy metrics.

## Inputs

- CircAST output table on simulated data (tab-separated: chromosome, start, end, circular transcript name, parental gene, strand, number of exons, exon sizes, exon offsets, junction read counts, FPKM, TPM, assigned reads)
- Simulated ground-truth circular transcript definitions with back-splice junction genomic coordinates
- Simulated RNA-seq alignments in name-sorted SAM format (accepted_hits.sorted.sam) as consumed by CircAST

## Outputs

- Per-site RI assignments (BSJ site to relation-index equivalence-class size mapping)
- RI-stratified performance metrics table (ri_performance_metrics.csv) with sensitivity, precision, and FPKM/TPM correlation per RI bin (RI 1-4 vs RI > 4)

## How to apply

Load the CircAST output table (bed-like columns: chromosome, start, end, circular transcript name, junction read counts, FPKM, TPM, assigned reads) with pandas. Build the BSJ overlap relation between simulated back-splice junction sites from their genomic coordinates (chromosome, start, end) and assign each site an RI equal to the cardinality of its equivalence class under this overlap relation. Stratify all simulated BSJ sites into RI bins (RI 1-4 vs RI > 4) and, per bin, compare CircAST-assembled circular transcripts against simulated ground-truth isoforms to compute sensitivity (fraction of true isoforms recovered) and precision (fraction of assembled isoforms that are true), plus the correlation between theoretical and CircAST-estimated expression (FPKM/TPM). The rationale is that CircAST's multiple splice graphs and EMPC/maximum-likelihood read allocation become harder to disambiguate as overlapping circRNAs multiply, so RI quantifies the assembly difficulty directly. Validate against the reported simulation ranges: sensitivity 85.63%-94.32%, precision 81.96%-87.55%, quantification correlation 0.85-0.99.

## Related tools

- **CircAST** (Assembles full-length circular transcripts and estimates FPKM/TPM expression whose accuracy is stratified by the computed relation index; run downstream of BSJ detection (CIRCexplorer2/CIRI2/UROBORUS) on tophat2-sorted SAM input) — https://github.com/xiaofengsong/CircAST
- **pandas** (Loads the CircAST output table and implements the BSJ overlap-relation grouping and RI-bin stratification for the metrics computation)

## Examples

```
import pandas as pd; df = pd.read_csv('CircAST_result.txt', sep='\t', names=['chrom','start','end','name','gene','strand','nexons','sizes','offsets','junction_reads','fpkm','tpm','assigned_reads'])
```

## Evaluation signals

- Sensitivity in the RI 1-4 bin stays above 93% and drops to roughly 86% for RI > 4, reproducing the reported pattern.
- Overall sensitivity falls within the published simulation range 85.63%-94.32% and precision within 81.96%-87.55%.
- Correlation between theoretical and CircAST-estimated expression (FPKM/TPM) per RI bin falls within the reported 0.85-0.99 range, declining as RI increases.
- Every simulated BSJ site receives exactly one RI value, and RI-bin site counts sum to the total number of simulated BSJ sites.
- Output ri_performance_metrics.csv contains both the per-site RI assignments and the RI-stratified metrics table without missing coordinate fields.

## Limitations

- CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over.
- Remnant linear mRNA in the samples can affect the performance of circular transcript assembly and generate false transcripts, inflating apparent precision/sensitivity error in high-RI bins.
- The current version of the CircAST pipeline may miss intronic or intergenic circular transcripts, so RI cannot be computed for those loci.

## Evidence

- [abstract] Sensitivity/precision ranges from the simulation study that RI stratification should reproduce: "CircAST correctly assembled the full sequences of circRNAs with a sensitivity of 85.63%-94.32% and a precision of 81.96%-87.55%"
- [abstract] Quantification accuracy metric used per RI bin: "CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values"
- [discussion] Assembly algorithm whose performance degrades with increasing RI: "used the EMPC algorithm on these multiple splice graphs to assemble the multiple circular transcripts"
- [readme] CircAST output columns consumed by the RI computation: "The first 3 columns are the same with bed file format."
- [readme] Accepted alignment input format for CircAST runs being stratified: "SAM file sotsorted by name can be obtained by the following commands"
