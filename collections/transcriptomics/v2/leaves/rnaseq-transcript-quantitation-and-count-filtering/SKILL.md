---
name: rnaseq-transcript-quantitation-and-count-filtering
description: Use when a GTF transcriptome (e.g., from StringTie plus aptardi) contains
  many candidate transcript isoforms sharing 3' terminal exons and needs expression-based
  filtering before quantitation and differential analysis.
when_to_use_negative:
- When working with a single sample or few samples where the 2/3-samples rule is undefined
  — a simple expression threshold may be more appropriate, and the cutoff depends
  on the dataset.
- When the input transcriptome is already curated and lacks many isoforms sharing
  3' terminal exons — the README's redundancy rationale for filtering does not apply.
- When the goal is differential 3' UTR usage analysis requiring all aptardi-added
  transcripts retained without count-based removal.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: RSEM
  role: Quantitates the aptardi transcriptome to obtain per-transcript counts used
    for the 2/3-samples filter, then re-quantitates the high-quality transcriptome
    for differential expression
- name: aptardi
  role: Produces the modified transcriptome whose added transcripts are quantitated
    and filtered
  repo: github.com/luskry/aptardi
- name: StringTie
  role: Reconstructs the initial transcriptome from the sorted BAM using the Ensembl
    guide annotation prior to aptardi refinement
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:d820b8bbfef0c13216f1a8e4bb94a0bd7a63e4ae6c4ae9f2c98ed98ede1f9059
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - APARENT
  - RSEM
  - StringTie
  - TAPAS
  - aptardi
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rnaseq-transcript-quantitation-and-count-filtering

## Summary

Quantitate a transcriptome (e.g., an aptardi-modified GTF) with RSEM and then generate a high-quality transcriptome by removing transcripts without at least 1 count in 2/3 of samples. This filtering step removes redundant/false-positive aptardi transcripts before downstream differential expression analyses.

## When to use

Use when a GTF transcriptome (e.g., from StringTie plus aptardi) contains many candidate transcript isoforms sharing 3' terminal exons and needs expression-based filtering before quantitation and differential analysis.

## When NOT to use

- When working with a single sample or few samples where the 2/3-samples rule is undefined — a simple expression threshold may be more appropriate, and the cutoff depends on the dataset.
- When the input transcriptome is already curated and lacks many isoforms sharing 3' terminal exons — the README's redundancy rationale for filtering does not apply.
- When the goal is differential 3' UTR usage analysis requiring all aptardi-added transcripts retained without count-based removal.

## Inputs

- aptardi-modified transcriptome in GTF/GFF format (hbr_aptardi.gtf)
- Sorted, genome-aligned RNA-Seq BAM file (e.g., HISAT2-aligned HBR reads)
- Reference genome FASTA (hg38.fa)
- RSEM quantitation outputs (per-transcript counts) across samples

## Outputs

- High-quality filtered transcriptome GTF (transcripts with at least 1 count in 2/3 of samples)
- RSEM expression estimates for the filtered transcriptome for downstream differential expression
- Comparison metrics table (e.g., aptardi_tapas_aparent_comparison.csv) of correct 3' termini, PPV, and unique true PolyA-Seq sites

## How to apply

After constructing the transcriptome (StringTie reconstruction with an Ensembl guide, then aptardi Mode 1 refinement of 3' ends), quantitate all transcripts with RSEM. Because aptardi annotates polyA sites in genomic regions and can add a transcript per isoform even when isoforms share the same 3' terminal exon, apply the conservative filter of removing transcripts without at least 1 count in 2/3 of samples to yield the high-quality transcriptome. Re-quantitate the filtered high-quality transcriptome with RSEM before performing differential expression analysis. Correct application is judged by the reduction in redundant aptardi transcripts and by consistent counts in the downstream comparison metrics (e.g., aptardi 62,688 correct 3' termini vs TAPAS 22,804 on HBR, PPV 0.44 vs 0.31).

## Related tools

- **RSEM** (Quantitates the aptardi transcriptome to obtain per-transcript counts used for the 2/3-samples filter, then re-quantitates the high-quality transcriptome for differential expression)
- **aptardi** (Produces the modified transcriptome whose added transcripts are quantitated and filtered) — github.com/luskry/aptardi
- **StringTie** (Reconstructs the initial transcriptome from the sorted BAM using the Ensembl guide annotation prior to aptardi refinement)

## Examples

```
stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file> && aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --g hbr_aptardi.gtf --n model.hdf5 --t scale.pk --o output_dir && rsem-prepare-reference --gtf hbr_aptardi.gtf hg38.fa ref && rsem-calculate-expression --paired-end sample_1.fq sample_2.fq ref sample_rsem
```

## Evaluation signals

- All transcripts in the filtered GTF have at least 1 count in at least 2/3 of samples in the RSEM count matrix.
- Number of transcripts in the high-quality transcriptome is substantially reduced relative to the raw aptardi output, reflecting removal of redundant false-positive aptardi transcripts.
- Re-running RSEM on the filtered transcriptome succeeds and yields expression estimates for downstream differential expression analysis.
- Downstream benchmark metrics fall in the reported ranges: aptardi correct 3' termini ~62,688 vs TAPAS 22,804 on HBR; PPV 0.44 vs 0.31; fraction of added aptardi transcripts matching a true PolyA-Seq site ~82% (22,846/27,853) vs 35% (39,842/113,923) for the original transcriptome.
- Filtered metrics are written to the comparison CSV (aptardi_tapas_aparent_comparison.csv) with all four predictors compared against true PolyA-Seq sites.

## Limitations

- aptardi annotates polyA sites in genomic regions and may enumerate an aptardi transcript for multiple isoforms sharing the same 3' terminal exon, inflating false positives if filtering is skipped.
- aptardi may attribute a transcript stop site to the wrong isoform, since it does not distinguish transcripts overlapping the same genomic region; the README suggests mitigating this by curating a more-selective input transcriptome (Ensembl includes many pseudo transcripts).
- Predictions are made on 100 base bins, a resolution limitation, though polyA sites wiggle up to 30 nucleotides and few bins contained multiple sites.
- The 2/3-of-samples count threshold is dataset-dependent; with fewer or a single sample a different threshold is likely needed.

## Evidence

- [methods] we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [readme] 1. Quantitate aptardi transcriptome with RSEM 2. Generate high quality transcriptome by removing transcripts without at least 1 count in 2/3 of samples 3. Proceed with downstream analyses (we re-quantitated the high quality transcriptome with RSEM to perform differential expression analysis): "1. Quantitate aptardi transcriptome with RSEM
	2. Generate high quality transcriptome by removing transcripts without at least 1 count in 2/3 of samples"
- [readme] if aptardi probes the same region for two transcript isoforms and identifies a polyA site, it will enumerate an aptardi transcript for both isoforms: "if aptardi probes the same region for two transcript isoforms and identifies a polyA site, it will enumerate an aptardi transcript for both isoforms"
- [discussion] Adding aptardi transcripts increased the number of unique true polyA sites captured by the transcriptome and furthermore increased the ratio of true positive to false positive termini: "Adding aptardi transcripts increased the number of unique true polyA sites captured by the transcriptome and furthermore increased the ratio of true positive to false positive termini"
