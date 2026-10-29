---
name: clonal-assignment-concordance-evaluation
description: Use when quantifying how clonal assignments from mtDNA variants hold
  up under reduced sequencing depth — e.g., comparing vireoSNP clone labels at downsampled
  mt-read depths against full-depth labels on 10x scDNA/scRNA BAMs.
when_to_use_negative:
- The analysis goal is variant calling or clone reconstruction at a single fixed depth
  rather than depth-sensitivity assessment — run the cellSNP-MQuad-VireoSNP pipeline
  once instead.
- Lineage inference is based on nuclear SNVs or CNVs (e.g., cardelino-style clone
  deconvolution); concordance here is defined only for mtDNA-variant-based vireoSNP
  clone labels.
- Input is mtscATAC-seq or Smart-seq2 data where depth behavior differs and the 3,000
  mt-reads/cell recommendation was derived specifically for 10x scDNA-seq.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0199
tools:
- name: cellsnp-lite
  role: Piles up mitochondrial reads from each downsampled BAM into the SNP-by-cell
    AD/DP matrices consumed by MQuad
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MQuad
  role: Selects clonally informative mtDNA variants at each depth via binomial mixture
    model with the ΔBIC/Kneedle cutoff
  repo: https://github.com/single-cell-genetics/MQuad
- name: vireoSNP
  role: Reconstructs clonal populations via variational inference from the MQuad-selected
    SNPs; labels are compared to full-depth assignments for concordance
  repo: https://github.com/single-cell-genetics/vireo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:9e083dc36da32e02b64560a140913b9ee55056388dfe1153b13772004d4ecd2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/cellsnp-lite
  tools:
  - MQuad
  - cellsnp-lite
  - vireoSNP
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# clonal-assignment-concordance-evaluation

## Summary

Evaluates the robustness of the cellsnp-lite→MQuad→vireoSNP mtDNA clonality pipeline by measuring clonal assignment concordance, assignable cell counts, and informative variant counts across decreasing mitochondrial reads-per-cell depths relative to full-depth clone labels.

## When to use

Use when quantifying how clonal assignments from mtDNA variants hold up under reduced sequencing depth — e.g., comparing vireoSNP clone labels at downsampled mt-read depths against full-depth labels on 10x scDNA/scRNA BAMs.

## When NOT to use

- The analysis goal is variant calling or clone reconstruction at a single fixed depth rather than depth-sensitivity assessment — run the cellSNP-MQuad-VireoSNP pipeline once instead.
- Lineage inference is based on nuclear SNVs or CNVs (e.g., cardelino-style clone deconvolution); concordance here is defined only for mtDNA-variant-based vireoSNP clone labels.
- Input is mtscATAC-seq or Smart-seq2 data where depth behavior differs and the 3,000 mt-reads/cell recommendation was derived specifically for 10x scDNA-seq.

## Inputs

- Analysis-ready 10x single-cell BAM (e.g., MKN-45 scDNA-seq aligned to hg38) with cell barcodes
- Cell barcode list (TSV/whitelist for cellsnp-lite -b)
- Full-depth vireoSNP clone labels used as the concordance reference
- Grid of target reads-per-cell depths for downsampling

## Outputs

- Table keyed by reads-per-cell with columns: assignable cells, informative mtDNA variants detected, clonal assignment concordance vs. full depth
- Per-depth MQuad informative-variant lists and vireoSNP clone assignment files
- Recommended minimum mitochondrial reads-per-cell threshold (e.g., 3,000)

## How to apply

Downsample the analysis-ready BAM (e.g., MKN-45 10x scDNA-seq, hg38) to a grid of decreasing mitochondrial reads-per-cell depths. At each depth, pile up mtDNA reads with cellsnp-lite (v1.2.1) to produce the SNP-by-cell AD/DP matrices, run MQuad (v0.1.6) selecting informative variants with the binomial mixture model using the ΔBIC cutoff determined by the Kneedle algorithm (kneed v0.7.0), and reconstruct clones with vireoSNP (v0.5.3) variational inference. Explicitly remove error-prone rCRS regions 302–315, 513–525, and 3105–3109 before selection. At each depth record three metrics keyed by reads per cell: number of cells with assignable clones, number of informative mtDNA variants detected, and clonal assignment concordance relative to the full-depth clone labels. Validation is confirming a linear drop in all three metrics with decreasing depth and identifying the minimum adequate depth (reported as ~3,000 mitochondrial reads per cell for MQuad on scDNA-seq).

## Related tools

- **cellsnp-lite** (Piles up mitochondrial reads from each downsampled BAM into the SNP-by-cell AD/DP matrices consumed by MQuad) — https://github.com/single-cell-genetics/cellsnp-lite
- **MQuad** (Selects clonally informative mtDNA variants at each depth via binomial mixture model with the ΔBIC/Kneedle cutoff) — https://github.com/single-cell-genetics/MQuad
- **vireoSNP** (Reconstructs clonal populations via variational inference from the MQuad-selected SNPs; labels are compared to full-depth assignments for concordance) — https://github.com/single-cell-genetics/vireo

## Examples

```
cellSNP -s $BAM -b $BARCODE -O $OUT_DIR -p 22 --minMAF 0.1 --minCOUNT 100 --chrom chrMT
```

## Evaluation signals

- Assignable cell count, informative variant count, and concordance all decrease (approximately linearly) as reads per cell decrease
- Concordance approaches 1.0 at full depth, since full-depth labels are the reference
- Result is assembled as a single table keyed by reads per cell with all three metrics populated at every depth
- The knee/knee-point depth supporting the minimum-depth recommendation falls near 3,000 mitochondrial reads per cell for scDNA-seq
- Per-depth AD/DP matrices contain the expected number of cell barcodes (matching the downsampled cell set) and mtDNA positions with error-prone rCRS regions 302–315, 513–525, 3105–3109 excluded

## Limitations

- Concordance is defined relative to full-depth clone labels, so it is bounded by the accuracy of the full-depth assignment itself.
- cellSNP/cellsnp-lite duplicate handling interacts with depth estimates: PCR duplicate filtering is off by default (--maxFLAG) because CellRanger flags duplicates poorly, which can inflate apparent coverage.
- MQuad v0.1.6 / cellsnp-lite v1.2.1 / vireoSNP v0.5.3 were tested on Python 3.8.8 (Windows 10, CentOS 7); behavior on other stacks is not asserted.
- The 3,000 mt-reads-per-cell minimum is grounded in the MKN-45 scDNA-seq downsampling and may not transfer to scRNA-seq or mtscATAC-seq protocols.

## Evidence

- [discussion] raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23. This generates a SNP-by-cell matrix: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23. This generates a SNP-by-cell matrix"
- [discussion] vireoSNP variational inference reconstructs clones from MQuad-selected SNPs: "vireoSNP (v0.5.3)24 uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad"
- [discussion] ΔBIC cutoff for SNP selection determined via Kneedle: "a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm36"
- [discussion] error-prone rCRS regions removed: "We also explicitly removed frequent false positive variants occurring in error prone regions as described by Xu et al.22, including rCRS 302–315, 513–525, 3105–3109"
- [readme] README pipeline order: cellSNP → MQuad → vireoSNP clone assignment: "use `vireoSNP <https://github.com/single-cell-genetics/vireo>`_ to assign cells to clones based on mtDNA variant profile"
- [readme] README duplicate-filtering caveat affecting coverage: [evidence span withheld — anchor status: external_doc]
