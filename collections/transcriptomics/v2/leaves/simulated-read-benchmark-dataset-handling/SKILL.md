---
name: simulated-read-benchmark-dataset-handling
description: Use when comparing transcriptome assemblers on complex multi-isoform
  genes (e.g., GYPC, MAPT, BIN1) from simulated barcoded RNA-seq reads, and you need
  per-gene full-length isoform recovery counts to rank standard bulk assemblers vs
  barcode-aware methods.
when_to_use_negative:
- The reads carry barcode information that the pipeline should exploit — this skill
  deliberately ignores barcodes to model standard bulk short-read assemblers
- The goal is de novo isoform recovery from real (not simulated) barcoded linked-read
  data, where cloudrnaSPAdes itself is the appropriate tool
- Assessing genome rather than transcriptome assembly — rnaSPAdes is a transcriptome
  assembler and these evaluation criteria (full-length isoform recovery) do not apply
edam_operation: http://edamontology.org/operation_3458
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Trinity
  role: Bulk de novo transcriptome assembler run in standard mode, ignoring barcode
    information, as one baseline assembler
- name: RNA-Bloom
  role: De novo transcriptome assembler run with default settings as a second baseline
    assembler
- name: rnaSPAdes
  role: Transcriptome assembler run via rnaspades.py with automatic k-mer size selection
    as a third baseline assembler
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: cloudrnaSPAdes
  role: Barcode-aware assembler that serves as the comparison target whose isoform
    recovery the standard assemblers are benchmarked against
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: IsoQuant
  role: Assigns contigs from each assembler's contigs.fasta to genes and isoforms
    using the reference annotation
- name: SPAdes
  role: Assembly toolkit from which rnaSPAdes is invoked; K values are selected automatically
    based on read length and data set type
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:bf7209e978ec89a985e774486dbb62f8d3fb1bfb438a0d5797ccb4a3ad4cbedd
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
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_3458
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# simulated-read-benchmark-dataset-handling

## Summary

Benchmarking short-read RNA assemblers (Trinity, RNA-Bloom, rnaSPAdes) against barcode-aware pipelines on simulated genes (GYPC, MAPT, BIN1) by counting uniquely-assigned full-length isoforms recovered per gene per assembler using IsoQuant against a reference annotation.

## When to use

Use when comparing transcriptome assemblers on complex multi-isoform genes (e.g., GYPC, MAPT, BIN1) from simulated barcoded RNA-seq reads, and you need per-gene full-length isoform recovery counts to rank standard bulk assemblers vs barcode-aware methods.

## When NOT to use

- The reads carry barcode information that the pipeline should exploit — this skill deliberately ignores barcodes to model standard bulk short-read assemblers
- The goal is de novo isoform recovery from real (not simulated) barcoded linked-read data, where cloudrnaSPAdes itself is the appropriate tool
- Assessing genome rather than transcriptome assembly — rnaSPAdes is a transcriptome assembler and these evaluation criteria (full-length isoform recovery) do not apply

## Inputs

- Simulated short-read RNA-seq read sets for complex genes GYPC, MAPT, and BIN1
- Reference genome annotation for isoform assignment
- Assembler outputs: contigs.fasta from Trinity, RNA-Bloom, and rnaSPAdes

## Outputs

- isoform_counts_per_gene.tsv — table of uniquely-assigned full-length isoform counts per gene per assembler
- Per-gene list of isoforms with unique full-length contig assignments for each assembler

## How to apply

Run each standard assembler in bulk mode on the same simulated gene read sets, ignoring barcode information: Trinity in standard bulk mode, RNA-Bloom with default de novo transcriptome settings, and rnaSPAdes via rnaspades.py with k-mer sizes selected automatically from the read length (note: --careful is not supported in rnaSPAdes). Take each assembler's contigs.fasta output and assign contigs to genes and isoforms with IsoQuant against the reference annotation. For each gene and each assembler, identify isoforms receiving a unique full-length contig assignment and tally these counts. Write the tallies to isoform_counts_per_gene.tsv; a correct run shows Trinity recovering more uniquely-assigned full-length isoforms than rnaSPAdes, and all three standard assemblers detecting far fewer isoforms than barcode-aware pipelines.

## Related tools

- **Trinity** (Bulk de novo transcriptome assembler run in standard mode, ignoring barcode information, as one baseline assembler)
- **RNA-Bloom** (De novo transcriptome assembler run with default settings as a second baseline assembler)
- **rnaSPAdes** (Transcriptome assembler run via rnaspades.py with automatic k-mer size selection as a third baseline assembler) — https://github.com/1dayac/cloudrnaSPAdes-release
- **cloudrnaSPAdes** (Barcode-aware assembler that serves as the comparison target whose isoform recovery the standard assemblers are benchmarked against) — https://github.com/1dayac/cloudrnaSPAdes-release
- **IsoQuant** (Assigns contigs from each assembler's contigs.fasta to genes and isoforms using the reference annotation)
- **SPAdes** (Assembly toolkit from which rnaSPAdes is invoked; K values are selected automatically based on read length and data set type) — https://github.com/1dayac/cloudrnaSPAdes-release

## Examples

```
rnaspades.py -1 GYPC_1.fastq.gz -2 GYPC_2.fastq.gz -o gypc_rnaspades_out  # then run IsoQuant on contigs.fasta against the reference annotation
```

## Evaluation signals

- isoform_counts_per_gene.tsv exists and contains one row per gene (GYPC, MAPT, BIN1) per assembler with full-length isoform counts
- Trinity's uniquely-assigned full-length isoform count exceeds rnaSPAdes's count on the simulated genes
- Total isoforms detected by barcode-aware pipelines far exceeds the counts from all three standard assemblers combined
- Each counted isoform is confirmed as having a unique full-length contig assignment in the IsoQuant output against the reference annotation
- rnaSPAdes runs complete successfully without invoking unsupported options such as --careful

## Limitations

- Standard short-read assemblers ignore barcode information and left most isoforms of the complex simulated genes undetected, so they are not appropriate for isoform-level recovery of highly diverse genes
- rnaSPAdes underperformed Trinity on these datasets; it also does not support the --careful option available elsewhere in the SPAdes toolkit
- SPAdes/rnaSPAdes was initially designed for small genomes and 'is not intended for larger genomes (e.g. mammalian size genomes)'

## Evidence

- [results] On the simulated genes, standard short-read RNA assemblers left most isoforms undetected, and Trinity outperformed rnaSPAdes on these datasets.: [evidence span withheld — anchor status: not_found]
- [methods] rnaSPAdes k-mer selection: "SPAdes – iterative short-read genome assembly module; values of K are selected automatically based on the read length and data set type."
- [abstract] cloudrnaSPAdes accuracy: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [readme] supported pipelines: "rnaSPAdes – a de novo transcriptome assembler from RNA-Seq data (see [rnaSPAdes manual](rnaspades_manual.html))."
