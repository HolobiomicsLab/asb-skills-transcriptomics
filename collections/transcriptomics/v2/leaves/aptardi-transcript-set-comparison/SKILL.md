---
name: aptardi-transcript-set-comparison
description: Use when you have RSEM quantitations for the same samples against two
  transcriptome GTFs (aptardi-modified and original) and need to identify which differentially
  expressed transcripts are attributable to aptardi-added 3'-end annotations.
when_to_use_negative:
- Working with non-mammalian taxa, where aptardi's manually engineered DNA sequence
  features may not apply
- Single-sample or very few-sample analyses where the '1 count in 2/3 of samples'
  filter cannot be meaningfully applied
- Questions about coding alternative polyadenylation, since aptardi does not analyze
  upstream exons
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: aptardi
  role: Produces the aptardi-modified transcriptome GTF whose transcripts are evaluated
    against the original transcriptome's DE results
  repo: github.com/luskry/aptardi
- name: RSEM
  role: Quantitates all samples against both the aptardi-modified and original transcriptomes,
    and re-quantitates the filtered high-quality transcriptomes before DE testing
- name: DESeq2
  role: Performs BNLx vs SHR differential expression testing on both filtered quantitation
    sets to extract p <= 0.001 transcripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:fd4a053fc3bc705c2ba3c712b70641642eef1003e90bc40b4af526f41f955258
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - RSEM
  - aptardi
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# aptardi-transcript-set-comparison

## Summary

Compares differential expression results (DESeq2 on RSEM quantitations) computed against an aptardi-modified transcriptome versus the original StringTie/Ensembl transcriptome, to quantify how aptardi-added transcripts change the set of significant transcripts (p <= 0.001).

## When to use

Use when you have RSEM quantitations for the same samples against two transcriptome GTFs (aptardi-modified and original) and need to identify which differentially expressed transcripts are attributable to aptardi-added 3'-end annotations.

## When NOT to use

- Working with non-mammalian taxa, where aptardi's manually engineered DNA sequence features may not apply
- Single-sample or very few-sample analyses where the '1 count in 2/3 of samples' filter cannot be meaningfully applied
- Questions about coding alternative polyadenylation, since aptardi does not analyze upstream exons

## Inputs

- Sorted BAM alignments of BNLx/SHR rat brain RNA-Seq samples
- aptardi-modified transcriptome GTF (original transcripts plus aptardi transcripts)
- Original StringTie/Ensembl transcriptome GTF
- RSEM quantitation tables for both transcriptomes (filtered and unfiltered)

## Outputs

- DESeq2 differential expression results (BNLx vs SHR) for aptardi and original transcriptomes
- List of significant transcripts (p <= 0.001) per transcriptome
- Set of aptardi-added differentially expressed transcripts (in aptardi DE set but absent from original DE set)
- differential_expression_results.tsv summarizing DE counts and validation comparison

## How to apply

Quantitate all samples with RSEM against both the aptardi-modified GTF and the original StringTie/Ensembl GTF. Apply the article's conservative quality filter — remove transcripts without at least 1 count in 2/3 of samples — then re-quantitate the high-quality transcriptomes with RSEM before differential expression testing. Run DESeq2 for the BNLx vs SHR contrast on both filtered quantitation sets and extract transcripts with p value <= 0.001. Identify aptardi-added DE transcripts as those present in the aptardi DE set but absent from the original transcriptome's DE set. Compare DE totals to the reported benchmark values (1166 of 32,348 for aptardi; 918 of 28,328 original: 918 of 28,329 as reported) and write results to differential_expression_results.tsv.

## Related tools

- **aptardi** (Produces the aptardi-modified transcriptome GTF whose transcripts are evaluated against the original transcriptome's DE results) — github.com/luskry/aptardi
- **RSEM** (Quantitates all samples against both the aptardi-modified and original transcriptomes, and re-quantitates the filtered high-quality transcriptomes before DE testing)
- **DESeq2** (Performs BNLx vs SHR differential expression testing on both filtered quantitation sets to extract p <= 0.001 transcripts)

## Examples

```
conda create -n aptardi_env --channel conda-forge -c bioconda aptardi
```

## Evaluation signals

- Number of significant transcripts (p <= 0.001) matches reported values: 1166 of 32,348 (aptardi transcriptome) and 918 of 28,329 (original transcriptome)
- The aptardi DE set is a superset-style comparison: aptardi-added DE transcripts must be absent from the original transcriptome's DE set by transcript ID
- Filtered transcriptomes must contain only transcripts with at least 1 count in 2/3 of samples before DESeq2 is run
- RSEM re-quantitation after filtering (not direct use of unfiltered counts) is the input to differential expression analysis
- All comparison results are written to differential_expression_results.tsv

## Limitations

- aptardi identifies genomic polyA regions and enumerates a transcript per isoform sharing a 3' terminal exon, creating redundant aptardi transcripts that make filtering essential
- aptardi performed modestly worse on the BNLx and SHR data sets; this may be due to true polyA sites derived from Sprague Dawley rat rather than the specific strain, unstranded RNA-Seq, or different species
- aptardi makes predictions on 100 base bins and only evaluates 3' terminal exons, so upstream coding APA events cannot be detected
- A stop site may be attributed to the wrong transcript when another isoform of the same gene already terminates at that location

## Evidence

- [readme] conservative filter for high-quality aptardi transcripts: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [readme] RSEM re-quantitation workflow: "1. Quantitate aptardi transcriptome with RSEM
	2. Generate high quality transcriptome by removing transcripts without at least 1 count in 2/3 of samples"
- [discussion] aptardi transcriptome benefit: "Adding aptardi transcripts increased the number of unique true polyA sites captured by the transcriptome and furthermore increased the ratio of true positive to false positive termini"
- [discussion] BNLx/SHR performance caveat: "However, we cannot rule out the possibility that this is owing to the unstranded RNA-Seq for these data sets and/or different species."
- [readme] 3'-terminal-exon-only scope: "Note that aptardi does not evaluate intron junctions but rather only evalutes 3' terminal exons of input transcripts and annotates 3' ends accordingly."
