---
name: transcriptome-quantitation-with-rsem
description: Use when sample-specific transcriptomes (e.g. aptardi or StringTie/Ensembl
  GTFs) need per-transcript expression counts for filtering or differential expression
  analysis.
when_to_use_negative:
- When the input transcriptome has not yet been constructed or refined — RSEM requires
  an existing GTF (e.g. StringTie output or aptardi-modified GTF) as reference
- When the goal is transcriptome assembly or polyA site prediction itself — use StringTie
  or aptardi first; RSEM only quantitates against a fixed annotation
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: RSEM
  role: Quantitates the aptardi-modified and original transcriptomes to generate counts
    used for the 2/3-samples filter and DESeq2 differential expression
- name: DESeq2
  role: Consumes the re-quantitated RSEM counts to call differentially expressed transcripts
    (BNLx vs SHR, p <= 0.001)
- name: aptardi
  role: Produces the aptardi-modified transcriptome GTF that RSEM quantitates
  repo: https://github.com/luskry/aptardi
- name: StringTie
  role: Produces the sample-specific input transcriptome (stringtie.gtf) and the original
    comparison transcriptome quantitated with RSEM
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:20a107d797d2de94f48ca8adbb2e6bd4c1885be7c4e80bf42d9b77fe68ca55b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - Cutadapt
  - DESeq2
  - FastQC
  - HISAT2
  - RSEM
  - SAMtools
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# transcriptome-quantitation-with-rsem

## Summary

Quantitate a transcriptome (GTF) with RSEM to obtain per-transcript counts, enabling both quality filtering of aptardi-modified transcriptomes and downstream DESeq2 differential expression analysis. In the aptardi workflow, RSEM quantitation is run twice: once to count transcripts for filtering, and again on the filtered high-quality transcriptome for DE testing.

## When to use

Use when sample-specific transcriptomes (e.g. aptardi or StringTie/Ensembl GTFs) need per-transcript expression counts for filtering or differential expression analysis.

## When NOT to use

- When the input transcriptome has not yet been constructed or refined — RSEM requires an existing GTF (e.g. StringTie output or aptardi-modified GTF) as reference
- When the goal is transcriptome assembly or polyA site prediction itself — use StringTie or aptardi first; RSEM only quantitates against a fixed annotation

## Inputs

- Sorted BAM file of genome-aligned RNA-Seq reads (e.g. HISAT2-aligned BNLx/SHR rat brain samples)
- Transcriptome GTF file (aptardi-modified name_aptardi.gtf or original StringTie/Ensembl GTF)
- Reference genome FASTA (e.g. hg38.fa)

## Outputs

- RSEM per-transcript expression/counts table for each sample and each transcriptome
- Filtered high-quality transcriptome (transcripts with >= 1 count in 2/3 of samples)
- RSEM re-quantitation of the high-quality transcriptome for DESeq2 differential expression analysis

## How to apply

Quantitate each sample's BAM alignments against the transcriptome of interest (the aptardi-modified GTF or the original StringTie/Ensembl GTF) using RSEM. Apply the conservative quality filter of removing transcripts without at least 1 count in 2/3 of samples to identify high-quality aptardi transcripts. Re-quantitate the filtered high-quality transcriptome with RSEM, then feed the counts into DESeq2 for differential expression testing (e.g. BNLx vs SHR, transcripts with p <= 0.001). RSEM is run separately for each annotation so DE results from aptardi and original transcriptomes can be compared.

## Related tools

- **RSEM** (Quantitates the aptardi-modified and original transcriptomes to generate counts used for the 2/3-samples filter and DESeq2 differential expression)
- **DESeq2** (Consumes the re-quantitated RSEM counts to call differentially expressed transcripts (BNLx vs SHR, p <= 0.001))
- **aptardi** (Produces the aptardi-modified transcriptome GTF that RSEM quantitates) — https://github.com/luskry/aptardi
- **StringTie** (Produces the sample-specific input transcriptome (stringtie.gtf) and the original comparison transcriptome quantitated with RSEM)

## Examples

```
rsem-calculate-expression --paired-end sorted.bam aptardi_transcriptome rsem_out
```

## Evaluation signals

- Filter invariant: after filtering, every retained transcript has at least 1 count in at least 2/3 of samples
- Differential expression benchmark: aptardi transcriptome yields 1166 of 32,348 transcripts with p <= 0.001; original transcriptome yields 918 of 28,329
- Aptardi-added DE transcripts are those present in the aptardi DE set but absent from the original transcriptome's DE set
- Re-quantitation is performed on the filtered transcriptome, not the unfiltered one, before running DESeq2

## Limitations

- The 1-count-in-2/3-of-samples filter is conservative; with fewer or a single sample, a simple threshold may work but the exact cutoff is dataset-dependent
- aptardi enumerates an aptardi transcript per overlapping isoform sharing the same 3' region, so unfiltered transcriptomes contain redundant false-positive transcripts that RSEM counts help remove
- aptardi performed modestly worse on BNLx and SHR data sets, possibly owing to unstranded RNA-Seq and/or different species

## Evidence

- [readme] RSEM quantitation for filtering and DE: "1. Quantitate aptardi transcriptome with RSEM"
- [readme] 2/3-samples filter: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [methods] Re-quantitation for downstream DE: "(we re-quantitated the high quality transcriptome with RSEM to perform differential expression analysis)"
- [results] DE result comparison: [evidence span withheld — anchor status: not_found]
