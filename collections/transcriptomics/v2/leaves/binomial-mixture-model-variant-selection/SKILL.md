---
name: binomial-mixture-model-variant-selection
description: Use when single-cell mtDNA AD/DP matrices (from cellsnp-lite or cellSNP)
  are available and you must distinguish clone-discriminative heteroplasmic variants
  from noisy, non-inherited SNVs before clonal reconstruction with vireoSNP.
when_to_use_negative:
- The input is nuclear genomic SNV data — nuclear SNV callers such as Monovar and
  Conbase assume a diploid context, which is violated in the mitochondrial genome,
  and this model is tuned for mtDNA heteroplasmy.
- The goal is donor deconvolution from common SNPs (use cellSNP/vireo in demultiplexing
  mode rather than clonal variant selection on somatic mtDNA variants).
- Data are mtscATAC-seq where mtDNA-specific callers like EMBLEM or mgatk were the
  primary design target — MQuad was evaluated mainly on scRNA-seq, scDNA-seq and Smart-seq2
  protocols.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MQuad
  role: Fits per-variant one- vs two-component binomial mixture models and selects
    informative SNPs via the ΔBIC/Kneedle cutoff (v0.1.6)
  repo: https://github.com/single-cell-genetics/MQuad
- name: cellsnp-lite
  role: Piles up mitochondrial reads from BAM files to generate the SNP-by-cell AD/DP
    matrices that MQuad consumes (v1.2.1)
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: kneed
  role: Determines the ΔBIC selection cutoff from the knee point of the cumulative
    ΔBIC distribution (v0.7.0)
- name: vireoSNP
  role: Downstream variational-inference assignment of cells to mitochondrial clones
    using the MQuad-selected SNPs (v0.5.3)
  repo: https://github.com/single-cell-genetics/vireo
provenance:
  source_task_ids:
  - task_007
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:fbf18399bc1e489c3757b279b9deb1c62acdf25c5aafda7ba15004baccaf89cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/MQuad
  tools:
  - Cardelino
  - CellRanger
  - MQuad
  - Picard
  - STAR
  - cellsnp-lite
  - kneed
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# binomial-mixture-model-variant-selection

## Summary

Selects clonally informative mtDNA variants from single-cell pileup matrices by fitting one- versus two-component binomial mixture models per variant with MQuad and ranking them by ΔBIC = BIC(M0) − BIC(M1), with the selection cutoff taken at the Kneedle knee point. This model-selection approach is more robust than raw allele-frequency filters and reduces false positives, especially in deeply sequenced Smart-seq2 data.

## When to use

Use when single-cell mtDNA AD/DP matrices (from cellsnp-lite or cellSNP) are available and you must distinguish clone-discriminative heteroplasmic variants from noisy, non-inherited SNVs before clonal reconstruction with vireoSNP.

## When NOT to use

- The input is nuclear genomic SNV data — nuclear SNV callers such as Monovar and Conbase assume a diploid context, which is violated in the mitochondrial genome, and this model is tuned for mtDNA heteroplasmy.
- The goal is donor deconvolution from common SNPs (use cellSNP/vireo in demultiplexing mode rather than clonal variant selection on somatic mtDNA variants).
- Data are mtscATAC-seq where mtDNA-specific callers like EMBLEM or mgatk were the primary design target — MQuad was evaluated mainly on scRNA-seq, scDNA-seq and Smart-seq2 protocols.

## Inputs

- SNP-by-cell matrices of alternate-allele counts (AD) and depths (DP) from cellsnp-lite/cellSNP pileup over chrMT
- Analysis-ready single-cell BAM files (10x Genomics scRNA-seq/scDNA-seq aligned to hg38, or Smart-seq2 aligned to hg19 with STAR and Picard MarkDuplicates)
- Cell barcode list for barcoded (10x) samples
- List of error-prone rCRS regions to exclude (302–315, 513–525, 3105–3109)

## Outputs

- List of clone-discriminative (informative) mtDNA SNVs with per-variant ΔBIC values
- Kneedle knee-point ΔBIC cutoff used for SNP selection
- AD/DP matrices restricted to selected SNPs, formatted for vireoSNP clone assignment
- Per-clone allele-frequency table of selected variants

