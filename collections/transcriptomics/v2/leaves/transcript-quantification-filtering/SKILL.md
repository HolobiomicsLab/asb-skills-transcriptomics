---
name: transcript-quantification-filtering
description: Use when an aptardi (or assembler-generated) GTF has been quantitated
  and redundant/false-positive 3'-terminal transcripts must be filtered by an expression
  count threshold across replicate samples before differential analysis.
when_to_use_negative:
- Working with a single sample or very few replicates, where the 2/3-of-samples rule
  is undefined and a dataset-specific count threshold must be chosen instead
- The input transcriptome is already a curated, non-redundant annotation (e.g., Ensembl)
  with no aptardi-enumerated duplicate 3'-terminal transcripts to remove
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: RSEM
  role: Quantitates the aptardi transcriptome per sample, and re-quantitates the high-quality
    filtered transcriptome for differential expression analysis
- name: aptardi
  role: Produces the modified transcriptome GTF whose added transcripts the quantitation-and-filter
    step must prune
  repo: https://github.com/luskry/aptardi
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:b075dbb69bc4d75bf365f9931800036c7a39e2a8070f5d01e35f3c27b947b1ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - BEDtools
  - HISAT2
  - RSEM
  - SAMtools
  - aptardi
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# transcript-quantification-filtering

## Summary

Quantify an aptardi-modified transcriptome with RSEM, then remove transcripts lacking at least one count in two-thirds of samples to build a high-quality transcriptome that eliminates redundant false-positive aptardi transcripts before downstream differential analyses.

## When to use

Use when an aptardi (or assembler-generated) GTF has been quantitated and redundant/false-positive 3'-terminal transcripts must be filtered by an expression count threshold across replicate samples before differential analysis.

## When NOT to use

- Working with a single sample or very few replicates, where the 2/3-of-samples rule is undefined and a dataset-specific count threshold must be chosen instead
- The input transcriptome is already a curated, non-redundant annotation (e.g., Ensembl) with no aptardi-enumerated duplicate 3'-terminal transcripts to remove

## Inputs

- aptardi-modified transcriptome GTF/GFF (original plus aptardi-added transcripts)
- Sorted, genome-aligned RNA-Seq BAM files for all samples (e.g., HISAT2-aligned brain/liver samples)
- Sample count table per replicate group (to determine the 2/3-of-samples rule)

## Outputs

- RSEM quantitation of the full aptardi transcriptome
- High-quality filtered transcriptome (transcripts with ≥1 count in 2/3 of samples)
- Re-quantitated RSEM expression matrix suitable for differential expression analysis

## How to apply

First quantitate the aptardi transcriptome with RSEM using the sample alignments. Apply the article's conservative filter: remove any transcript without at least 1 count in 2/3 of the samples, since aptardi can enumerate multiple aptardi transcripts for isoforms sharing a 3' terminal exon and identifies genomic regions rather than sample-specific sites. Re-quantitate the resulting high-quality transcriptome with RSEM to perform downstream differential expression analysis (e.g., comparing brain- vs liver-specific polyA site counts against PolyA-Seq ground truth). For few-sample or single-sample datasets, a simple count threshold may substitute, but the cutoff should be tuned to the dataset.

## Related tools

- **RSEM** (Quantitates the aptardi transcriptome per sample, and re-quantitates the high-quality filtered transcriptome for differential expression analysis)
- **aptardi** (Produces the modified transcriptome GTF whose added transcripts the quantitation-and-filter step must prune) — https://github.com/luskry/aptardi

## Examples

```
rsem-calculate-expression --paired-end aligned_sample aptardi_transcriptome aptardi_quant; rsem-calculate-expression --paired-end aligned_sample high_quality_transcriptome hq_quant
```

## Evaluation signals

- Filtered transcriptome contains only transcripts with ≥1 count in 2/3 of samples; verify counts against RSEM quantitation output
- Re-quantitation with RSEM on the filtered transcriptome runs without errors and yields an expression matrix with the expected number of transcripts
- In the mouse brain/liver validation, the filtered aptardi transcriptome yields 26 unannotated brain-specific sites (fold increase 9 vs StringTie's 3) and 69 unannotated liver-specific sites (fold increase 17 vs StringTie's 4) versus Derti et al. PolyA-Seq ground truth
- False-positive aptardi transcripts sharing duplicated 3' terminal exons are reduced relative to the unfiltered transcriptome

## Limitations

- aptardi annotates genomic regions, so isoforms sharing a 3' terminal exon get duplicate aptardi transcripts; filtering mitigates but does not fully resolve redundant enumeration, and a stop site may be attributed to the wrong transcript
- With low sequencing depth (as in the mouse tissue data), the overall number of polyA sites detected is modest, which can also depress counts passing the filter
- The exact count threshold may depend on the dataset, particularly with few or single samples

## Evidence

- [readme] we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [readme] Filtering pipeline: quantitate with RSEM, remove transcripts without at least 1 count in 2/3 of samples, re-quantitate for differential expression: "1. Quantitate aptardi transcriptome with RSEM
2. Generate high quality transcriptome by removing transcripts without at least 1 count in 2/3 of samples
3. Proceed with downstream analyses (we"
- [readme] aptardi identifies genomic regions, so redundant aptardi transcripts must be filtered when isoforms share 3' terminal exons: "Since aptardi identifies **genomic** regions to annotate polyA sites, we HIGHLY RECOMMEND filtering to remove redundant, i.e. false positive, aptardi transcripts"
- [discussion] Adding aptardi transcripts increased the number of unique true polyA sites captured by the transcriptome and furthermore increased the ratio of true positive to false positive termini: "Adding aptardi transcripts increased the number of unique true polyA sites captured by the transcriptome and furthermore increased the ratio of true positive to false positive termini"
