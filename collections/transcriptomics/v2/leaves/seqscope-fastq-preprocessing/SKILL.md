---
name: seqscope-fastq-preprocessing
description: Use when raw Seq-Scope FASTQ files must be converted into spatial coordinate
  files, STARsolo barcode whitelists, HDMI density QC plots, and digital expression
  matrices — i.e., 1st-seq spatial-barcode FASTQ plus 2nd-seq cDNA FASTQ pairs are
  the starting point.
when_to_use_negative:
- Input is already a digital expression matrix or Seurat object rather than raw Seq-Scope
  FASTQ — start instead at the C1/C2 grid-collapsing steps.
- The data come from VISIUM or Slide-seq with existing expression matrices and spatial
  coordinates, for which STtools provides separate example inputs rather than 1st/2nd-seq
  FASTQ preprocessing.
- You cannot supply a STAR genome index or install STAR >=2.7.5c, seqtk, R >=4.0.0,
  Python 3, perl, and pigz, since A3 alignment requires STARsolo with a reference
  index.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: STtools
  role: Pipeline driver executing Steps A1 (coordinate extraction + whitelist), A2
    (barcode/HDMI density QC), and A3 (STARsolo alignment) via sttools.py --run-all
    or per-step invocations
  repo: https://github.com/seqscope/STtools
- name: STAR
  role: Aligner (>=2.7.5c) whose STARsolo mode maps the 2nd-seq cDNA reads against
    the genome index using the A1 whitelist to generate the DGE
  repo: https://github.com/alexdobin/STAR
- name: seqtk
  role: Lightweight FASTQ processing utility invoked by the pipeline (SEQTKPATH) during
    read preprocessing
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:7038fc885d9aa390e0d2180e98b15f07078402ed9740ed4c6299571182c26895
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STAR
  - STtools
  - seqtk
  - spacemake
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# seqscope-fastq-preprocessing

## Summary

Preprocessing of raw Seq-Scope FASTQ data with STtools: extract spatial barcode coordinates from the 1st-seq FASTQ (A1), generate barcode/HDMI density QC plots (A2), and run STARsolo alignment of the 2nd-seq cDNA pair to produce digital expression matrices (A3).

## When to use

Use when raw Seq-Scope FASTQ files must be converted into spatial coordinate files, STARsolo barcode whitelists, HDMI density QC plots, and digital expression matrices — i.e., 1st-seq spatial-barcode FASTQ plus 2nd-seq cDNA FASTQ pairs are the starting point.

## When NOT to use

- Input is already a digital expression matrix or Seurat object rather than raw Seq-Scope FASTQ — start instead at the C1/C2 grid-collapsing steps.
- The data come from VISIUM or Slide-seq with existing expression matrices and spatial coordinates, for which STtools provides separate example inputs rather than 1st/2nd-seq FASTQ preprocessing.
- You cannot supply a STAR genome index or install STAR >=2.7.5c, seqtk, R >=4.0.0, Python 3, perl, and pigz, since A3 alignment requires STARsolo with a reference index.

## Inputs

- 1st-seq FASTQ file (fastq.gz) containing spatial barcodes with pixel coordinates in the Illumina sequence identifiers
- 2nd-seq paired FASTQ files (fastq.gz) containing cDNA reads
- STAR genome index (built from reference FASTA and GTF, e.g. mm10)
- Paths to STAR (>=2.7.5c) and seqtk binaries

## Outputs

- Spatial coordinates .txt files
- whitelist.txt of valid barcodes for STARsolo
- Barcode/HDMI density plot (QC against HE tissue boundary)
- Digital expression matrix (DGE) from STARsolo, including Gene, GeneFull, and Velocyto-format matrices

## How to apply

