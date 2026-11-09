---
name: contig-to-isoform-assignment
description: Use when you have assembled isoform contigs from barcoded or bulk RNA-seq
  (contigs.fasta) and need to quantify which annotated GENCODE isoforms they recover,
  per gene, with assignment-class statistics.
when_to_use_negative:
- Input is still raw barcoded FASTQ reads rather than assembled contigs — run read
  error correction (BayesHammer) and cloudrnaSPAdes assembly first
- The target organism lacks a curated transcript annotation (e.g. no GENCODE GTF),
  since unique isoform-level assignment depends on a reference transcript set
- The goal is reference-free isoform assembly itself rather than evaluating assembled
  contigs against an annotation
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3173
tools:
- name: IsoQuant
  role: Assigns assembled contigs to reference transcripts against GENCODE v32/GRCh38
    and classifies each assignment (unique, minor-difference, ambiguous, inconsistent,
    uninformative)
- name: cloudrnaSPAdes
  role: Produces the barcoded linked-read isoform contigs (contigs.fasta) that are
    subsequently assigned to GENCODE isoforms
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: Bandage
  role: Visualizes the SPAdes assembly graph paths underlying contigs when inspecting
    ambiguous isoform assignments
  repo: https://github.com/rrwick/Bandage/wiki/Graph-paths
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:495c8cb23b5db79ddbe3d523bfebd865506ff23c056eea59799ab84334320699
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - Bandage
  - BayesHammer
  - InSilicoSeq
  - IsoQuant
  - RNA-Bloom
  - SPAdes
  - Trinity
  - cloudrnaSPAdes
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# contig-to-isoform-assignment

## Summary

Assign de novo assembled isoform contigs (e.g. contigs.fasta from cloudrnaSPAdes) to annotated reference transcripts using IsoQuant against GENCODE v32 on GRCh38, classifying each contig as unique, minor-difference, ambiguous, inconsistent, or uninformative and computing per-gene isoform precision/recall.

## When to use

Use when you have assembled isoform contigs from barcoded or bulk RNA-seq (contigs.fasta) and need to quantify which annotated GENCODE isoforms they recover, per gene, with assignment-class statistics.

## When NOT to use

- Input is still raw barcoded FASTQ reads rather than assembled contigs — run read error correction (BayesHammer) and cloudrnaSPAdes assembly first
- The target organism lacks a curated transcript annotation (e.g. no GENCODE GTF), since unique isoform-level assignment depends on a reference transcript set
- The goal is reference-free isoform assembly itself rather than evaluating assembled contigs against an annotation

## Inputs

- contigs.fasta from cloudrnaSPAdes/rnaSPAdes/Trinity/RNA-Bloom assembly, filtered to contigs > 300 bp
- GENCODE v32 GTF transcript annotation
- GRCh38 genome reference FASTA

## Outputs

- Per-contig IsoQuant assignment class (unique, minor-difference, ambiguous, inconsistent, uninformative)
- isoform_assignment_counts.csv: total contigs >300 bp plus count per assignment class
- gene_isoform_assignment_metrics.tsv: per-gene precision, recall, and fixed-recall from IsoQuant assignments

## How to apply

Assemble each gene (or the whole transcriptome) with cloudrnaSPAdes so that contigs.fasta contains candidate isoform sequences, then extract contigs longer than 300 bp before assignment. Run IsoQuant on the filtered contigs against the GENCODE v32 GTF annotation on the GRCh38 genome reference so each contig is assigned to a reference transcript and classified as unique, minor-difference, ambiguous, inconsistent, or uninformative. Tabulate the total contig count (>300 bp) plus one count per assignment class into a summary such as isoform_assignment_counts.csv, checking that class counts reconcile with the assembler and IsoQuant logs. For benchmark-style evaluation, compute per-gene precision, recall, and 'Fixed recall' (after filtering read clouds that produced multiple isoforms for the same gene) into gene_isoform_assignment_metrics.tsv; on the simulated GYPC, MAPT, and BIN1 datasets more than 98% of assembled isoforms should be accurately assigned, and the filtered run should show a recall gain over the unfiltered run.

## Related tools

- **IsoQuant** (Assigns assembled contigs to reference transcripts against GENCODE v32/GRCh38 and classifies each assignment (unique, minor-difference, ambiguous, inconsistent, uninformative))
- **cloudrnaSPAdes** (Produces the barcoded linked-read isoform contigs (contigs.fasta) that are subsequently assigned to GENCODE isoforms) — https://github.com/1dayac/cloudrnaSPAdes-release
- **Bandage** (Visualizes the SPAdes assembly graph paths underlying contigs when inspecting ambiguous isoform assignments) — https://github.com/rrwick/Bandage/wiki/Graph-paths

## Evaluation signals

- On the simulated GYPC, MAPT, and BIN1 datasets, more than 98% of assembled isoforms receive accurate IsoQuant assignment to reference isoforms
- isoform_assignment_counts.csv contains the total contig count (>300 bp) plus one count per IsoQuant class, and class counts sum consistently with the assembler and IsoQuant run logs
- For the real SPIso-seq human brain data, class counts should reproduce the reported scale (1,463,514 unique and 55,611 minor-difference assignments against GENCODE v32 + GRCh38)
- The 'Fixed recall' run (after barcode-based filtering of read clouds yielding multiple isoforms per gene) shows a recall gain over the unfiltered run, exceeding the 'Separate strategy' competitor on MAPT and BIN1
- Contig set used for assignment is restricted to sequences longer than 300 bp in contigs.fasta

## Limitations

- Read clouds producing multiple isoforms for the same gene yield ambiguous assignments that inflate naive recall; the 'Fixed recall' filtering procedure is needed for a fair per-gene recall estimate on complex genes like MAPT (16 isoforms) and BIN1 (15 isoforms)
- Assignment quality depends on the specific reference annotation build (GENCODE v32 on GRCh38); switching annotation versions changes class counts
- Only contigs longer than 300 bp are counted, so short isoform fragments are excluded from assignment statistics

## Evidence

- [abstract] we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [abstract] Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity.: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [methods] MismatchCorrector improves contigs before assignment: "MismatchCorrector – a tool which improves mismatch and short indel rates in resulting contigs and scaffolds"
- [readme] BayesHammer error correction precedes assembly: "BayesHammer &ndash; read error correction tool for Illumina reads, which works well on both single-cell and standard data sets."
