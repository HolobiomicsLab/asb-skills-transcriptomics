---
name: single-cell-read-downsampling
description: 'Use when planning or budgeting single-cell mtDNA clonality experiments:
  the input is an analysis-ready cell-barcoded BAM and the goal is to determine the
  minimum reads-per-cell depth at which variant calling (MQuad) and clonal assignment
  (vireoSNP) remain adequate.'
when_to_use_negative:
- The data are already at or near full depth and the goal is definitive clone calling
  rather than depth assessment — run the full-depth cellSNP-MQuad-vireoSNP pipeline
  directly without downsampling.
- The aim is comparing variant-calling methods (e.g., MQuad vs mgatk or EMBLEM) rather
  than characterizing the effect of sequencing depth on a fixed pipeline.
- The dataset is mtscATAC-seq or Smart-seq2 scRNA-seq, where the article's depth recommendation
  (3,000 mt reads/cell for 10x scDNA-seq) and processing route do not directly apply.
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: cellsnp-lite
  role: Piles up mitochondrial reads from each downsampled BAM into SNP-by-cell AD/DP
    matrices (v1.2.1) feeding the depth grid.
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MQuad
  role: Selects clonally informative mtDNA variants at each depth (v0.1.6) using a
    binomial mixture model with a Kneedle ΔBIC cutoff.
  repo: https://github.com/single-cell-genetics/MQuad
- name: vireoSNP
  role: Reconstructs clonal populations per depth (v0.5.3) via variational inference
    on the MQuad-selected SNPs, giving the concordance labels.
  repo: https://github.com/single-cell-genetics/vireo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:829828ea499d9c5bc44bc14bbe389ad8255ec2a1c59371468448a3ff1a4a3076
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/cellsnp-lite
  tools:
  - MQuad
  - cellsnp-lite
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-read-downsampling

## Summary

Systematically downsample single-cell sequencing reads (e.g., the MKN-45 10x scDNA-seq BAM aligned to hg38) to a grid of decreasing mitochondrial reads-per-cell depths and quantify how depth affects MQuad mtDNA variant detection and clonal assignment, establishing a minimum-depth recommendation (3,000 mitochondrial reads per cell for MQuad on scDNA-seq).

## When to use

Use when planning or budgeting single-cell mtDNA clonality experiments: the input is an analysis-ready cell-barcoded BAM and the goal is to determine the minimum reads-per-cell depth at which variant calling (MQuad) and clonal assignment (vireoSNP) remain adequate.

## When NOT to use

- The data are already at or near full depth and the goal is definitive clone calling rather than depth assessment — run the full-depth cellSNP-MQuad-vireoSNP pipeline directly without downsampling.
- The aim is comparing variant-calling methods (e.g., MQuad vs mgatk or EMBLEM) rather than characterizing the effect of sequencing depth on a fixed pipeline.
- The dataset is mtscATAC-seq or Smart-seq2 scRNA-seq, where the article's depth recommendation (3,000 mt reads/cell for 10x scDNA-seq) and processing route do not directly apply.

## Inputs

- Analysis-ready single-cell BAM file (10x Genomics MKN-45 scDNA-seq aligned to hg38) with cell barcodes
- Cell barcode whitelist/TSF list for the 10x run
- Downsampling grid of target mitochondrial reads-per-cell depths
- Full-depth clone labels as reference for concordance comparison

## Outputs

- Per-depth SNP-by-cell AD/DP pile-up matrices from cellsnp-lite
- Per-depth MQuad-selected informative mtDNA variants
- Per-depth vireoSNP clone assignment per cell
- A summary table keyed by reads-per-cell with columns: assignable cells, informative mtDNA variants detected, and clonal-assignment concordance vs full depth
- A minimum-depth recommendation (3,000 mitochondrial reads per cell for MQuad on scDNA-seq)

## How to apply

Start from an analysis-ready BAM (for MKN-45, 10x Genomics scDNA-seq aligned to hg38) and downsample reads to a grid of decreasing reads-per-cell depths (e.g., with samtools view -s or equivalent random subsampling). At each depth, pile up mitochondrial reads with cellsnp-lite (v1.2.1) to generate SNP-by-cell AD/DP matrices, then run MQuad (v0.1.6) selecting informative variants with the binomial mixture model and the ΔBIC knee-point cutoff from the Kneedle algorithm (kneed v0.7.0), removing error-prone rCRS regions 302–315, 513–525, and 3105–3109. Reconstruct clones at each depth with vireoSNP (v0.5.3) variational inference on the MQuad-selected SNPs. Record, per depth, the number of cells with assignable clones, the number of informative mtDNA variants detected, and clonal-assignment concordance against full-depth clone labels, assembling these into a single table keyed by reads per cell. The correct application is confirmed when the table shows the reported linear drop in assignable cells, informative variants, and concordance with decreasing depth, supporting the minimum of 3,000 mitochondrial reads per cell for MQuad on scDNA-seq.

## Related tools

- **cellsnp-lite** (Piles up mitochondrial reads from each downsampled BAM into SNP-by-cell AD/DP matrices (v1.2.1) feeding the depth grid.) — https://github.com/single-cell-genetics/cellsnp-lite
- **MQuad** (Selects clonally informative mtDNA variants at each depth (v0.1.6) using a binomial mixture model with a Kneedle ΔBIC cutoff.) — https://github.com/single-cell-genetics/MQuad
- **vireoSNP** (Reconstructs clonal populations per depth (v0.5.3) via variational inference on the MQuad-selected SNPs, giving the concordance labels.) — https://github.com/single-cell-genetics/vireo

## Examples

```
samtools view -s $FRACTION $BAM > ds.bam && cellSNP -s ds.bam -b $BARCODE -O $OUT_DIR -p 22 --chrom chrMT
```

## Evaluation signals

- The depth table shows a linear (monotonic decreasing) relationship between reads per cell and each of: assignable cells, informative mtDNA variants detected, and clonal concordance with full-depth labels.
- The depth at which the recommendation is supported matches ~3,000 mitochondrial reads per cell for MQuad on the MKN-45 scDNA-seq data.
- Full-depth (undownsampled) run reproduces the original MKN-45 clone structure, so concordance at 100% depth is 1.0 by construction.
- At every depth, MQuad variant selection applies the same ΔBIC knee-point cutoff and excludes error-prone rCRS regions 302–315, 513–525, 3105–3109, so differences across depths are attributable to depth alone.
- Each downsampled pile-up yields valid SNP-by-cell AD/DP matrices consumable by MQuad without reformatting.

## Limitations

- The 3,000 mitochondrial reads-per-cell recommendation is derived from the MKN-45 scDNA-seq dataset and may not transfer to other assays (Smart-seq2 scRNA-seq, mtscATAC-seq) or other cell lines.
- MQuad's informative-variant selection depends on the ΔBIC knee-point cutoff from the Kneedle algorithm, so knee-point instability at low depths can affect apparent variant counts.
- Mitochondrial reads pile-up quality depends on upstream alignment/reference choice — the 10x MKN-45 BAMs are aligned to hg38, different from the hg19-processed reference used for the authors' other datasets.

## Evidence

- [discussion] Per-depth pile-up tool: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23. This generates a SNP-by-cell matrix"
- [discussion] Per-depth variant selection: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
- [discussion] ΔBIC cutoff determination: "a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm36"
- [discussion] Per-depth clonal reconstruction: "vireoSNP (v0.5.3)24 uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad"
- [readme] Recommended pipeline order from README: "use cellSNP or cellsnp-lite (a faster version of cellSNP, still at testing stage so might be unstable) to pileup mtDNA variants from raw .bam file(s)"
