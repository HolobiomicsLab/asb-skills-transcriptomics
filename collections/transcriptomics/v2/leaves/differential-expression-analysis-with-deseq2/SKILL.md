---
name: differential-expression-analysis-with-deseq2
description: Use when comparing two conditions (e.g., BNLx vs SHR strains) at transcript
  level after RSEM quantitation, especially to test whether a modified transcriptome
  changes the DE call set.
when_to_use_negative:
- Quantitation has not been performed with RSEM against a defined transcriptome GTF
  (DESeq2 needs a count matrix from the filtered high-quality transcriptome).
- You have fewer than the recommended replicates and no way to apply the 1-count-in-2/3-of-samples
  filter, since unfiltered aptardi transcriptomes contain redundant false-positive
  transcripts.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DESeq2
  role: Performs differential expression testing (BNLx vs SHR) on the filtered RSEM
    quantitation sets
- name: RSEM
  role: Quantitates all samples against the aptardi-modified and original transcriptomes;
    re-quantitates the high-quality filtered transcriptome before DE testing
- name: aptardi
  role: Produces the modified transcriptome GTF whose DE call set is compared against
    the original transcriptome's DE set
  repo: https://github.com/luskry/aptardi
- name: StringTie
  role: Reconstructs the original transcriptome GTF used as the comparison baseline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:9f0546e71f4a95646f5183dca54516ec9aba277fa8c952aa1f3a3bcd3dd2007c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - DESeq2
  - RSEM
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- tncrna-differential-expression-analysis
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# differential-expression-analysis-with-deseq2

## Summary

Run DESeq2 differential expression testing (e.g., BNLx vs SHR rat brain) on RSEM-quantitated, quality-filtered transcriptomes to identify transcripts with p <= 0.001, and compare DE sets between an aptardi-modified and an original StringTie/Ensembl transcriptome.

## When to use

Use when comparing two conditions (e.g., BNLx vs SHR strains) at transcript level after RSEM quantitation, especially to test whether a modified transcriptome changes the DE call set.

## When NOT to use

- Quantitation has not been performed with RSEM against a defined transcriptome GTF (DESeq2 needs a count matrix from the filtered high-quality transcriptome).
- You have fewer than the recommended replicates and no way to apply the 1-count-in-2/3-of-samples filter, since unfiltered aptardi transcriptomes contain redundant false-positive transcripts.

## Inputs

- Sorted BAM alignments of BNLx/SHR rat brain RNA-Seq samples
- Aptardi-modified transcriptome GTF (original transcripts plus aptardi transcripts)
- Original StringTie/Ensembl transcriptome GTF
- RSEM quantitation count matrices per transcriptome (pre- and post-filter)

## Outputs

- DESeq2 DE results per transcriptome with transcripts passing p <= 0.001
- Set of aptardi-added differentially expressed transcripts (in aptardi DE set, absent from original DE set)
- differential_expression_results.tsv summarizing DE transcript totals

## How to apply

Quantitate all samples against both transcriptomes with RSEM, then apply the conservative quality filter — remove transcripts without at least 1 count in 2/3 of samples — and re-quantitate the high-quality transcriptomes with RSEM before testing. Run DESeq2 differential expression testing (BNLx vs SHR) on both filtered quantitation sets and extract transcripts with p <= 0.001. Identify aptardi-added DE transcripts as those present in the aptardi DE set but absent from the original transcriptome's DE set, then compare totals against expected counts (1166 of 32,348 aptardi vs 918 of 28,329 original). Filtering is recommended because aptardi annotates genomic polyA regions and can enumerate redundant transcripts for isoforms sharing 3' terminal exons.

## Related tools

- **DESeq2** (Performs differential expression testing (BNLx vs SHR) on the filtered RSEM quantitation sets)
- **RSEM** (Quantitates all samples against the aptardi-modified and original transcriptomes; re-quantitates the high-quality filtered transcriptome before DE testing)
- **aptardi** (Produces the modified transcriptome GTF whose DE call set is compared against the original transcriptome's DE set) — https://github.com/luskry/aptardi
- **StringTie** (Reconstructs the original transcriptome GTF used as the comparison baseline)

## Examples

```
Rscript deseq2_bnlx_vs_shr.R --counts aptardi_filtered.rsem.counts --counts original_filtered.rsem.counts --pval 0.001 --out differential_expression_results.tsv
```

## Evaluation signals

- Number of transcripts with p <= 0.001 matches 1166 of 32,348 for the aptardi transcriptome.
- Number of transcripts with p <= 0.001 matches 918 of 28,329 for the original transcriptome.
- The aptardi DE set is larger than the original DE set, and aptardi-added DE transcripts are exactly those absent from the original transcriptome's DE set.
- Filtered transcriptomes contain only transcripts with at least 1 count in 2/3 of samples before DE testing.
- Results are written to differential_expression_results.tsv with per-transcriptome totals.

## Limitations

- aptardi incorporates a single 3'-end into multiple transcripts per gene because it does not distinguish transcripts overlapping the same genomic region, so a polyA site may be attributed to the wrong transcript without filtering.
- The 1-count-in-2/3-of-samples filter is a conservative heuristic; with fewer or single samples a simple threshold may work but the exact number depends on the dataset.
- aptardi predicted modestly worse on the BNLx and SHR data sets, possibly owing to unstranded RNA-Seq and/or a different species from the model training data.

## Evidence

- [readme] Conservative count filter rationale: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [methods] Re-quantitation before DE analysis: "we re-quantitated the high quality transcriptome with RSEM to perform differential expression analysis"
- [readme] Filtering recommendation due to redundancy: "we HIGHLY RECOMMEND filtering to remove redundant, i.e. false positive, aptardi transcripts, especially if your input transcriptome has many transcript isoforms that share 3' terminal exons"
- [discussion] aptardi limitation for multi-isoform genes: "aptardi incorporates a single 3′-end into multiple transcripts for each gene listed because it does not distinguish transcripts overlapping the same genomic region"
