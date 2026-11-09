---
name: transcript-abundance-quantification
description: Use when RNA-seq reads must be quantified against an augmented transcriptome
  that includes InPACT-reconstructed novel IPA isoforms, and per-site IPA usage (PAU)
  values are needed downstream.
when_to_use_negative:
- Only annotated 3'UTR APA quantification is needed and no novel IPA isoform discovery
  is required — tools like QAPA or LABRAT suffice and InPACT transcript assembly/quantification
  is unnecessary.
- Input is single-cell UMI count data (e.g., SMART-seq2 or 10x scRNA-seq matrices)
  — Salmon quant of full transcriptomes from bulk FASTQ does not apply directly to
  scRNA-seq pipelines.
- Only 3'-end sequencing (3P-seq, polyA-seq, A-seq) data are available — this skill
  targets conventional RNA-seq alignments/reads, not dedicated 3'-end assays.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: InPACT_quantify
  role: Computes per-site IPA usage (PAU) from Salmon quant.sf output, merged.gtf
    annotation, and predict.result.txt IPA info
  repo: https://github.com/YY-TMU/InPACT
- name: Salmon
  role: Indexes the gffread-extracted transcript sequences (k-mer size 31) and quantifies
    transcript abundances with selective alignment (--validateMappings)
  repo: https://github.com/COMBINE-lab/salmon
- name: gffread
  role: Extracts transcript sequences from merged.gtf against genome.fa to build the
    quantification reference
- name: InPACT_transcript
  role: Upstream step that assembles novel IPA isoforms into merged.gtf, which augments
    the reference transcriptome for quantification
  repo: https://github.com/YY-TMU/InPACT
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:b975aefcce27f8b8b927199fa67122999a154f13fc033ea1483402d2c18a1cbb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - HISAT2
  - InPACT
  - InPACT_quantify
  - InPACT_transcript
  - SAMtools
  - Salmon
  - gffread
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# transcript-abundance-quantification

## Summary

Quantify transcript-level abundances of annotated and InPACT-assembled novel IPA isoforms by extracting transcript sequences with gffread, building a Salmon index (k-mer size 31), running salmon quant with --validateMappings, and computing per-site IPA usage (PAU) with InPACT_quantify.

## When to use

Use when RNA-seq reads must be quantified against an augmented transcriptome that includes InPACT-reconstructed novel IPA isoforms, and per-site IPA usage (PAU) values are needed downstream.

## When NOT to use

- Only annotated 3'UTR APA quantification is needed and no novel IPA isoform discovery is required — tools like QAPA or LABRAT suffice and InPACT transcript assembly/quantification is unnecessary.
- Input is single-cell UMI count data (e.g., SMART-seq2 or 10x scRNA-seq matrices) — Salmon quant of full transcriptomes from bulk FASTQ does not apply directly to scRNA-seq pipelines.
- Only 3'-end sequencing (3P-seq, polyA-seq, A-seq) data are available — this skill targets conventional RNA-seq alignments/reads, not dedicated 3'-end assays.

## Inputs

- merged.gtf — augmented annotation from InPACT_transcript (RefSeq GTF + novel IPA isoforms)
- Reference genome FASTA (genome.fa, GRCh38/hg38)
- RNA-seq FASTQ reads (paired-end reads1/reads2.fq or single-end reads.fq)
- predict.result.txt — InPACT IPA-site prediction table

## Outputs

- transcripts.fa — transcript sequences extracted by gffread
- transcripts_index — Salmon k-31 index
- quant.sf — transcript-level TPM/NumReads per sample
- ipa_usage.txt — per-site IPA usage (PAU) estimates with terminal exon, IPA type (skipped/composite), gene, upstream coordinate, and polyA site columns

## How to apply

