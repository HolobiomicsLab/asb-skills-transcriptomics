---
name: starsolo-alignment-dge-generation
description: Use when raw SeqScope-style 2nd-seq cDNA fastq.gz files and a STARsolo-compatible
  barcode whitelist are available and a digital expression matrix is needed for spatial
  transcriptomics analysis.
when_to_use_negative:
- Input is already a processed DGE or Seurat object — skip directly to Step C1/C2
  instead of re-running alignment
- Only a barcode/HDMI density estimate of tissue boundary is needed — Step A2 handles
  that without STARsolo alignment
- Downstream clustering/visualization is the goal and a DGE already exists — use Steps
  C3/V2 rather than regenerating the matrix
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: STtools
  role: Pipeline wrapper; Step A3 takes whitelist.txt, 2nd-seq fastq.gz and STAR indices
    to run STARsolo alignment
  repo: https://github.com/seqscope/STtools
- name: STARsolo / STAR
  role: Aligns each cDNA sequence to the reference genome and generates the DGE; requires
    STAR >=2.7.5c
  repo: https://github.com/alexdobin/STAR
- name: seqtk
  role: Used in upstream Step A1 to extract spatial coordinates and build the whitelist
    that this step consumes
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:d5ec96e602f616b3b135f81981c80b6e9f5c5374fe33547c4019a4a5d313fb9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STARsolo
  - STARsolo / STAR
  - STtools
  - seqtk
  - spacemake
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# STARSolo alignment and DGE generation

## Summary

Runs STARsolo (STAR >=2.7.5c) within the STtools Step A3 to align SeqScope 2nd-seq cDNA FASTQ reads against a mm10 STAR genome index using an error-corrected spatial-barcode whitelist, producing digital expression matrices (Gene, GeneFull, and Velocyto variants).

## When to use

Use when raw SeqScope-style 2nd-seq cDNA fastq.gz files and a STARsolo-compatible barcode whitelist are available and a digital expression matrix is needed for spatial transcriptomics analysis.

## When NOT to use

- Input is already a processed DGE or Seurat object — skip directly to Step C1/C2 instead of re-running alignment
- Only a barcode/HDMI density estimate of tissue boundary is needed — Step A2 handles that without STARsolo alignment
- Downstream clustering/visualization is the goal and a DGE already exists — use Steps C3/V2 rather than regenerating the matrix

## Inputs

- Valid barcodes whitelist.txt (error-corrected spatial barcode whitelist from Step A1)
- 2nd-seq cDNA paired fastq.gz files (e.g. liver_tile2106_sub_R1.fastq.gz and R2)
- STAR genome index for the reference genome (e.g. prebuilt mm10 STAR index)

## Outputs

- Digital expression matrix (DGE) in Gene format
- Digital expression matrix in GeneFull format
- Digital expression matrix in Velocyto format (spliced/unspliced, for Step V1)

## How to apply

First run STtools Step A1 with seqtk on the 1st-seq spatial-barcode FASTQ to extract spatial coordinates (.txt) and generate the error-corrected barcode whitelist (whitelist.txt) for STARsolo. Then invoke STtools Step A3 (via sttools.py) with the whitelist.txt, the 2nd-seq paired fastq.gz files, and a prebuilt mm10 STAR genome index (STAR >=2.7.5c) to run STARsolo alignment of each cDNA sequence to the reference genome. STARsolo outputs the digital expression matrix in Gene, GeneFull, and Velocyto formats, which feed downstream STtools steps C1/C2 (grid-collapsed Seurat objects) and V1 (subcellular spliced/unspliced plots). Validate that the DGE output files exist and are non-empty before proceeding. In the published benchmark this step on the SRR14082757 mouse liver subset (104.7G bases) completed within a 254-minute STtools runtime, roughly four times faster than spacemake's 1100 minutes.

## Related tools

- **STtools** (Pipeline wrapper; Step A3 takes whitelist.txt, 2nd-seq fastq.gz and STAR indices to run STARsolo alignment) — https://github.com/seqscope/STtools
- **STARsolo / STAR** (Aligns each cDNA sequence to the reference genome and generates the DGE; requires STAR >=2.7.5c) — https://github.com/alexdobin/STAR
- **seqtk** (Used in upstream Step A1 to extract spatial coordinates and build the whitelist that this step consumes) — https://github.com/lh3/seqtk

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --seqscope1st 'HiSeq' --clustering False --lane-tiles 1_2106 --binsize 300 --window 150 -l 20 -o 'Sample' -c 2
```

## Evaluation signals

- Step A3 DGE output files (Gene, GeneFull, Velocyto) exist and are non-empty
- The whitelist.txt consumed by STARsolo matches the error-corrected whitelist produced by Step A1 seqtk extraction
- Alignment uses STAR >=2.7.5c, the minimum version required by STtools for STARsolo support
- On the SRR14082757 mouse liver subset, wall-clock runtime reproduces the reported ~4x speedup over spacemake (254 min vs 1100 min)

## Limitations

- Step A3 requires a prebuilt STAR genome index; users must build one (e.g. from mm10 fasta + gtf with sjdbOverhang 99) or supply a prebuilt index
- The DGE from Step A3 is extremely sparse at SeqScope resolution (UMI count per spatial barcode is typically much lower at higher resolution), limiting direct per-barcode clustering
- Subcellular analysis with the sliding-window strategy (MSSW) is not feasible — spatial segments must be larger than subcellular compartments (~10um)

## Evidence

- [readme] Step A3 takes valid barcodes whitelist.txt, 2nd-seq fastq.gz files, and the STAR indices of reference genome as input to run STARsolo alignment: "**Step A3** takes valid barcodes `whitelist.txt`, 2nd-seq `fastq.gz` files, and the STAR indices of reference genome as input to run `STARsolo` alignment"
- [methods] STtools aligns each cDNA sequence to the reference genome using STARsolo: "STtools aligns each cDNA sequence to the reference genome using STARsolo (Kaminow et al., 2021)."
- [results] we ran experiments for a subset (SRR14082757) with 104.7G bases: "we ran experiments for a subset (SRR14082757) with 104.7G bases"
- [readme] STAR>=2.7.5c requirement: "* STAR>=2.7.5c (Click for instructions to install [STAR](https://github.com/alexdobin/STAR))"
- [intro] the number of spatial barcodes per mm2 rapidly increases as resolution increases: "the number of spatial barcodes per mm2 rapidly increases as resolution increases (~120 for VISIUM, ~3K for Slide-seq, >1M for Seq-Scope)"
