---
name: spatial-transcriptomics-pipeline-execution
description: Use when raw Seq-Scope 1st-seq/2nd-seq FASTQ files (spatial barcodes
  in Illumina sequence identifiers plus cDNA reads) must be converted into DGE matrices,
  Seurat objects, clustering, and subcellular plots at ultra-high spatial resolution
  (>1M barcodes/mm2).
when_to_use_negative:
- Input is already a processed DGE matrix or Seurat object — use only the downstream
  C-steps or V-steps rather than --run-all
- Analysis goal requires subcellular resolution with the sliding-window strategy —
  spatial segments must be larger than subcellular compartments (~10um), so MSSW subcellular
  analysis is not feasible
- Data requires histological-image overlay, spatial expression imputation, or joint
  cellular/subcellular clustering — these features are not implemented in STtools
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STtools
  role: Master pipeline orchestrating steps A1–V2 via sttools.py --run-all
  repo: https://github.com/seqscope/STtools
- name: STARsolo
  role: Aligns each cDNA sequence to the reference genome and produces the DGE matrices
    consumed by C1/C3/V1
  repo: https://github.com/alexdobin/STAR
- name: STAR >=2.7.5c
  role: Required aligner binary and genome-index builder for the A3 STARsolo step
  repo: https://github.com/alexdobin/STAR
- name: seqtk
  role: Sequence toolkit invoked during FASTQ preprocessing in the alignment steps
  repo: https://github.com/lh3/seqtk
- name: Seurat
  role: R framework used in C1/C2/C3 for grid-collapsed Seurat objects, dimension
    reduction, clustering, and reference mapping
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:61f4ec896982de76d218a283e291e462a60a30ab6e3aa7786f917d4f2fd77961
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STAR >=2.7.5c
  - STARsolo
  - STtools
  - Seurat
  - seqtk
  - spacemake
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spatial-transcriptomics-pipeline-execution

## Summary

Execute the eight-step STtools pipeline (A1–V2) to process raw Seq-Scope/Slide-seq/VISIUM spatial transcriptomics FASTQ data through alignment, grid-based clustering, and visualization, scaling to >1M spatial barcodes. The pipeline was demonstrated end-to-end on the full mouse liver Seq-Scope dataset GSM5212844 (15M spatial barcodes, 1.9B raw reads) in ~16 hours on 6 CPUs and 30GB RAM.

## When to use

Use when raw Seq-Scope 1st-seq/2nd-seq FASTQ files (spatial barcodes in Illumina sequence identifiers plus cDNA reads) must be converted into DGE matrices, Seurat objects, clustering, and subcellular plots at ultra-high spatial resolution (>1M barcodes/mm2).

## When NOT to use

- Input is already a processed DGE matrix or Seurat object — use only the downstream C-steps or V-steps rather than --run-all
- Analysis goal requires subcellular resolution with the sliding-window strategy — spatial segments must be larger than subcellular compartments (~10um), so MSSW subcellular analysis is not feasible
- Data requires histological-image overlay, spatial expression imputation, or joint cellular/subcellular clustering — these features are not implemented in STtools

## Inputs

- Seq-Scope 1st-seq FASTQ (fastq.gz) with spatial coordinates in Illumina sequence identifiers
- Seq-Scope 2nd-seq paired FASTQ (fastq.gz) with cDNA reads
- STAR genome index directory (e.g. mm10 FASTA + GTF with --sjdbOverhang 99)
- Path to seqtk binary
- Path to STAR binary
- Pipeline parameters: --seqscope1st, --lane-tiles, --binsize, --window, clustering flag

## Outputs

- Spatial coordinates .txt files and STARsolo barcode whitelist (A1)
- Barcode/HDMI density plot for tissue-boundary QC (A2)
- Digital expression matrix (DGE) in Gene/GeneFull and Velocyto formats from STARsolo (A3)
- Seurat objects with DGE collapsed into simple square grids (C1) and sliding-window grids (C2)
- RDS objects with dimension reduction, clustering, and reference mapping results (C3)
- Subcellular spliced/unspliced plots (V1) and UMAP, spatial clustering, and feature violin plots (V2)
- pipeline_run_log.md with per-step completion and total runtime

## How to apply

