---
name: spatial-expression-matrix-processing
description: Use when raw FASTQ reads from ultra-high-resolution ST platforms (Seq-Scope
  <1μm, Slide-seq 10μm, VISIUM 100μm) must be converted into a spatially annotated
  DGE with millions of spatial barcodes per mm².
when_to_use_negative:
- Input is already a DGE with spatial coordinates (externally supplied matrix) — skip
  alignment steps A1–A3 and go directly to visualization or clustering
- Data are single-cell RNA-seq without spatial barcodes — STARsolo spatial whitelisting
  and coordinate extraction do not apply
- Subcellular-resolution inference via the MSSW (sliding window) strategy is requested
  — spatial segments must exceed ~10μm, larger than subcellular compartments, so MSSW
  subcellular analysis is not feasible
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0203
tools:
- name: STtools
  role: 'Pipeline driver executing Steps A1–A3 and C1–C3: coordinate extraction, whitelisting,
    STARsolo alignment, and DGE collapsing into Seurat objects'
  repo: https://github.com/seqscope/STtools
- name: STARsolo
  role: Aligns cDNA sequences to the reference genome and generates the digital expression
    matrix for valid spatial barcodes
  repo: https://github.com/alexdobin/STAR
- name: seqtk
  role: Preprocessing of raw FASTQ sequence reads within the STtools pipeline
  repo: https://github.com/lh3/seqtk
- name: Seurat
  role: Target object format for collapsed DGEs (Step C1/C2) used for clustering and
    reference mapping (Step C3)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:dc67f43444c510f69640bb5947376f113b3b0769391e114cfda0711ca936814b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STARsolo
  - STtools
  - Seurat
  - seqtk
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# spatial-expression-matrix-processing

## Summary

Process raw Seq-Scope/Slide-seq/VISIUM FASTQ data into spatially annotated digital expression matrices (DGE) with STtools, using STARsolo alignment against a STAR genome index and retaining per-barcode spatial coordinates for downstream ultra-high-resolution analysis.

## When to use

Use when raw FASTQ reads from ultra-high-resolution ST platforms (Seq-Scope <1μm, Slide-seq 10μm, VISIUM 100μm) must be converted into a spatially annotated DGE with millions of spatial barcodes per mm².

## When NOT to use

- Input is already a DGE with spatial coordinates (externally supplied matrix) — skip alignment steps A1–A3 and go directly to visualization or clustering
- Data are single-cell RNA-seq without spatial barcodes — STARsolo spatial whitelisting and coordinate extraction do not apply
- Subcellular-resolution inference via the MSSW (sliding window) strategy is requested — spatial segments must exceed ~10μm, larger than subcellular compartments, so MSSW subcellular analysis is not feasible

## Inputs

- 1st-seq fastq.gz files carrying spatial barcode information (e.g. liver-MiSeq-tile2106-sub-R1.fastq.gz)
- 2nd-seq paired fastq.gz cDNA files (e.g. liver_tile2106_sub_R1/R2.fastq.gz)
- STAR genome index built from reference fasta and GTF (e.g. mm10) with sjdbOverhang 99
- seqtk and STAR binaries (STAR>=2.7.5c)

## Outputs

- Spatial coordinates .txt files and barcode whitelist.txt (Step A1)
- Barcode/HDMI density plot for tissue boundary estimation (Step A2)
- Digital expression matrix (DGE) and Velocyto-format DGE from STARsolo (Step A3)
- Seurat RDS objects with collapsed DGE of simple square grids (Step C1) and sliding-window grids (Step C2)
- Clustered/mapped Seurat object after dimension reduction and reference mapping (Step C3)

## How to apply

Run STtools Step A1 on the 1st-seq fastq.gz files to extract spatial coordinates and a barcode whitelist, Step A2 to generate a barcode/HDMI density plot for estimating tissue boundaries against HE images, and Step A3 to execute STARsolo alignment (STAR >=2.7.5c index, e.g. mm10 gtf/fasta) of 2nd-seq reads, producing the digital expression matrix and Velocyto-format DGE. Follow with Step C1 (collapse DGE into simple square grids, e.g. binsize 300) and Step C2 (sliding-window grids with window 150) as Seurat objects, then Step C3 for dimension reduction, clustering, and reference mapping with simple square grids as query. For sub-micrometer RGB geneset or subcellular plots, skip spatial binning and aggregate marker geneset expression per spatial barcode directly from the Step A3 DGE so sub-micrometer barcode resolution is retained. Sequence preprocessing uses seqtk; whitelists restrict STARsolo to valid spatial barcodes.

## Related tools

- **STtools** (Pipeline driver executing Steps A1–A3 and C1–C3: coordinate extraction, whitelisting, STARsolo alignment, and DGE collapsing into Seurat objects) — https://github.com/seqscope/STtools
- **STARsolo** (Aligns cDNA sequences to the reference genome and generates the digital expression matrix for valid spatial barcodes) — https://github.com/alexdobin/STAR
- **seqtk** (Preprocessing of raw FASTQ sequence reads within the STtools pipeline) — https://github.com/lh3/seqtk
- **Seurat** (Target object format for collapsed DGEs (Step C1/C2) used for clustering and reference mapping (Step C3))

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --seqscope1st 'HiSeq' --clustering False --lane-tiles 1_2106 --binsize 300 --window 150 -l 20 -o 'Sample' -c 2
```

## Evaluation signals

- Step A1 outputs spatial coordinates .txt files and whitelist.txt in the working directory
- Step A2 produces a barcode/HDMI density plot whose tissue boundary is consistent with the HE image
- Step A3 yields a DGE whose barcodes are restricted to the whitelist (valid spatial barcodes) and a companion Velocyto-format DGE
- Step C1/C2 produce Seurat RDS objects with collapsed grid DGEs at the requested binsize (e.g. 300) and window (e.g. 150)
- Downstream RGB geneset plots resolve cell-type (periportal vs pericentral) and subcellular patterns at <1 μm²/pixel when no spatial binning is applied

## Limitations

- MSSW sliding-window spatial segments must be larger than subcellular compartments (~10um), so subcellular analysis with MSSW is not feasible
- Spatial smoothing algorithms for extremely sparse per-barcode UMI profiles are not yet available in STtools
- No method to impute spatial expression profiles, no joint clustering of cellular and subcellular components, and no automatic overlay of histological images and spatial expressions
- UMI count per spatial barcode is much lower at higher resolution, which can degrade tools assuming sufficient UMIs per barcode

## Evidence

- [methods] STtools aligns each cDNA sequence to the reference genome using STARsolo (Kaminow et al., 2021).: "STtools aligns each cDNA sequence to the reference genome using STARsolo (Kaminow et al., 2021)."
- [readme] Step A1 description: "**Step A1** takes `fastq.gz` files as input and output spatial coordinates `.txt` files and whitelist used for `STARsolo` alignemnt in the current working directory."
- [readme] Step A3 description: "**Step A3** takes valid barcodes `whitelist.txt`, 2nd-seq `fastq.gz`
  files, and the STAR indices of reference genome as input to run
  `STARsolo` alignment; this step outputs digital expression"
- [readme] Step C1 description: "**Step C1** takes DGE from **Step A3** and output `Seurat` object with collapsed DGE of simple square grids."
- [intro] Scalability challenge: "the number of spatial barcodes per mm2 rapidly increases as resolution increases (~120 for VISIUM, ~3K for Slide-seq, >1M for Seq-Scope), and few tools seamlessly scale"
- [discussion] MSSW subcellular limitation: "each spatial segment still needs to be larger than subcellular compartments (e.g. ~10um), so subcellular analysis with MSSW is not feasible"
