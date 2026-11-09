---
name: assembler-benchmark-comparison
description: Use when comparing transcriptome assemblers on complex multi-isoform
  genes from short-read or barcoded linked-read RNA-seq, where the metric is the count
  of annotated isoforms receiving a unique IsoQuant assignment per assembler.
when_to_use_negative:
- The input is already assembled contigs/transcripts and the goal is only IsoQuant
  assignment or quantification — no assembler comparison is being made.
- The task is assembly of whole large genomes with the SPAdes toolkit; the README
  states SPAdes 'is not intended for larger genomes (e.g. mammalian size genomes)'.
- You need barcode-aware isoform assembly from barcoded linked reads as the end product
  rather than an assembler-vs-assembler comparison.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: cloudrnaSPAdes
  role: Barcode-aware assembler under test; feature release of the SPAdes assembler
    that assembles full-length isoforms from barcoded RNA-seq linked-read data reference-free
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: rnaSPAdes
  role: Standard de novo transcriptome assembler comparator, run via rnaspades.py
    with automatic k-mer selection
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: Trinity
  role: Standard short-read bulk-mode comparator that ignores barcode information
- name: RNA-Bloom
  role: Standard comparator run with default de novo transcriptome settings; treated
    as pipeline failure when its k-mer counting step fails
- name: IsoQuant
  role: Assigns assembled contigs to genes and annotated isoforms; unique single-isoform
    assignments are the benchmark metric
- name: pandas
  role: Parses IsoQuant assignment outputs, filters to the ATP6AP2 locus, and compiles
    the tidy per-assembler count table
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:024d431e179171fc4d8617bd9df84d45cccf9acc57e595874972e34cf20e538c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - IsoQuant
  - RNA-Bloom
  - SPAdes
  - Trinity
  - cloudrnaSPAdes
  - pandas
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# assembler-benchmark-comparison

## Summary

Benchmark de novo RNA-seq assemblers (cloudrnaSPAdes, Trinity, rnaSPAdes, RNA-Bloom) on a common read set by assigning each assembler's contigs to annotated reference isoforms with IsoQuant and counting uniquely assigned isoforms per gene (e.g. ATP6AP2 against GENCODE v32).

## When to use

Use when comparing transcriptome assemblers on complex multi-isoform genes from short-read or barcoded linked-read RNA-seq, where the metric is the count of annotated isoforms receiving a unique IsoQuant assignment per assembler.

## When NOT to use

- The input is already assembled contigs/transcripts and the goal is only IsoQuant assignment or quantification — no assembler comparison is being made.
- The task is assembly of whole large genomes with the SPAdes toolkit; the README states SPAdes 'is not intended for larger genomes (e.g. mammalian size genomes)'.
- You need barcode-aware isoform assembly from barcoded linked reads as the end product rather than an assembler-vs-assembler comparison.

## Inputs

- Barcoded RNA-seq linked-read data (e.g. real SPIso-seq human brain dataset)
- Simulated gene read sets for GYPC, MAPT, and BIN1 (standard bulk FASTQ)
- Reference annotation GTF (GENCODE v32, chromosome X for the ATP6AP2 case study)
- Assembler output contigs.fasta files (cloudrnaSPAdes, Trinity, rnaSPAdes, RNA-Bloom)
- IsoQuant transcript assignment output files generated per assembler against the reference annotation

## Outputs

- Tidy CSV of per-assembler unique-isoform counts with columns assembler, uniquely_assigned_isoforms, status (saved as atp6ap2_isoform_assignments.csv)
- TSV of uniquely-assigned full-length isoform counts per gene per assembler (isoform_counts_per_gene.tsv)

## How to apply

Run every assembler on identical input reads under settings that isolate their design differences: Trinity in standard bulk mode ignoring barcode information, RNA-Bloom with default de novo transcriptome settings, and rnaSPAdes via rnaspades.py with k-mer sizes selected automatically from the read length (the --careful flag is not supported in rnaSPAdes). cloudrnaSPAdes is included as the barcode-aware reference-free assembler for barcoded RNA-seq linked-read data. Assign each assembler's contigs.fasta to genes and isoforms using IsoQuant against the reference annotation (e.g. GENCODE v32 GTF), then parse the assignment tables with pandas and filter to the target gene locus (e.g. ATP6AP2 on chromosome X, 34 GENCODE v32 isoforms), restricting matches to annotated isoforms. For each assembler, tally isoforms that receive a unique (unambiguous single-isoform) assignment, and record pipeline failures such as RNA-Bloom's k-mer counting failure as 'failure' rather than as zero recovered isoforms. Compile per-assembler counts into a tidy table with columns assembler, uniquely_assigned_isoforms, status and check them against expected values (e.g. cloudrnaSPAdes 7, Trinity 2, rnaSPAdes 0 for ATP6AP2).

## Related tools

- **cloudrnaSPAdes** (Barcode-aware assembler under test; feature release of the SPAdes assembler that assembles full-length isoforms from barcoded RNA-seq linked-read data reference-free) — https://github.com/1dayac/cloudrnaSPAdes-release
- **rnaSPAdes** (Standard de novo transcriptome assembler comparator, run via rnaspades.py with automatic k-mer selection) — https://github.com/1dayac/cloudrnaSPAdes-release
- **Trinity** (Standard short-read bulk-mode comparator that ignores barcode information)
- **RNA-Bloom** (Standard comparator run with default de novo transcriptome settings; treated as pipeline failure when its k-mer counting step fails)
- **IsoQuant** (Assigns assembled contigs to genes and annotated isoforms; unique single-isoform assignments are the benchmark metric)
- **pandas** (Parses IsoQuant assignment outputs, filters to the ATP6AP2 locus, and compiles the tidy per-assembler count table)

## Examples

```
rnaspades.py -1 gene_R1.fastq.gz -2 gene_R2.fastq.gz -o rnaspades_out  # auto k-mer selection; --careful not supported
```

## Evaluation signals

- Per-assembler ATP6AP2 unique-isoform counts must equal the reported values (cloudrnaSPAdes 7, Trinity 2, rnaSPAdes 0), otherwise the output is rejected.
- The RNA-Bloom run must be recorded with status 'failure' (k-mer counting step), never as zero recovered isoforms.
- Output table schema check: columns must be exactly assembler, uniquely_assigned_isoforms, status.
- On simulated genes (GYPC, MAPT, BIN1), Trinity must show more uniquely-assigned full-length isoforms than rnaSPAdes, and barcode-aware pipelines must detect far more isoforms than any of the three standard assemblers.
- Assignments must be restricted to the 34 annotated GENCODE v32 ATP6AP2 isoforms before counting unambiguous single-isoform assignments.

## Limitations

- RNA-Bloom failed at the k-mer counting step on the real SPIso-seq data, yielding no IsoQuant assignments; this must be reported as a pipeline failure, not as zero recovered isoforms.
- rnaSPAdes does not support the --careful flag, and k-mer sizes are selected automatically from the read length, limiting fine-tuning during comparisons.
- Standard short-read assemblers that ignore barcodes leave most isoforms of complex genes undetected; Trinity recovered only two incomplete ATP6AP2 isoforms with unique assignments.
- The SPAdes toolkit was 'initially designed for small genomes' and 'is not intended for larger genomes (e.g. mammalian size genomes)' per the README.

## Evidence

- [abstract] abstract: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [intro] intro: "cloudrnaSPAdes is a feature release of a SPAdes assembler"
- [readme] readme: "rnaSPAdes – a *de novo* transcriptome assembler from RNA-Seq data"