After InPACT_transcript assembles novel IPA isoforms into merged.gtf, extract transcript sequences with gffread (gffread merged.gtf -g genome.fa -w transcripts.fa). Build a Salmon index on those transcripts with k-mer size 31 (salmon index -t transcripts.fa -i transcripts_index -k 31), then quantify each replicate/condition with salmon quant using --validateMappings and the correct library type (-l for paired-end -1/-2 or single-end -r), producing quant.sf. Compute per-isoform IPA usage with InPACT_quantify --transcript_tpm quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt. The resulting ipa_usage.txt provides PAU estimates keyed to each predicted intronic terminal exon, which can be compared across replicates (e.g., Pearson correlation r = 0.94 for HEK293 novel IPA isoform expression) or across conditions (e.g., Wilcoxon rank-sum test of IPA usage upon U1 AMO treatment).

## Related tools

- **InPACT_quantify** (Computes per-site IPA usage (PAU) from Salmon quant.sf output, merged.gtf annotation, and predict.result.txt IPA info) — https://github.com/YY-TMU/InPACT
- **Salmon** (Indexes the gffread-extracted transcript sequences (k-mer size 31) and quantifies transcript abundances with selective alignment (--validateMappings)) — https://github.com/COMBINE-lab/salmon
- **gffread** (Extracts transcript sequences from merged.gtf against genome.fa to build the quantification reference)
- **InPACT_transcript** (Upstream step that assembles novel IPA isoforms into merged.gtf, which augments the reference transcriptome for quantification) — https://github.com/YY-TMU/InPACT

## Examples

```
gffread merged.gtf -g genome.fa -w transcripts.fa; salmon index -t transcripts.fa -i transcripts_index -k 31; salmon quant -i transcripts_index -l A -1 reads1.fq -2 reads2.fq --validateMappings -o transcripts_quant; InPACT_quantify --transcript_tpm transcripts_quant/quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt
```

## Evaluation signals

- quant.sf exists per replicate with TPM columns for both annotated and novel InPACT IPA isoform transcripts from merged.gtf.
- ipa_usage.txt contains the expected schema: terminal exon, IPA type (Skipped/Composite), gene symbol, upstream coordinate, PolyAsite, and IPA usage (PAU) columns.
- Per-replicate PAU values for novel IPA isoforms should be reproducible across biological replicates — the HEK293 test case reports Pearson r = 0.94, P < 0.05.
- Condition-level comparison of PAU values should detect expected shifts, e.g., significantly increased IPA usage upon U1 AMO treatment (Wilcoxon rank sum test, P < 2.2e-16, 767 vs. 151 novel IPA events).
- Salmon index must be rebuilt whenever merged.gtf changes (novel isoform set), and the same index build (k = 31) must be used for all compared samples.

## Limitations

- Salmon 2.0 changed the index format, so indices must be rebuilt; the final C++ release (1.12.0) remains available as salmon-cpp.
- IPA usage (PAU) estimates depend on upstream InPACT prediction quality — read thresholds (>=5 spliced reads for skipped, >=10 boundary-crossing reads for composite terminal exons) and sequencing depth affect the novel isoform set being quantified.
- For single-end reads the correct library type must be passed to salmon quant (-l <LIBTYPE>); an incorrect LIBTYPE will distort abundance estimates.

## Evidence

- [readme] gffread and salmon is used to index and quantify the transcriptome, and then the usage is calculated.: "gffread and salmon is used to index and quantify the transcriptome, and then the usage is calculated."
- [methods] salmon index -t transcripts.fa -i transcripts_index -k 31: "salmon index -t transcripts.fa -i transcripts_index -k 31"
- [methods] salmon quant -i transcripts_index -l <LIBTYPE> -r reads.fq --validateMappings -o transcripts_quant: "salmon quant -i transcripts_index -l <LIBTYPE> -r reads.fq --validateMappings -o transcripts_quant"
- [readme] InPACT_quantify --transcript_tpm quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt: "InPACT_quantify --transcript_tpm quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt"
- [other] the expression levels of novel assembled IPA isoforms are robustly correlated between the two HEK293 RNA-seq replicates (Pearson's correlation r = 0.94, P < 0.05): [evidence span withheld — anchor status: not_found]
