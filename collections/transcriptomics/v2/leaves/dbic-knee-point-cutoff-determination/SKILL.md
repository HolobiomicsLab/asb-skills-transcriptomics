---
name: dbic-knee-point-cutoff-determination
description: Use when selecting informative mtDNA variants from cellsnp-lite AD/DP
  matrices with MQuad and you need an objective, data-driven ΔBIC cutoff via the Kneedle
  knee point rather than an arbitrary VAF threshold.
when_to_use_negative:
- Your goal is donor deconvolution from common germline SNPs (cellSNP/vireo donor
  demultiplexing) rather than discovering somatic, clone-discriminative mtDNA variants.
- Data were produced primarily by scATAC-seq, where mtDNA-specific callers such as
  EMBLEM or mgatk (with log10(VMR) ≥ −2 and strand correlation ≥0.65 cutoffs) are
  the designed tools.
- You only need per-cell heteroplasmy estimates and already have a fixed, externally
  validated ΔBIC or VAF threshold; re-fitting the knee point would add no information.
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3308
tools:
- name: MQuad
  role: Fits the binomial mixture model per mtDNA variant and computes ΔBIC used for
    informativeness selection (v0.1.6)
  repo: github.com/single-cell-genetics/MQuad
- name: kneed
  role: Kneedle algorithm implementation (v0.7.0) used to locate the knee point that
    defines the ΔBIC cutoff
- name: cellsnp-lite
  role: Upstream pileup of mitochondrial reads from BAM into the SNP-by-cell AD/DP
    matrices that MQuad consumes (v1.2.1)
  repo: github.com/single-cell-genetics/cellsnp-lite
- name: vireoSNP
  role: Downstream consumer of the knee-point-selected SNPs for variational-inference
    clone assignment (v0.5.3)
  repo: github.com/single-cell-genetics/vireo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:8c60ed7235bddfc37d6e3f91f1a36dcae51c0cc56068b7704cad90fb46c24471
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MQuad
  - cellsnp-lite
  - kneed
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ΔBIC knee-point cutoff determination

## Summary

Determines the ΔBIC threshold for selecting clonally informative mtDNA variants in MQuad by fitting the binomial mixture model per variant and applying the Kneedle algorithm (kneed v0.7.0) at the knee point of the ΔBIC distribution, instead of relying on raw allele frequencies.

## When to use

Use when selecting informative mtDNA variants from cellsnp-lite AD/DP matrices with MQuad and you need an objective, data-driven ΔBIC cutoff via the Kneedle knee point rather than an arbitrary VAF threshold.

## When NOT to use

- Your goal is donor deconvolution from common germline SNPs (cellSNP/vireo donor demultiplexing) rather than discovering somatic, clone-discriminative mtDNA variants.
- Data were produced primarily by scATAC-seq, where mtDNA-specific callers such as EMBLEM or mgatk (with log10(VMR) ≥ −2 and strand correlation ≥0.65 cutoffs) are the designed tools.
- You only need per-cell heteroplasmy estimates and already have a fixed, externally validated ΔBIC or VAF threshold; re-fitting the knee point would add no information.

## Inputs

- SNP-by-cell AD (alternate-depth) matrix from cellsnp-lite
- SNP-by-cell DP (total-depth) matrix from cellsnp-lite
- cellsnp-lite output directory from a pileup over the mitochondrial genome of a single-cell BAM (e.g., 10x Genomics scRNA-seq aligned to hg38)
- Cell barcode list (TSV) used for the cellsnp-lite pileup

## Outputs

- List of informative mtDNA variants passing the ΔBIC knee-point cutoff
- Per-variant ΔBIC values from MQuad binomial mixture model fits
- Kneedle knee-point ΔBIC cutoff value
- Filtered variant set formatted for downstream vireoSNP clonal assignment

## How to apply

