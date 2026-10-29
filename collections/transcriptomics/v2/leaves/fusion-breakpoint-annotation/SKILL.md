---
name: fusion-breakpoint-annotation
description: Use when Smart-Seq single-cell FASTQ files need fusion detection and
  breakpoint calling, and when fusion-bearing versus fusion-negative cells must be
  compared for oncogene overexpression (e.g. WHSC1 after IgH-WHSC1).
when_to_use_negative:
- Data are 10X droplet scRNA-seq rather than Smart-Seq full-transcript libraries —
  scFusion is designed for Smart-Seq data, not 10X
- Input is bulk RNA-seq or an existing fusion call set needing no per-cell resolution
- FASTQ filenames follow the *_1.fq convention, which scFusion explicitly does not
  allow
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3522
tools:
- name: scFusion
  role: 'Main pipeline: statistical + deep-learning fusion detection jointly across
    single cells, run via scFusion.py'
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Alignment of single-cell libraries and detection of chimeric/split reads (>=
    2.7.2d)
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: Processing of STAR alignments for chimeric read extraction (>= 1.0, tested
    on 1.10)
- name: bedtools
  role: Genomic interval operations on candidate fusion breakpoints (tested on 2.29.2)
- name: pyensembl
  role: Annotation of candidate fusion gene pairs using gene_name and gene_type/gene_biotype
    from the GTF
  repo: https://github.com/openvax/pyensembl
- name: stringr
  role: R package used to quantify and compare WHSC1 expression in fusion-positive
    vs fusion-negative cells
- name: TensorFlow/Keras
  role: Deep-learning model that filters false-positive fusion candidates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:a5c1604e7f46dfb5901b7a41cecde9eda6c894816d814a68c866ac8be443bdde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Python
  - R
  - STAR
  - TensorFlow
  - TensorFlow/Keras
  - bedtools
  - pyensembl
  - pysam
  - samtools
  - scFusion
  - stringr
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3522
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# fusion-breakpoint-annotation

## Summary

Annotate and validate scRNA-seq-derived gene fusion calls (e.g. IgH-WHSC1 in multiple myeloma) at single-cell resolution using the scFusion pipeline: STAR alignment, chimeric-read extraction, statistical/deep-learning false-positive control, and downstream expression quantification of the fused oncogene.

## When to use

Use when Smart-Seq single-cell FASTQ files need fusion detection and breakpoint calling, and when fusion-bearing versus fusion-negative cells must be compared for oncogene overexpression (e.g. WHSC1 after IgH-WHSC1).

## When NOT to use

- Data are 10X droplet scRNA-seq rather than Smart-Seq full-transcript libraries — scFusion is designed for Smart-Seq data, not 10X
- Input is bulk RNA-seq or an existing fusion call set needing no per-cell resolution
- FASTQ filenames follow the *_1.fq convention, which scFusion explicitly does not allow

## Inputs

- Smart-Seq single-cell FASTQ files named *_1.fastq and *_2.fastq (e.g. 1_1.fastq, 1_2.fastq)
- Reference genome FASTA (e.g. hg19.fa, ~3G)
- GTF annotation file from Ensembl, NCBI, or UCSC
- hg19 4-column mappability file (hg19mappability, optional but enables the mappability filter)

## Outputs

- Per-cell and per-patient fusion call table with gene pairs and genomic breakpoint coordinates (fusion_results.csv)
- WHSC1 expression summary comparing fusion-positive and fusion-negative cells
- Downstream-of-breakpoint coverage profile for the fused oncogene

## How to apply

Place Smart-Seq FASTQ pairs (*_1.fastq / *_2.fastq; *_1.fq is not allowed), the hg19 reference genome FASTA, an Ensembl GTF, and the hg19 4-column mappability file in the working folder and run the scFusion.py driver (optionally via Docker image jzj2035198/scfusion). Align each single-cell library with STAR (>= 2.7.2d) and extract chimeric/split reads with samtools and bedtools as candidate fusion pairs. Annotate candidate gene pairs with pyensembl (gene_name, gene_type/gene_biotype from the GTF) and pass candidates through the statistical model, the tensorflow/keras deep-learning model, and the mappability filter; jointly analyze all cells (e.g. 597 myeloma cells) so fusions shared across cells are boosted. Tabulate fusions per cell and per patient, verifying recurrent breakpoints (IgH-WHSC1 at position 1902353 in all 52 SMM0 cells and 1905943 in all 20 RRMM2 cells), then quantify WHSC1 expression in fusion-positive versus fusion-negative cells with R/stringr and write results to fusion_results.csv.

## Related tools

- **scFusion** (Main pipeline: statistical + deep-learning fusion detection jointly across single cells, run via scFusion.py) — https://github.com/ZijieJin/scFusion
- **STAR** (Alignment of single-cell libraries and detection of chimeric/split reads (>= 2.7.2d)) — https://github.com/alexdobin/STAR
- **samtools** (Processing of STAR alignments for chimeric read extraction (>= 1.0, tested on 1.10))
- **bedtools** (Genomic interval operations on candidate fusion breakpoints (tested on 2.29.2))
- **pyensembl** (Annotation of candidate fusion gene pairs using gene_name and gene_type/gene_biotype from the GTF) — https://github.com/openvax/pyensembl
- **stringr** (R package used to quantify and compare WHSC1 expression in fusion-positive vs fusion-negative cells)
- **TensorFlow/Keras** (Deep-learning model that filters false-positive fusion candidates)

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- scFusion calls the known recurrent IgH-WHSC1 fusion in the multiple myeloma dataset
- 38 total fusions detected, of which 94.7% involve immunoglobulin genes
- IgH-WHSC1 breakpoints reproduce at position 1902353 in all 52 SMM0 cells and 1905943 in all 20 RRMM2 cells
- Cells with IgH-WHSC1 fusions show significantly higher WHSC1 expression than fusion-negative cells (Fig 7d-e)
- WHSC1 sequencing coverage sharply increases downstream of the fusion breakpoints

## Limitations

- Reduced sensitivity for fusions with low expression, common to all RNA-based methods (shown in simulation)
- Limited detection power for rare fusions in highly heterogeneous tumor samples; overcome only by sequencing more and/or deeper cells
- Turning off the deep-learning model reduces precision; many potential artifacts are caught by the ad hoc filters
- scFusion did not find WHSC1 fusions in single cells from RRMM2, possibly because WHSC1 was activated by other mechanisms or the fusion was missed
- Omitting the mappability file turns off the mappability filter, potentially increasing false positives

## Evidence

- [abstract] scFusion detects IgH-WHSC1 in myeloma: "in a multiple myeloma dataset, scFusion detects the known recurrent fusion IgH-WHSC1, which is associated with overexpression of the WHSC1 oncogene."
- [intro] scFusion employs statistical + DL models: "scFusion employs a statistical model and a deep-learning model to control for false positives."
- [readme] Smart-Seq FASTQ naming requirement: "Single cell RNA sequencing files from Smart-Seq protocol. File names should be *_1.fastq, *_2.fastq (e.g. 1_1.fastq, 1_2.fastq, 2_1.fastq, 2_2.fastq) ** *_1.fq is not allowed.**"
- [methods] STAR version requirement: "STAR (https://github.com/alexdobin/STAR) >= 2.7.2d (tested on 2.7.2d and 2.7.8a)"
- [discussion] Joint analysis boosts shared fusions: "Another important difference compared to existing bulk methods is that scFusion takes advantage of the fact that multiple cells from a sample should contain the same fusion."
- [methods] mappability filter optional: "If you do not provide this file, scFusion will turn off the mappability filter."
