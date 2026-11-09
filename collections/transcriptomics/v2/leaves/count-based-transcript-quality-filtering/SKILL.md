---
name: count-based-transcript-quality-filtering
description: Use when an aptardi or StringTie-derived transcriptome has been quantitated
  with RSEM across multiple samples and a high-quality subset of transcripts is needed
  before re-quantitation and DESeq2 differential expression testing.
when_to_use_negative:
- Working with a single sample or very few samples, where a 2/3-of-samples count rule
  is undefined or too strict — a simple expression threshold may be more appropriate
- The transcriptome has already been filtered and re-quantitated, and the task is
  downstream DESeq2 testing rather than quality filtering
- Evaluating raw aptardi polyA-site predictions before any RSEM quantitation, since
  the filter operates on per-sample counts, not on GTF coordinates
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: RSEM
  role: Quantitates the aptardi and original transcriptomes per sample so the 1-count-in-2/3-of-samples
    rule can be applied, and re-quantitates the filtered high-quality transcriptome
    before differential expression analysis
- name: aptardi
  role: Produces the modified GTF transcriptome whose added transcripts are the main
    target of this redundant-transcript-removing quality filter
  repo: github.com/luskry/aptardi
- name: DESeq2
  role: Consumes the filtered, re-quantitated count matrices for BNLx-vs-SHR differential
    expression testing (p <= 0.001)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:648ef93a74b4cf31aacd639555db6e979b7a5123eb15f9da820f7c9c9ea1fe95
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - RSEM
  - aptardi
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Count-based transcript quality filtering

## Summary

A conservative filtering step that removes transcripts lacking at least 1 count in 2/3 of samples from an RSEM-quantitated (e.g., aptardi-modified) transcriptome, yielding a high-quality transcriptome suitable for downstream differential expression analysis.

## When to use

Use when an aptardi or StringTie-derived transcriptome has been quantitated with RSEM across multiple samples and a high-quality subset of transcripts is needed before re-quantitation and DESeq2 differential expression testing.

## When NOT to use

- Working with a single sample or very few samples, where a 2/3-of-samples count rule is undefined or too strict — a simple expression threshold may be more appropriate
- The transcriptome has already been filtered and re-quantitated, and the task is downstream DESeq2 testing rather than quality filtering
- Evaluating raw aptardi polyA-site predictions before any RSEM quantitation, since the filter operates on per-sample counts, not on GTF coordinates

## Inputs

- RSEM per-sample transcript count matrix for the aptardi-modified transcriptome (GTF-derived quantitation)
- RSEM per-sample transcript count matrix for the original StringTie/Ensembl transcriptome
- Number of samples per condition (to compute the 2/3 threshold)

## Outputs

- Filtered high-quality transcript count table (transcripts with >= 1 count in >= 2/3 of samples)
- List of retained transcript IDs used to subset the GTF for RSEM re-quantitation

## How to apply

First quantitate the full (e.g., aptardi-modified) transcriptome with RSEM across all samples. Then apply the conservative filter by removing any transcript that does not have at least 1 count in at least 2/3 of the samples; this was found to do a good job of identifying high-quality aptardi transcripts because aptardi annotates polyA sites on genomic regions and can enumerate redundant transcripts across isoforms sharing 3' terminal exons. Retain only transcripts passing the filter to define a high-quality transcriptome, then re-quantitate that filtered transcriptome with RSEM so the filtered set feeds into DESeq2 differential expression testing. In the BNLx-vs-SHR rat brain application, this reduced the aptardi transcriptome from 32,348 to a filtered set and the original transcriptome from 28,329, after which transcripts with DESeq2 p <= 0.001 were extracted (1166 vs 918 significant transcripts).

## Related tools

- **RSEM** (Quantitates the aptardi and original transcriptomes per sample so the 1-count-in-2/3-of-samples rule can be applied, and re-quantitates the filtered high-quality transcriptome before differential expression analysis)
- **aptardi** (Produces the modified GTF transcriptome whose added transcripts are the main target of this redundant-transcript-removing quality filter) — github.com/luskry/aptardi
- **DESeq2** (Consumes the filtered, re-quantitated count matrices for BNLx-vs-SHR differential expression testing (p <= 0.001))

## Examples

```
Rscript filter_transcripts.R --counts rsem_aptardi_counts.tsv --min_count 1 --min_fraction 0.6667 --out high_quality_transcripts.tsv
```

## Evaluation signals

- Every retained transcript has at least 1 count in at least 2/3 of samples; spot-check the count matrix to confirm no transcript below threshold survives
- Filtered aptardi transcriptome size matches the reported scale (32,348 transcripts before DE testing in the BNLx/SHR analysis; original transcriptome 28,329)
- After DESeq2 on the filtered sets, transcripts with p <= 0.001 approximate the reported values: 1166 of 32,348 (aptardi) and 918 of 28,329 (original)
- Re-quantitation with RSEM is performed on the filtered transcriptome, not the unfiltered one, before DESeq2 is run

## Limitations

- aptardi enumerates a polyA-site transcript for every isoform sharing a 3' terminal exon, so redundant false positives can persist if isoform redundancy is high; curating a more selective input transcriptome (e.g., dropping Ensembl pseudo transcripts) mitigates this
- The 1-count-in-2/3-of-samples threshold is dataset-dependent; with fewer or single samples a fixed count threshold may be more appropriate
- The filter is conservative and may discard genuine low-abundance or sample-specific aptardi transcripts

## Evidence

- [readme] we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [readme] 1. Quantitate aptardi transcriptome with RSEM 2. Generate high quality transcriptome by removing transcripts without at least 1 count in 2/3 of samples: "1. Quantitate aptardi transcriptome with RSEM
	2. Generate high quality transcriptome by removing transcripts without at least 1 count in 2/3 of samples"
- [readme] we re-quantitated the high quality transcriptome with RSEM to perform differential expression analysis: "(we re-quantitated the high quality transcriptome with RSEM to perform differential expression analysis)"
- [readme] if aptardi probes the same region for two transcript isoforms and identifies a polyA site, it will enumerate an aptardi transcript for both isoforms: "if aptardi probes the same region for two transcript isoforms and identifies a polyA site, it will enumerate an aptardi transcript for both isoforms"
- [other] Differential expression analysis of the BNLx and SHR rat brain data sets yielded more significant transcripts (p value ≤ 0.001) with the aptardi modified transcriptome (1166 of 32,348): [evidence span withheld — anchor status: not_found]