After cellsnp-lite (v1.2.1) piles up mitochondrial reads from the analysis-ready BAM (e.g., TNBC1 10x scRNA-seq aligned to hg38) into SNP-by-cell AD/DP matrices, run MQuad's (v0.1.6) binomial mixture model per mtDNA variant, which compares a two-component mixture against a null noise model and yields a ΔBIC per variant. Compute the ΔBIC cutoff at the knee point identified by the Kneedle algorithm (kneed v0.7.0) rather than choosing a fixed threshold, because the model selection approach is more robust than raw allele frequencies alone and reduces false positives, especially in deeply sequenced data. Before fitting, explicitly remove recurrent false positives in error-prone rCRS regions 302–315, 513–525, and 3105–3109. Pass only variants above the knee-point ΔBIC cutoff to vireoSNP (v0.5.3) for clone assignment. In the TNBC1 benchmark this procedure retained exactly two informative variants, which then assigned 1097 of 1146 cells (95.8%) at probability >0.8.

## Related tools

- **MQuad** (Fits the binomial mixture model per mtDNA variant and computes ΔBIC used for informativeness selection (v0.1.6)) — github.com/single-cell-genetics/MQuad
- **kneed** (Kneedle algorithm implementation (v0.7.0) used to locate the knee point that defines the ΔBIC cutoff)
- **cellsnp-lite** (Upstream pileup of mitochondrial reads from BAM into the SNP-by-cell AD/DP matrices that MQuad consumes (v1.2.1)) — github.com/single-cell-genetics/cellsnp-lite
- **vireoSNP** (Downstream consumer of the knee-point-selected SNPs for variational-inference clone assignment (v0.5.3)) — github.com/single-cell-genetics/vireo

## Examples

```
from kneed import KneeLocator; kneedle = KneeLocator(delta_bic_ranks, sorted_delta_bics, curve='concave', direction='increasing'); dbic_cutoff = kneedle.knee
```

## Evaluation signals

- The Kneedle knee point yields a ΔBIC cutoff that retains exactly the expected number of informative variants (two informative variants detected in the TNBC1 sample).
- Downstream vireoSNP assigns ≥95% of cells with probability >0.8 (TNBC1: 1097 of 1146 cells, 95.8%).
- No selected variant falls in the known error-prone rCRS regions 302–315, 513–525, or 3105–3109.
- Mitochondrial clones built from the selected SNPs reproduce known biology: Fisher's exact test for mitochondrial clone 1 enrichment among tumor cells gives p ≈ 6e-5, and clone 1 overlaps almost exclusively with CNV subclone B.
- Cutoff choice is reproducible: re-running the mixture fits and KneeLocator on the same AD/DP matrices recovers the same knee ΔBIC value.

## Limitations

- The Kneedle knee point depends on the shape of the ΔBIC distribution, so datasets with very few or very weak informative variants may yield an unstable or ill-defined knee.
- The ΔBIC model-selection approach is framed as most beneficial for deeply sequenced data (e.g., Smart-seq2); on shallow 10x data the separation between informative and noisy variants may be less pronounced.
- Recurrent false positives in error-prone rCRS regions (302–315, 513–525, 3105–3109) must be removed explicitly, otherwise they can contaminate the variant set regardless of the ΔBIC cutoff.

## Evidence

- [discussion] ΔBIC cutoff determined with Kneedle knee point: "a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm36"
- [discussion] Model selection more robust than raw allele frequencies: "it adopts a model selection approach in evaluating the informativeness of each variant, which is more robust than considering the raw allele frequencies alone"
- [readme] MQuad binomial mixture model assesses heteroplasmy against background noise: "It uses a binomial mixture model to assess the heteroplasmy of mtDNA variants among background noise."
- [discussion] Error-prone rCRS regions removed explicitly: "We also explicitly removed frequent false positive variants occurring in error prone regions as described by Xu et al.22, including rCRS 302–315, 513–525, 3105–3109"
- [discussion] Pipeline context: MQuad takes cellsnp-lite output and selects informative variants: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
