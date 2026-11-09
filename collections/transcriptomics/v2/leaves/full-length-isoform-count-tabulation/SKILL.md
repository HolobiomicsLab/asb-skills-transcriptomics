---
name: full-length-isoform-count-tabulation
description: Use when comparing bulk-mode short-read transcriptome assemblers on simulated
  or real genes (e.g. GYPC, MAPT, BIN1) and you need a per-gene, per-assembler tally
  of uniquely recovered full-length isoforms.
when_to_use_negative:
- Input is barcoded linked-read RNA-seq intended for barcode-aware assembly — use
  cloudrnaSPAdes instead of barcode-ignoring bulk assemblers for this data type
- No reference annotation is available, since IsoQuant-based contig-to-isoform assignment
  requires it
- Reads are long-read (PacBio CLR/ONT), which SPAdes-based pipelines only use for
  gap closure and repeat resolution rather than primary transcript assembly here
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Trinity
  role: Standard bulk-mode de novo transcriptome assembler run ignoring barcode information;
    its uniquely-assigned full-length isoform counts are compared against rnaSPAdes
- name: RNA-Bloom
  role: De novo transcriptome assembler run with default settings on the same read
    sets as a third barcode-ignoring baseline
- name: rnaSPAdes
  role: De novo transcriptome assembler from the SPAdes toolkit, invoked via rnaspades.py
    with automatic k-mer selection; produces contigs.fasta for IsoQuant assignment
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: cloudrnaSPAdes
  role: Barcode-aware reference-free isoform assembler whose recovery of full-length
    isoforms from barcoded RNA-seq serves as the upper bound that standard assemblers
    are compared against
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: IsoQuant
  role: Assigns contigs from each assembler's contigs.fasta to genes and isoforms
    against the reference annotation, enabling unique full-length isoform tallies
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:529590d7d58e9793b7bb5382b8020419718881dbe5d20debba8ab492aa8808fe
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# full-length-isoform-count-tabulation

## Summary

Benchmark short-read de novo RNA assemblers (Trinity, RNA-Bloom, rnaSPAdes) that ignore barcode information on complex multi-isoform genes by assigning their contigs to reference isoforms and tabulating uniquely-assigned full-length isoform counts per gene per assembler.

## When to use

Use when comparing bulk-mode short-read transcriptome assemblers on simulated or real genes (e.g. GYPC, MAPT, BIN1) and you need a per-gene, per-assembler tally of uniquely recovered full-length isoforms.

## When NOT to use

- Input is barcoded linked-read RNA-seq intended for barcode-aware assembly — use cloudrnaSPAdes instead of barcode-ignoring bulk assemblers for this data type
- No reference annotation is available, since IsoQuant-based contig-to-isoform assignment requires it
- Reads are long-read (PacBio CLR/ONT), which SPAdes-based pipelines only use for gap closure and repeat resolution rather than primary transcript assembly here

## Inputs

- Per-gene paired-end short-read FASTQ sets from simulated complex genes (GYPC, MAPT, BIN1)
- Reference transcript annotation (for IsoQuant gene/isoform assignment)
- contigs.fasta assemblies from Trinity, RNA-Bloom, and rnaSPAdes runs

## Outputs

- isoform_counts_per_gene.tsv — uniquely-assigned full-length isoform counts per gene per assembler

## How to apply

Run each assembler in standard bulk mode on the same per-gene read sets, deliberately ignoring any barcode information: Trinity in default bulk mode, RNA-Bloom with default de novo transcriptome settings, and rnaSPAdes via the rnaspades.py executable with k-mer sizes selected automatically from the read length (note that --careful is not supported in rnaSPAdes). Collect each assembler's contigs.fasta output. Assign contigs to genes and isoforms using IsoQuant against the reference annotation, then for each gene and assembler identify the isoforms that receive a unique full-length contig assignment and tally these counts. Write the counts to isoform_counts_per_gene.tsv; the rationale is that standard short-read assemblers leave most isoforms of complex genes undetected, so unique full-length assignments provide a stringent recovery metric.

## Related tools

- **Trinity** (Standard bulk-mode de novo transcriptome assembler run ignoring barcode information; its uniquely-assigned full-length isoform counts are compared against rnaSPAdes)
- **RNA-Bloom** (De novo transcriptome assembler run with default settings on the same read sets as a third barcode-ignoring baseline)
- **rnaSPAdes** (De novo transcriptome assembler from the SPAdes toolkit, invoked via rnaspades.py with automatic k-mer selection; produces contigs.fasta for IsoQuant assignment) — https://github.com/1dayac/cloudrnaSPAdes-release
- **cloudrnaSPAdes** (Barcode-aware reference-free isoform assembler whose recovery of full-length isoforms from barcoded RNA-seq serves as the upper bound that standard assemblers are compared against) — https://github.com/1dayac/cloudrnaSPAdes-release
- **IsoQuant** (Assigns contigs from each assembler's contigs.fasta to genes and isoforms against the reference annotation, enabling unique full-length isoform tallies)

## Examples

```
rnaspades.py -1 GYPC_R1.fastq.gz -2 GYPC_R2.fastq.gz -o rnaSPAdes_GYPC
```

## Evaluation signals

- isoform_counts_per_gene.tsv exists and contains one row per gene (GYPC, MAPT, BIN1) per assembler (Trinity, RNA-Bloom, rnaSPAdes) with integer isoform counts
- Per the validation criterion, Trinity recovers more uniquely-assigned full-length isoforms than rnaSPAdes on these simulated gene datasets
- Barcode-aware pipelines detect far more isoforms than any of the three standard assemblers (Trinity, RNA-Bloom, rnaSPAdes)
- Every isoform counted received a unique full-length contig assignment from IsoQuant against the reference annotation (no ambiguous or partial assignments tallied)
- Each assembler run used automatic k-mer size selection from read length (default behavior) and no unsupported flags such as --careful for rnaSPAdes

## Limitations

- Standard short-read RNA assemblers ignore barcode information and left most isoforms of the simulated complex genes undetected, so counts are low and the metric mainly demonstrates their ceiling
- rnaSPAdes does not support the --careful option, so caution-mode settings used for other SPAdes modes cannot be applied
- K values are selected automatically based on read length and dataset type, limiting fine-grained control over k-mer sizes

## Evidence

- [results] Trinity outperformed rnaSPAdes on these datasets: [evidence span withheld — anchor status: not_found]
- [readme] rnaSPAdes de novo transcriptome assembler: "rnaSPAdes &ndash; a *de novo* transcriptome assembler from RNA-Seq data (see [rnaSPAdes manual](rnaspades_manual.html))."
- [readme] Automatic K selection: "SPAdes &ndash; iterative short-read genome assembly module; values of K are selected automatically based on the read length and data set type."
- [abstract] cloudrnaSPAdes assembles full-length isoforms from barcoded reads: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [abstract] Barcode-aware upper bound on simulated and real data: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
