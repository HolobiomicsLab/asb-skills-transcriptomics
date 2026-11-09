---
name: vireosnp-clonal-assignment
description: Use when a SNP-by-cell AD/DP mtDNA matrix (from cellsnp-lite/cellSNP)
  has been filtered by MQuad and you need per-cell clone labels from mitochondrial
  variants.
when_to_use_negative:
- Variant calling and informativeness filtering have not yet been done — run MQuad
  first; vireoSNP expects selected SNPs, not raw pile-ups.
- Input data are nuclear SNVs or CNV-derived subclones where the diploid context applies
  — the mtDNA-specific pipeline violates diploid assumptions.
- Per-cell mitochondrial read depth is far below ~3,000 reads, where assignable cells,
  informative variants, and concordance drop linearly and assignments become unreliable.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: vireoSNP
  role: Variational inference clonal reconstruction using the MQuad-selected mtDNA
    SNPs to assign cells to clones
  repo: https://github.com/single-cell-genetics/vireo
- name: MQuad
  role: Upstream selection of high-quality informative mtDNA variants via binomial
    mixture model with ΔBIC Kneedle cutoff
  repo: https://github.com/single-cell-genetics/MQuad
- name: cellsnp-lite
  role: Pile-up of raw mtDNA reads from BAM to generate the SNP-by-cell AD/DP matrices
    consumed by the pipeline
  repo: https://github.com/single-cell-genetics/cellsnp-lite
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:ef8df0d1a4f4f80bba3983864eb0ee2d1751c7ade6596812b1ccf7ce90549536
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/vireo
  tools:
  - MQuad
  - cellsnp-lite
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# vireosnp-clonal-assignment

## Summary

Reconstruct clonal substructure from single-cell mtDNA variant data by running vireoSNP (v0.5.3) variational inference on the MQuad-selected informative mitochondrial SNPs, assigning each cell to a clone in the cellSNP-MQuad-VireoSNP pipeline.

## When to use

Use when a SNP-by-cell AD/DP mtDNA matrix (from cellsnp-lite/cellSNP) has been filtered by MQuad and you need per-cell clone labels from mitochondrial variants.

## When NOT to use

- Variant calling and informativeness filtering have not yet been done — run MQuad first; vireoSNP expects selected SNPs, not raw pile-ups.
- Input data are nuclear SNVs or CNV-derived subclones where the diploid context applies — the mtDNA-specific pipeline violates diploid assumptions.
- Per-cell mitochondrial read depth is far below ~3,000 reads, where assignable cells, informative variants, and concordance drop linearly and assignments become unreliable.

## Inputs

- Analysis-ready single-cell BAM file (e.g., 10x Genomics MKN-45 scDNA-seq aligned to hg38)
- SNP-by-cell AD/DP matrices from cellsnp-lite mitochondrial pile-up
- MQuad-selected informative mtDNA SNPs (ΔBIC knee-point filtered)

## Outputs

- Per-cell clone assignments (cells with assignable clones)
- Reconstructed clonal populations based on mtDNA variant profiles
- Concordance metrics vs. full-depth clone labels (for downsampling analyses)

## How to apply

First pile up mitochondrial reads from the analysis-ready BAM with cellsnp-lite to obtain the SNP-by-cell AD/DP matrices, then run MQuad (v0.1.6) to select high-quality informative variants via its binomial mixture model with a ΔBIC cutoff determined by the Kneedle knee point (removing error-prone rCRS regions 302–315, 513–525, 3105–3109). Feed the MQuad-selected SNPs into vireoSNP (v0.5.3), which uses variational inference to reconstruct clonal populations and assign every cell to a clone. On deeply sequenced or shallow datasets, check that per-cell mitochondrial depth supports reliable assignment — downsampling of MKN-45 scDNA-seq showed linear drops in assignable cells, informative variants, and concordance, supporting a minimum of ~3,000 mitochondrial reads per cell. Judge success by the number of cells with assignable clones, the number of informative mtDNA variants used, and concordance of assignments against full-depth labels.

## Related tools

- **vireoSNP** (Variational inference clonal reconstruction using the MQuad-selected mtDNA SNPs to assign cells to clones) — https://github.com/single-cell-genetics/vireo
- **MQuad** (Upstream selection of high-quality informative mtDNA variants via binomial mixture model with ΔBIC Kneedle cutoff) — https://github.com/single-cell-genetics/MQuad
- **cellsnp-lite** (Pile-up of raw mtDNA reads from BAM to generate the SNP-by-cell AD/DP matrices consumed by the pipeline) — https://github.com/single-cell-genetics/cellsnp-lite

## Evaluation signals

- Number of cells with assignable clones matches expected counts at the given sequencing depth (no mass unassigned clones).
- Number of informative mtDNA variants detected is consistent with depth; a linear drop with downsampling follows the reported MKN-45 trend.
- Clonal assignment concordance relative to full-depth clone labels is high and degrades linearly with decreasing reads per cell.
- Assignment is stable at/above the ~3,000 mitochondrial-reads-per-cell minimum and degrades below it.
- Error-prone rCRS regions 302–315, 513–525, 3105–3109 were excluded before reconstruction, so no clones are driven by those false-positive variants.

## Limitations

- Uninformative, noisy mtDNA SNVs and sequencing errors confound clonal inference if not filtered by MQuad's ΔBIC/Kneedle selection.
- Low sequencing depth (mitochondrial reads per cell) linearly reduces assignable cells, informative variants, and concordance; ~3,000 mt reads/cell is the recommended minimum for scDNA-seq.
- cellSNP mode 2 (pysam pileup) filters duplicates/orphan reads by default and can cause unexpected coverage loss; cellsnp-lite or large --maxFLAG is preferred.

## Evidence

- [discussion] vireoSNP (v0.5.3) uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad: "vireoSNP (v0.5.3)24 uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad"
- [other] Downsampling MKN-45 scDNA-seq showed a linear drop in assignable cells, informative mtDNA variants detected, and clonal assignment concordance with decreasing reads per cell, leading to a recommended minimum of 3,000 mitochondrial reads per cell for MQuad on scDNA-seq.: [evidence span withheld — anchor status: not_found]
- [discussion] We also explicitly removed frequent false positive variants occurring in error prone regions as described by Xu et al.22, including rCRS 302–315, 513–525, 3105–3109: "We also explicitly removed frequent false positive variants occurring in error prone regions as described by Xu et al.22, including rCRS 302–315, 513–525, 3105–3109"
- [readme] use vireoSNP to assign cells to clones based on mtDNA variant profile: "use `vireoSNP <https://github.com/single-cell-genetics/vireo>`_ to assign cells to clones based on mtDNA variant profile"
- [discussion] raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23"
