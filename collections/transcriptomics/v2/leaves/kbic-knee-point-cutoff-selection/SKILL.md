---
name: kbic-knee-point-cutoff-selection
description: Use when filtering mtDNA variant candidates from MQuad binomial mixture
  output — i.e., when a ΔBIC distribution over SNPs must be converted into an informative-variant
  list via an objective, data-driven knee-point cutoff (kneed v0.7.0).
when_to_use_negative:
- The input is nuclear SNV calls or a diploid-context variant set — the ΔBIC mixture
  model assumes mitochondrial heteroplasmy, which nuclear SNV callers like Monovar
  and Conbase do not model
- Only a handful of candidate variants exist, so the ΔBIC distribution is too sparse
  to define a meaningful Kneedle knee point
- The goal is donor demultiplexing with known common germline SNPs (cellSNP+vireo
  mode), not somatic mtDNA clone discovery
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3173
tools:
- name: MQuad
  role: Fits one- vs two-component binomial mixture models per mtDNA variant and computes
    ΔBIC for informativeness scoring
  repo: https://github.com/single-cell-genetics/MQuad
- name: kneed
  role: Detects the knee point in the ΔBIC distribution to set the SNP-selection cutoff
    (v0.7.0)
- name: cellsnp-lite
  role: Piles up mitochondrial reads from BAM into the SNP-by-cell AD/DP matrices
    that feed the mixture models (v1.2.1)
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: vireoSNP
  role: 'Downstream consumer: reconstructs clonal populations from the knee-point-selected
    SNPs via variational inference (v0.5.3)'
  repo: https://github.com/single-cell-genetics/vireo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:9b98b2923acb150436a62bacff93e612f6f48f86b25cae3d5bb31cdd85684378
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/MQuad
  tools:
  - CellRanger
  - MQuad
  - cellsnp-lite
  - kneed
  - vireoSNP
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# kbic-knee-point-cutoff-selection

## Summary

Selects clonally informative mitochondrial SNPs by computing ΔBIC = BIC(M0) − BIC(M1) from MQuad's one- versus two-component binomial mixture model fits, then choosing an objective ΔBIC cutoff at the Kneedle knee point instead of a hand-picked threshold.

## When to use

Use when filtering mtDNA variant candidates from MQuad binomial mixture output — i.e., when a ΔBIC distribution over SNPs must be converted into an informative-variant list via an objective, data-driven knee-point cutoff (kneed v0.7.0).

## When NOT to use

- The input is nuclear SNV calls or a diploid-context variant set — the ΔBIC mixture model assumes mitochondrial heteroplasmy, which nuclear SNV callers like Monovar and Conbase do not model
- Only a handful of candidate variants exist, so the ΔBIC distribution is too sparse to define a meaningful Kneedle knee point
- The goal is donor demultiplexing with known common germline SNPs (cellSNP+vireo mode), not somatic mtDNA clone discovery

## Inputs

- SNP-by-cell alternate-allele depth (AD) and total depth (DP) matrices from cellsnp-lite pileup of a mtDNA BAM
- Per-variant ΔBIC = BIC(M0) − BIC(M1) values from MQuad one- vs two-component binomial mixture fits
- List of error-prone rCRS regions to exclude (302–315, 513–525, 3105–3109)

## Outputs

- A ΔBIC knee-point cutoff value determined by the Kneedle algorithm (kneed v0.7.0)
- A filtered list of clonally informative mtDNA SNPs with ΔBIC above the knee-point cutoff (e.g., 24 variants for MKN-45)

## How to apply

Run MQuad (v0.1.6) on the SNP-by-cell AD/DP matrices from cellsnp-lite (v1.2.1) to fit one- versus two-component binomial mixture models per mtDNA variant and compute ΔBIC = BIC(M0) − BIC(M1), which scores how much a two-cluster (informative) fit improves over a single background component. Rank variants by ΔBIC and pass the sorted values to the Kneedle algorithm (kneed v0.7.0) to detect the knee point; the ΔBIC at this knee is the selection cutoff, making the threshold dataset-specific rather than arbitrary. Before selection, explicitly drop frequent false positives in error-prone rCRS regions 302–315, 513–525, and 3105–3109. Rationale: model selection via ΔBIC is more robust than raw allele frequencies alone and reduces false positives, especially in deeply sequenced data. On the MKN-45 10x scDNA-seq dataset this procedure yielded 24 clone-discriminative variants that vireoSNP (v0.5.3) then used to resolve 5 mitochondrial clones.

## Related tools

- **MQuad** (Fits one- vs two-component binomial mixture models per mtDNA variant and computes ΔBIC for informativeness scoring) — https://github.com/single-cell-genetics/MQuad
- **kneed** (Detects the knee point in the ΔBIC distribution to set the SNP-selection cutoff (v0.7.0))
- **cellsnp-lite** (Piles up mitochondrial reads from BAM into the SNP-by-cell AD/DP matrices that feed the mixture models (v1.2.1)) — https://github.com/single-cell-genetics/cellsnp-lite
- **vireoSNP** (Downstream consumer: reconstructs clonal populations from the knee-point-selected SNPs via variational inference (v0.5.3)) — https://github.com/single-cell-genetics/vireo

## Examples

```
mquad run --cellSNP out_cellsnp/ --outDir mquad_out/ --nProc 8  # then: from kneed import KneeLocator; kl = KneeLocator(range(len(dbic_sorted)), dbic_sorted, curve='convex', direction='increasing'); cutoff = dbic_sorted[kl.knee]
```

## Evaluation signals

- The knee-point cutoff recovers a plausible number of informative variants (24 on the MKN-45 dataset)
- vireoSNP on the selected SNPs yields the expected number of mitochondrial clones (5 for MKN-45's 5220 cells)
- Flagship variants such as 2393C>T and 8368G>A marking clone MT0 are retained and largely absent in other clones
- None of the retained variants fall in the excluded rCRS error-prone regions 302–315, 513–525, or 3105–3109
- Downstream clone labels show high-degree overlap between CNV clone 1 and mitochondrial clone 0 in the confusion matrix

## Limitations

- Uninformative and noisy mtDNA SNVs plus sequencing errors confound inference if the cutoff is set by raw allele frequency instead of ΔBIC model selection
- Frequent false positives occur in specific error-prone rCRS regions (302–315, 513–525, 3105–3109) and must be explicitly removed
- The knee-point cutoff is dataset-specific; MQuad has been primarily benchmarked on scRNA-seq (Smart-seq2), 10x scDNA-seq (MKN-45, TNBC), and mtscATAC-seq data

## Evidence

- [discussion] ΔBIC cutoff via Kneedle: "a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm36"
- [discussion] Error-prone region exclusion: "We also explicitly removed frequent false positive variants occurring in error prone regions as described by Xu et al.22, including rCRS 302–315, 513–525, 3105–3109"
- [discussion] Model selection rationale: "it adopts a model selection approach in evaluating the informativeness of each variant, which is more robust than considering the raw allele frequencies alone"
- [discussion] Pipeline context: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
- [readme] README pipeline framing: "use MQuad to differentiate informative mtDNA variants from noisy backbground"
