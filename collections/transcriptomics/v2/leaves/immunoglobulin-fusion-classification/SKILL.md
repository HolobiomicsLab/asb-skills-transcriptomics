---
name: immunoglobulin-fusion-classification
description: Use when analyzing Smart-Seq scRNA-seq FASTQs (e.g. myeloma, T cells)
  to detect recurrent gene fusions per cell and test whether fusion carriage drives
  target-gene overexpression.
when_to_use_negative:
- Data are 10X droplet scRNA-seq rather than Smart-Seq single-cell libraries — scFusion
  is built 'for Smart-Seq data rather than 10X'.
- Fusions of interest are rare in highly heterogeneous tumor samples or have very
  low expression, where scFusion's detection power is limited.
- Only bulk RNA-seq data are available; scFusion is designed for scRNA-seq and jointly
  leverages multiple single cells from a sample.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0622
tools:
- name: scFusion
  role: 'Driver pipeline: statistical model, deep-learning model, and joint single-cell
    analysis to call fusions'
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Aligns each single-cell FASTQ and emits chimeric/split reads for candidate
    fusion pairs
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: Processes STAR alignments during candidate extraction
- name: bedtools
  role: Extracts chimeric/split read candidate fusion pairs from alignments
- name: pyensembl
  role: Annotates candidate fusion gene pairs via GTF gene_name and gene_type/gene_biotype
    fields
  repo: https://github.com/openvax/pyensembl
- name: TensorFlow/Keras
  role: Deep-learning model used to filter false-positive fusion candidates
- name: stringr (R)
  role: Quantifies WHSC1 expression in fusion-positive versus fusion-negative cells
- name: pysam
  role: Python module used by the pipeline to work with BAM/alignment files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:2926521c8db29483cd0bf335be1dace94da0cc99940b51f6bf1ca6ed2a76a4ca
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
  - stringr (R)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# immunoglobulin-fusion-classification

## Summary

A single-cell fusion-detection skill using scFusion (STAR chimeric reads + statistical/deep-learning models) to identify recurrent immunoglobulin gene fusions such as IgH-WHSC1 in Smart-Seq scRNA-seq and link them to oncogene overexpression.

## When to use

Use when analyzing Smart-Seq scRNA-seq FASTQs (e.g. myeloma, T cells) to detect recurrent gene fusions per cell and test whether fusion carriage drives target-gene overexpression.

## When NOT to use

- Data are 10X droplet scRNA-seq rather than Smart-Seq single-cell libraries — scFusion is built 'for Smart-Seq data rather than 10X'.
- Fusions of interest are rare in highly heterogeneous tumor samples or have very low expression, where scFusion's detection power is limited.
- Only bulk RNA-seq data are available; scFusion is designed for scRNA-seq and jointly leverages multiple single cells from a sample.

## Inputs

- Smart-Seq single-cell FASTQ files (*_1.fastq, *_2.fastq)
- Reference genome FASTA (e.g. hg19.fa, ~3G)
- Ensembl GTF annotation file (*.gtf)
- hg19 4-column UCSC mappability file

## Outputs

- Per-cell and per-patient fusion call table (fusion_results.csv) with gene pairs and breakpoints
- IgH-WHSC1 breakpoint table (e.g. 1902353 in SMM0, 1905943 in RRMM2 cells)
- WHSC1 expression comparison between fusion-positive and fusion-negative cells

## How to apply

Place Smart-Seq FASTQ files (named *_1.fastq / *_2.fastq; *_1.fq is not allowed), the hg19 reference FASTA, an Ensembl GTF, and the hg19 4-column mappability file in the working folder, then run the scFusion.py driver (optionally via the jzj2035198/scfusion Docker image). Align each cell with STAR >= 2.7.2d, process with samtools and bedtools to extract chimeric/split reads as candidate pairs, and annotate candidates with pyensembl using gene_name and gene_type/gene_biotype fields. scFusion's statistical model and deep-learning model (tensorflow/keras) plus the mappability filter remove false positives; jointly analyze all cells (e.g. 597 myeloma cells) so fusions shared across cells are boosted. Tabulate fusions per cell and per patient — expect the recurrent IgH-WHSC1 fusion at breakpoint 1902353 in all 52 SMM0 cells and 1905943 in all 20 RRMM2 cells, with 38 total fusions of which 94.7% involve immunoglobulin genes. Finally quantify the target gene (WHSC1) in fusion-positive versus fusion-negative cells (R with stringr) to confirm the fusion-associated overexpression of Fig 7d-e.

## Related tools

- **scFusion** (Driver pipeline: statistical model, deep-learning model, and joint single-cell analysis to call fusions) — https://github.com/ZijieJin/scFusion
- **STAR** (Aligns each single-cell FASTQ and emits chimeric/split reads for candidate fusion pairs) — https://github.com/alexdobin/STAR
- **samtools** (Processes STAR alignments during candidate extraction)
- **bedtools** (Extracts chimeric/split read candidate fusion pairs from alignments)
- **pyensembl** (Annotates candidate fusion gene pairs via GTF gene_name and gene_type/gene_biotype fields) — https://github.com/openvax/pyensembl
- **TensorFlow/Keras** (Deep-learning model used to filter false-positive fusion candidates)
- **stringr (R)** (Quantifies WHSC1 expression in fusion-positive versus fusion-negative cells)
- **pysam** (Python module used by the pipeline to work with BAM/alignment files)

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- The recurrent IgH-WHSC1 fusion is called with breakpoints at 1902353 in all 52 SMM0 cells and 1905943 in all 20 RRMM2 cells.
- Fusion summary shows 38 total fusions, 94.7% of which involve immunoglobulin genes.
- WHSC1 expression is significantly higher in fusion-positive cells, and WHSC1 coverage sharply increases downstream of the breakpoints (Fig 7d-e).
- The false-positive rate stays low: statistical model, deep-learning model, and mappability filter are all active (mappability filter is off if the file is omitted).
- Output fusion_results.csv contains per-cell fusion calls, breakpoint table, and WHSC1 expression summary.

## Limitations

- scFusion has reduced sensitivity for fusions with low expression, common to all RNA-based methods.
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; overcome only by sequencing more cells and/or deeper.
- Turning off the deep-learning model reduces precision; many artifacts are also removed only by ad hoc filters.
- scFusion did not find WHSC1 fusions in single cells from RRMM2 — WHSC1 there may be activated by other mechanisms or the fusion was missed.
- Mappability filter is disabled if the mappability file is not provided.

## Evidence

- [abstract] scFusion detects IgH-WHSC1: "in a multiple myeloma dataset, scFusion detects the known recurrent fusion IgH-WHSC1, which is associated with overexpression of the WHSC1 oncogene."
- [methods] scFusion.py pipeline: "Download all scripts, unzip the hg19mappability file in the data folder, and run scFusion.py."
- [readme] Smart-Seq input naming: "Single cell RNA sequencing files from Smart-Seq protocol. File names should be *_1.fastq, *_2.fastq (e.g. 1_1.fastq, 1_2.fastq, 2_1.fastq, 2_2.fastq)"
- [discussion] deep-learning filter: "When deep-learning model was turned off, the precisions of scFusion reduced"
- [discussion] low-expression sensitivity: "scFusion has reduced sensitivity for fusions with low expression (as shown in the simulation), just as is the case for all RNA-based methods"
- [discussion] RRMM2 WHSC1 miss: "scFusion did not find WHSC1 fusions in single cells from RRMM2. It is likely that the expression of WHSC1 was activated in RRMM2 by mechanisms other than WHSC1 fusions"
