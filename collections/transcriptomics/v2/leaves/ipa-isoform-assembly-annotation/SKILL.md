---
name: ipa-isoform-assembly-annotation
description: Use when sorted RNA-seq or scRNA-seq BAM alignments (BAM against GRCh38
  with RefSeq GTF) have yielded InPACT-predicted novel IPA terminal exons and you
  need assembled isoform GTFs plus per-site IPA usage values.
when_to_use_negative:
- The goal is only 3'UTR alternative polyadenylation quantification without novel
  intronic IPA isoform discovery — tools like DaPars, Aptardi, APAtrap, or TAPAS address
  that narrower scope
- Input alignments are unsorted or contain secondary alignments — InPACT requires
  coordinate-sorted BAMs and the workflow removes secondary alignments with SAMtools
  first
- The genome/annotation is not human GRCh38 with a precomputed InPACT_polyAsites SAF,
  since the provided candidate-site file is hg38-specific
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: InPACT_transcript
  role: Assembles novel IPA isoforms from predict.result.txt into an augmented merged
    GTF with the annotated RefSeq GTF
  repo: https://github.com/YY-TMU/InPACT
- name: gffread
  role: Extracts transcript sequences from merged.gtf for Salmon index building
- name: Salmon
  role: Indexes transcript sequences (k=31) and quantifies transcript TPM for annotated
    and novel IPA isoforms
  repo: https://github.com/COMBINE-lab/salmon
- name: InPACT_quantify
  role: Computes per-site IPA usage (PAU) from Salmon quant.sf, merged.gtf, and predict.result.txt
  repo: https://github.com/YY-TMU/InPACT
- name: HISAT2
  role: Aligns raw reads to GRCh38 to produce the coordinate-sorted BAM input required
    by InPACT
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:170441fdc4205c37643e7bfbd01af3415e23ee06b27aad891d9280c22c4892c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - ClusterProfiler
  - DRIMSeq
  - HISAT2
  - InPACT
  - InPACT_quantify
  - InPACT_transcript
  - SAMtools
  - Salmon
  - gffread
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# ipa-isoform-assembly-annotation

## Summary

Assemble novel intronic polyadenylation (IPA) isoforms identified by InPACT into an augmented transcript annotation, then quantify transcript-level abundance with Salmon/gffread to compute IPA usage (PAU) per site. This extends IPA site detection from a coordinate list to full isoforms with measurable relative usage.

## When to use

Use when sorted RNA-seq or scRNA-seq BAM alignments (BAM against GRCh38 with RefSeq GTF) have yielded InPACT-predicted novel IPA terminal exons and you need assembled isoform GTFs plus per-site IPA usage values.

## When NOT to use

- The goal is only 3'UTR alternative polyadenylation quantification without novel intronic IPA isoform discovery — tools like DaPars, Aptardi, APAtrap, or TAPAS address that narrower scope
- Input alignments are unsorted or contain secondary alignments — InPACT requires coordinate-sorted BAMs and the workflow removes secondary alignments with SAMtools first
- The genome/annotation is not human GRCh38 with a precomputed InPACT_polyAsites SAF, since the provided candidate-site file is hg38-specific

## Inputs

- predict.result.txt from InPACT site prediction (terminal exon, IPA type, gene, coordinates, PolyAsite)
- Sorted coordinate BAM file of RNA-seq or scRNA-seq reads aligned to GRCh38 (HISAT2, secondary alignments filtered)
- RefSeq GTF annotation (e.g., hg38.109.20211119.ncbiRefSeq.gtf)
- Reference genome FASTA (GRCh38/hg38)
- InPACT_polyAsites.hg38.saf candidate polyA site file
- Raw FASTQ reads (paired-end or single-end) for Salmon quantification

## Outputs

- merged.gtf containing annotated plus novel IPA isoforms
- transcripts.fa extracted by gffread
- quant.sf transcript abundance file from salmon quant
- ipa_usage.txt table with columns: Terminal exon, IPA type (Skipped/Composite), Gene, Upstream coordinate, PolyAsite, IPA usage (PAU)

## How to apply

