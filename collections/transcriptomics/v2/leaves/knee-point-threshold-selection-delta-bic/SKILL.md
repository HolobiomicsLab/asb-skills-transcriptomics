---
name: knee-point-threshold-selection-delta-bic
description: Use when calling clonally informative mtDNA variants from single-cell
  genotyping matrices (e.g., cellsnp-lite AD/DP) and you need an objective, unsupervised
  ΔBIC cutoff instead of raw allele frequency thresholds.
when_to_use_negative:
- Data already processed by mgatk or another mtDNA variant caller with its own quality
  cutoffs (e.g., log10(VMR) ≥ −2 and strand correlation ≥ 0.65) — do not double-filter
  with the ΔBIC knee.
- Nuclear genome SNV calling, where the diploid context assumed by tools like Monovar
  or Conbase applies and mitochondrial heteroplasmy modeling is inappropriate.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: MQuad
  role: Fits the 1-component vs 2-component binomial mixture model per mtDNA variant
    and computes ΔBIC for informativeness scoring
  repo: https://github.com/single-cell-genetics/MQuad
- name: cellsnp-lite
  role: Produces the SNP-by-cell AD/DP pileup matrices that serve as MQuad's input
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: vireoSNP
  role: Consumes the Kneedle-selected ΔBIC-filtered variants to reconstruct clonal
    populations via variational inference
  repo: https://github.com/single-cell-genetics/vireo
- name: kneed
  role: Kneedle algorithm implementation used to identify the knee point of the ΔBIC
    CDF as the variant-selection cutoff
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:f9aa9a1192aba0cf1db3f1e7d7faea8805a2776c36c6b792fc3e2c0a4ed92391
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/MQuad
  tools:
  - MQuad
  - Picard
  - STAR
  - cellsnp-lite
  - kneed
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# knee-point-threshold-selection-delta-bic

## Summary

Selects clonally informative mtDNA variants in MQuad by computing ΔBIC = BIC(M0) − BIC(M1) from a 1-component vs 2-component binomial mixture fit per variant, then setting the ΔBIC cutoff at the knee of the cumulative distribution function via the Kneedle algorithm.

## When to use

Use when calling clonally informative mtDNA variants from single-cell genotyping matrices (e.g., cellsnp-lite AD/DP) and you need an objective, unsupervised ΔBIC cutoff instead of raw allele frequency thresholds.

## When NOT to use

- Data already processed by mgatk or another mtDNA variant caller with its own quality cutoffs (e.g., log10(VMR) ≥ −2 and strand correlation ≥ 0.65) — do not double-filter with the ΔBIC knee.
- Nuclear genome SNV calling, where the diploid context assumed by tools like Monovar or Conbase applies and mitochondrial heteroplasmy modeling is inappropriate.

## Inputs

- SNP-by-cell AD (alternate depth) and DP (depth) count matrices from cellsnp-lite pileup of mitochondrial BAM files
- Per-variant allele count/heteroplasmy profiles across single cells

## Outputs

- Per-variant ΔBIC values (BIC(M0) − BIC(M1)) from binomial mixture model fits
- Cumulative distribution function (CDF) plot of ΔBIC values
- Kneedle-derived ΔBIC cutoff and final list of selected informative mtDNA variants for vireoSNP clone assignment

## How to apply

After cellsnp-lite piles up mtDNA reads into SNP-by-cell AD/DP matrices, run MQuad's binomial mixture model fitting each variant under a 1-component (M0) and 2-component (M1) model and compute ΔBIC = BIC(M0) − BIC(M1), which evaluates informativeness more robustly than raw allele frequencies alone. Plot the cumulative distribution function of ΔBIC across variants; the CDF shows a sharp knee where informative variants separate from noise. Set the selection cutoff at this knee point using the Kneedle algorithm (kneed package). Optionally exclude frequent false positives in error-prone rCRS regions 302–315, 513–525, and 3105–3109, then pass the retained variants to vireoSNP for clonal assignment.

## Related tools

- **MQuad** (Fits the 1-component vs 2-component binomial mixture model per mtDNA variant and computes ΔBIC for informativeness scoring) — https://github.com/single-cell-genetics/MQuad
- **cellsnp-lite** (Produces the SNP-by-cell AD/DP pileup matrices that serve as MQuad's input) — https://github.com/single-cell-genetics/cellsnp-lite
- **vireoSNP** (Consumes the Kneedle-selected ΔBIC-filtered variants to reconstruct clonal populations via variational inference) — https://github.com/single-cell-genetics/vireo
- **kneed** (Kneedle algorithm implementation used to identify the knee point of the ΔBIC CDF as the variant-selection cutoff)

## Examples

```
from kneed import KneeLocator; kn = KneeLocator(x=dbic_sorted, y=cdf_values, curve='concave', direction='increasing'); cutoff = kn.knee
```

## Evaluation signals

- On the Kim ccRCC Smart-seq2 dataset, the Kneedle ΔBIC cutoff should retain approximately 146 informative mtDNA variants yielding ~93% concordance between vireoSNP mitochondrial clones and source labels (PDX pRCC, PDX mRCC, Pt mRCC).
- The ΔBIC CDF must show a sharp knee; the Kneedle algorithm should converge on a knee point rather than returning an edge/extreme of the distribution.
- Variants in rCRS regions 302–315, 513–525, and 3105–3109 must be absent from the final selected set.
- Selected variants should drive a 2-component fit (ΔBIC > 0), separating heteroplasmic cells from background noise, consistent with MQuad's false-positive reduction on deeply sequenced Smart-seq2 data.

## Limitations

- Model selection via ΔBIC is calibrated for detecting bimodal (clonal vs wild-type) heteroplasmy; variants with more complex mixture structure may be misclassified.
- Uninformative and noisy mtDNA SNVs plus sequencing errors confound clonal inference, so the knee cutoff can still admit false positives if the CDF knee is ambiguous.
- mgatk with its suggested cutoffs (log10(VMR) ≥ −2, strand correlation ≥ 0.65) detected 312 mtSNVs but only 64% concordance on the same ccRCC data, showing caller choice strongly affects downstream clone assignments.

## Evidence

- [discussion] MQuad uses a model selection approach (ΔBIC) to evaluate variant informativeness, more robust than raw allele frequencies alone: "it adopts a model selection approach in evaluating the informativeness of each variant, which is more robust than considering the raw allele frequencies alone"
- [discussion] Kneedle knee point as ΔBIC cutoff: "a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm36"
- [discussion] Binomial mixture model selects informative variants: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
- [discussion] cellsnp-lite pileup generates SNP-by-cell matrix input: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23. This generates a SNP-by-cell matrix"
- [discussion] Explicit removal of error-prone rCRS region false positives: "We also explicitly removed frequent false positive variants occurring in error prone regions as described by Xu et al.22, including rCRS 302–315, 513–525, 3105–3109"
- [readme] MQuad differentiates informative variants from noisy background: "use MQuad to differentiate informative mtDNA variants from noisy backbground"