## How to apply

First generate SNP-by-cell alternate-allele depth (AD) and total depth (DP) matrices by piling up mitochondrial reads from BAM files with cellsnp-lite (v1.2.1), targeting chrMT. For each variant, run MQuad (v0.1.6) fitting a one-component binomial model (null) versus a two-component model, and compute ΔBIC = BIC(M0) − BIC(M1); select informative SNPs using the ΔBIC cutoff at the Kneedle (kneed v0.7.0) knee point of the cumulative ΔBIC distribution. Explicitly remove error-prone rCRS regions 302–315, 513–525, and 3105–3109, which are known false-positive hotspots (Xu et al.). Feed the selected SNPs to vireoSNP (v0.5.3) variational inference to assign cells to mtDNA clones, keeping cells with assignment probability >0.8. Expected yields in the source tasks: ~24 informative SNVs for joxm fibroblast and MKN-45 scDNA-seq, and 2 informative variants for TNBC1 scRNA-seq.

## Related tools

- **MQuad** (Fits per-variant one- vs two-component binomial mixture models and selects informative SNPs via the ΔBIC/Kneedle cutoff (v0.1.6)) — https://github.com/single-cell-genetics/MQuad
- **cellsnp-lite** (Piles up mitochondrial reads from BAM files to generate the SNP-by-cell AD/DP matrices that MQuad consumes (v1.2.1)) — https://github.com/single-cell-genetics/cellsnp-lite
- **kneed** (Determines the ΔBIC selection cutoff from the knee point of the cumulative ΔBIC distribution (v0.7.0))
- **vireoSNP** (Downstream variational-inference assignment of cells to mitochondrial clones using the MQuad-selected SNPs (v0.5.3)) — https://github.com/single-cell-genetics/vireo

## Examples

```
cellSNP -s $BAM -b $BARCODE -O $OUT_DIR -p 22 --minMAF 0.1 --minCOUNT 100 --chrom chrMT
```

## Evaluation signals

- Recovered informative-variant counts match published runs: ~24 SNVs for the joxm fibroblast Smart-seq2 and MKN-45 scDNA-seq datasets, and 2 informative variants for TNBC1
- vireoSNP assigns ≥95% of cells at probability >0.8 (e.g., 1097/1146 cells, 95.8%, in TNBC1)
- Per-clone variant table confirms signature alleles: MT0 of MKN-45 carries 2393C>T and 8368G>A largely absent in other clones; joxm shows clone-specific 11196G>A and 2619A>T
- No selected variant falls in error-prone rCRS regions 302–315, 513–525, or 3105–3109
- Downstream cross-checks hold, e.g., TNBC1 Fisher's exact test p ≈ 6e-5 for mtDNA clone 1 enrichment in tumor cells

## Limitations

- cellSNP mode 2 uses pysam pileup() which filters duplicates and orphan reads by default, potentially causing unexpected coverage reduction; cellsnp-lite is recommended with tunable read filtering.
- PCR duplicates are often not well flagged in CellRanger BAMs, so aggressive duplicate filtering can lose a substantial fraction of SNPs.
- Uninformative and noisy mtDNA SNVs, together with sequencing errors, confound clonal inference if not filtered by the ΔBIC model-selection step.

## Evidence

- [discussion] MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
- [discussion] ΔBIC cutoff via Kneedle: "a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm36"
- [discussion] Model selection more robust than raw allele frequencies: "it adopts a model selection approach in evaluating the informativeness of each variant, which is more robust than considering the raw allele frequencies alone"
- [discussion] rCRS error-prone region filtering: "We also explicitly removed frequent false positive variants occurring in error prone regions as described by Xu et al.22, including rCRS 302–315, 513–525, 3105–3109"
- [readme] MQuad purpose from README: "MQuad is a tool that detects mitochondrial mutations that are informative for clonal substructure inference. It uses a binomial mixture model to assess the heteroplasmy of mtDNA variants"
- [readme] vireoSNP downstream from README: "use vireoSNP to assign cells to clones based on mtDNA variant profile"