Run STtools Step A1 on the 1st-seq fastq.gz (spatial barcode with pixel coordinates encoded in the Illumina sequence identifier, sequenced on HiSeq/MiSeq), passing --seqscope1st ('HiSeq') and --lane-tiles (e.g. 1_2106) so it outputs spatial coordinates .txt files and a whitelist.txt of valid barcodes for STARsolo. Run Step A2 with the barcode info and spatial coordinates file to generate a barcode/HDMI density plot that can be compared with HE images for estimating the tissue boundary. Then run Step A3 with whitelist.txt, the 2nd-seq FASTQ pair, and the STAR genome index (STAR >=2.7.5c) to execute STARsolo alignment, yielding Gene/GeneFull/Velocyto digital expression matrices. The full sequence can be launched as 'python3 sttools.py --run-all' with --first-fq, --second-fq1/--second-fq2, --genome, --star-path and --seqtk-path; the rationale is that millions of Seq-Scope spatial barcodes (>1M per mm2) cannot be handled by standard ST pipelines, so coordinate extraction and barcode whitelisting precede STARsolo.

## Related tools

- **STtools** (Pipeline driver executing Steps A1 (coordinate extraction + whitelist), A2 (barcode/HDMI density QC), and A3 (STARsolo alignment) via sttools.py --run-all or per-step invocations) — https://github.com/seqscope/STtools
- **STAR** (Aligner (>=2.7.5c) whose STARsolo mode maps the 2nd-seq cDNA reads against the genome index using the A1 whitelist to generate the DGE) — https://github.com/alexdobin/STAR
- **seqtk** (Lightweight FASTQ processing utility invoked by the pipeline (SEQTKPATH) during read preprocessing) — https://github.com/lh3/seqtk

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --seqscope1st 'HiSeq' --lane-tiles 1_2106
```

## Evaluation signals

- Step A1 must produce spatial coordinates .txt files and a whitelist.txt in the current working directory suitable for STARsolo.
- The Step A2 barcode/HDMI density plot should visually agree with the HE image tissue boundary estimate.
- Step A3 must emit STARsolo digital expression matrices (Gene, GeneFull, Velocyto-format) without alignment errors from STAR >=2.7.5c.
- Barcode whitelist size should reflect Seq-Scope scale (>1M spatial barcodes per mm2), not the ~120/mm2 VISIUM or ~3K/mm2 Slide-seq scale.
- The --seqscope1st flag must match the sequencer identifier format (e.g. 'HiSeq') and --lane-tiles (e.g. 1_2106) must match the 1st-seq tile, otherwise coordinate extraction fails.

## Limitations

- Requires a Linux OS and installation of STAR>=2.7.5c, seqtk, R>=4.0.0, Python 3, perl, and pigz before preprocessing can run.
- To incorporate transgenes into alignment, the GTF and FASTA must be modified (per igordot/genomics ref-genome-gfp.md) before building the STAR index.
- The sequencing-platform encoding (--seqscope1st, e.g. 'HiSeq') and lane-tile selection must be correct; misconfiguration yields empty or wrong spatial coordinate outputs.

## Evidence

- [methods] **Step A1** takes `fastq.gz` files as input and output spatial coordinates `.txt` files and whitelist used for `STARsolo` alignemnt: "**Step A1** takes `fastq.gz` files as input and output spatial coordinates `.txt` files and whitelist used for `STARsolo` alignemnt"
- [methods] **Step A2** takes barcode info, and spatial coordinates file to generate a barcode/HDMI density plot: "**Step A2** takes barcode info, and spatial coordinates file to generate a barcode/HDMI density plot"
- [readme] The STTools pipeline includes preprocessing of raw sequence reads, alignment, collapsing barcodes into grids, clustering cell types, and high-resolution analysis with sliding window strategy.: "The STTools pipeline includes preprocessing of raw sequence reads, alignment, collapsing barcodes into grids, clustering cell types, and high-resolution analysis with sliding window strategy."
- [readme] export SEQTKPATH=/path/to/seqtk/bin  ## path that contains seqtk binary: "export SEQTKPATH=/path/to/seqtk/bin  ## path that contains seqtk binary"