Run InPACT_transcript with --predict_terminal predict.result.txt, --annotated_gtf RefSeq.gtf (hg38.109.20211119.ncbiRefSeq.gtf), --fa_path genome.fa, and --save_gtf merged.gtf to merge novel IPA isoforms with the RefSeq annotation. Extract transcript sequences from merged.gtf with gffread, build a Salmon index with k-mer size 31, and quantify with salmon quant --validateMappings (use -r for single-end SMART-seq2 reads, -1/-2 for paired-end). Then run InPACT_quantify with --transcript_tpm quant.sf, --annotation_file merged.gtf, and --ipa_info predict.result.txt to compute IPA usage (PAU) per site. In scRNA-seq settings, filter low-abundance genes with TPM < 1 before comparing IPA usage across cell types. The rationale is that transcript-level quantification over both annotated and novel IPA isoforms gives a relative usage measure rather than raw read counts alone.

## Related tools

- **InPACT_transcript** (Assembles novel IPA isoforms from predict.result.txt into an augmented merged GTF with the annotated RefSeq GTF) — https://github.com/YY-TMU/InPACT
- **gffread** (Extracts transcript sequences from merged.gtf for Salmon index building)
- **Salmon** (Indexes transcript sequences (k=31) and quantifies transcript TPM for annotated and novel IPA isoforms) — https://github.com/COMBINE-lab/salmon
- **InPACT_quantify** (Computes per-site IPA usage (PAU) from Salmon quant.sf, merged.gtf, and predict.result.txt) — https://github.com/YY-TMU/InPACT
- **HISAT2** (Aligns raw reads to GRCh38 to produce the coordinate-sorted BAM input required by InPACT)

## Examples

```
InPACT_transcript --predict_terminal predict.result.txt --annotated_gtf RefSeq.gtf --fa_path genome.fa --save_gtf merged.gtf; gffread merged.gtf -g genome.fa -w transcripts.fa; salmon index -t transcripts.fa -i transcripts_index -k 31; InPACT_quantify --transcript_tpm quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt
```

## Evaluation signals

- ipa_usage.txt contains all six expected columns including the IPA usage (PAU) estimate, matching the README output schema (Terminal exon, IPA type, Gene, Upstream coordinate, PolyAsite, IPA usage)
- IPA types in output are restricted to 'Skipped' or 'Composite', with counts per category summing to the total novel sites
- Novel site counts are reproducible between biological replicates (HEK293 replicates reported 471 and 393 novel IPA sites with ~218 overlapping and Pearson r = 0.94 for novel isoform expression)
- Per-cell or per-condition IPA usage values are in the expected relative-usage range after filtering TPM < 1 in scRNA-seq applications
- Downstream totals match reported benchmarks when the full pipeline is reproduced (e.g., 2635 novel IPA events across 2157 genes in fetal bone marrow scRNA-seq; 2977 sites = 1105 skipped + 1872 composite in monocytes)

## Limitations

- Default read thresholds (minimum 5 spliced reads for skipped, 10 boundary-crossing reads for composite terminal exons) mean low-coverage events will be missed; sequencing depth affects sensitivity
- Novel IPA isoforms are only considered translated in ribosome profiling analysis if they have an in-frame stop codon
- InPACT takes about an hour to run the README test file using five cores; runtime scales with input size
- Salmon 2.0 changed the index format, so indexes must be rebuilt when migrating from the C++ 1.x releases

## Evidence

- [readme] InPACT_transcript --predict_terminal predict.result.txt --annotated_gtf RefSeq.gtf --fa_path genome.fa --save_gtf merged.gtf: "InPACT_transcript --predict_terminal predict.result.txt --annotated_gtf RefSeq.gtf --fa_path genome.fa --save_gtf merged.gtf"
- [readme] InPACT_quantify --transcript_tpm quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt: "InPACT_quantify --transcript_tpm quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt"
- [readme] Extract the sequence of transcripts using gffread ... salmon index -t transcripts.fa -i transcripts_index -k 31: "gffread merged.gtf -g genome.fa -w transcripts.fa"
- [methods] The sequence module incorporates a convolutional neural network (CNN) with multiple convolutional and pooling layers: "The sequence module incorporates a convolutional neural network (CNN) with multiple convolutional and pooling layers"
- [methods] low-abundance genes with TPM < 1 were filtered out: "low-abundance genes with TPM < 1 were filtered out"
- [intro] InPACT can reproducibly and accurately identify IPA sites and reconstruct IPA isoforms: "InPACT can reproducibly and accurately identify IPA sites and reconstruct IPA isoforms."