Clone seqscope/STtools, install STAR>=2.7.5c, seqtk, R>=4.0.0, Python 3, perl, and pigz, set STHOME/STDATA/STOUT/SEQTKPATH/STARPATH/GENOMEINDEX environment variables, and build a STAR genome index (e.g. mm10 with --sjdbOverhang 99). Launch with `python3 sttools.py --run-all` supplying the 1st-seq FASTQ (spatial-coordinate extraction, Step A1), the 2nd-seq FASTQ pair, the STAR genome index, and STAR/seqtk paths, plus platform parameters such as --seqscope1st 'HiSeq', --lane-tiles, --binsize 300, and --window 150 for the sliding-window strategy. The run executes all eight steps: A1 extracts spatial coordinates and STARsolo whitelists, A2 generates barcode/HDMI density plots for tissue-boundary QC, A3 runs STARsolo to produce Gene/GeneFull/Velocyto DGE matrices, C1/C2 collapse barcodes into simple square grids and sliding-window grids as Seurat objects, C3 performs dimension reduction, clustering, and reference mapping, and V1/V2 generate subcellular spliced/unspliced plots and UMAP/spatial-clustering/feature-violin plots. Choose bin size and window parameters according to the target resolution, remembering that spatial segments must exceed ~10um, so subcellular analysis with MSSW is not feasible. Monitor on an HPC node and record per-step completion and wall-clock time; the full 625.1G-base GSM5212844 dataset completes in ~16 hours with 6 CPUs and 30GB RAM.

## Related tools

- **STtools** (Master pipeline orchestrating steps A1–V2 via sttools.py --run-all) — https://github.com/seqscope/STtools
- **STARsolo** (Aligns each cDNA sequence to the reference genome and produces the DGE matrices consumed by C1/C3/V1) — https://github.com/alexdobin/STAR
- **STAR >=2.7.5c** (Required aligner binary and genome-index builder for the A3 STARsolo step) — https://github.com/alexdobin/STAR
- **seqtk** (Sequence toolkit invoked during FASTQ preprocessing in the alignment steps) — https://github.com/lh3/seqtk
- **Seurat** (R framework used in C1/C2/C3 for grid-collapsed Seurat objects, dimension reduction, clustering, and reference mapping)

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --seqscope1st 'HiSeq' --binsize 300 --window 150
```

## Evaluation signals

- All eight steps (A1–V2) complete without error and each step's declared outputs (whitelist.txt, DGE, Seurat/RDS objects, plots) are present
- Step A2 barcode/HDMI density plot is consistent with expected tissue boundary when compared with H&E imagery
- STARsolo step A3 yields Gene, GeneFull, and Velocyto matrices from the whitelist and 2nd-seq FASTQs
- Runtime scales plausibly with input size (e.g. full GSM5212844, 625.1G bases, ~16h at 6 CPUs/30GB RAM)
- C3 RDS objects show dimension reduction, clustering, and reference mapping results; V2 outputs UMAP and spatial-clustering plots

## Limitations

- Requires a Linux operating system (e.g. Ubuntu 18.04) and externally installed STAR>=2.7.5c, seqtk, perl, and pigz
- Subcellular analysis with the sliding-window strategy (MSSW) is not feasible because each spatial segment must be larger than subcellular compartments (~10um)
- No spatial smoothing, expression imputation, joint cellular/subcellular clustering, or automatic overlay of histological images; these are noted as future additions
- Thresholds to filter genes and spatial segments before clustering must be chosen manually

## Evidence

- [methods] STtools workflow currently performs three major tasks – alignment, clustering, and visualization – consisting of eight individual steps: "STtools workflow currently performs three major tasks – alignment, clustering, and visualization – consisting of eight individual steps (Figure S1)."
- [methods] STARsolo alignment role: "STtools aligns each cDNA sequence to the reference genome using STARsolo (Kaminow et al., 2021)."
- [readme] Step A1 description: "**Step A1** takes `fastq.gz` files as input and output spatial coordinates `.txt` files and whitelist used for `STARsolo` alignemnt in the current working directory."
- [readme] Software requirements: "Linux operatin system is necessary to run STtools package. You also need to install the following software tools and librares/modules before using this package."
- [results] Comparative scalability result: "we ran both tools with the same mouse liver SeqScope data (GSM5212844)"
- [intro] Barcode density scaling challenge: "the number of spatial barcodes per mm2 rapidly increases as resolution increases (~120 for VISIUM, ~3K for Slide-seq, >1M for Seq-Scope), and few tools seamlessly scale"
