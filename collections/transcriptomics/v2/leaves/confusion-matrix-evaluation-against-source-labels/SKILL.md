---
name: confusion-matrix-evaluation-against-source-labels
description: Use when predicted single-cell clone labels (e.g., from vireoSNP on mtDNA
  variants) must be validated against ground-truth source/population labels, and a
  concordance percentage or confusion matrix is the required benchmark output.
when_to_use_negative:
- No independent ground-truth labels (e.g., source population or donor identity) exist
  for the cells — concordance cannot be computed and only unsupervised clonal structure
  can be assessed
- Predictions come from nuclear SNV/CNV-based methods with different clone definitions
  that do not correspond one-to-one to mitochondrial clones
- The goal is caller performance on simulated data without known clone-source structure;
  use the simulation-specific selection criteria (average depth >50, VAF >1% in >5%
  of cells) instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: vireoSNP
  role: Produces the per-cell predicted clone assignments (variational inference)
    that are cross-tabulated against source labels
  repo: https://github.com/single-cell-genetics/vireo
- name: MQuad
  role: Selects the high-quality informative mtDNA variants whose resulting clones
    are evaluated via concordance
  repo: https://github.com/single-cell-genetics/MQuad
- name: mgatk
  role: Comparison variant caller whose clones are evaluated with the same confusion-matrix/concordance
    protocol
- name: cellsnp-lite
  role: Piles up mtDNA alleles per cell (SNP-by-cell AD/DP matrices) feeding the clones
    under evaluation
  repo: https://github.com/single-cell-genetics/cellsnp-lite
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:0b14746a9577f0f3c830418b324da1b27f68ed5d9450e4280eb8ad62e90d7539
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/vireo
  tools:
  - MQuad
  - Picard
  - STAR
  - cellsnp-lite
  - mgatk
  - vireoSNP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# confusion-matrix-evaluation-against-source-labels

## Summary

Evaluates single-cell clonal assignment accuracy by cross-tabulating vireoSNP-predicted mitochondrial clones against known source populations (e.g., PDX pRCC, PDX mRCC, Pt mRCC) and reporting assignment concordance, as done to benchmark MQuad (93% concordance, ~146 variants) versus mgatk (64% concordance, 312 mtSNVs) on the Kim ccRCC Smart-seq2 dataset.

## When to use

Use when predicted single-cell clone labels (e.g., from vireoSNP on mtDNA variants) must be validated against ground-truth source/population labels, and a concordance percentage or confusion matrix is the required benchmark output.

## When NOT to use

- No independent ground-truth labels (e.g., source population or donor identity) exist for the cells — concordance cannot be computed and only unsupervised clonal structure can be assessed
- Predictions come from nuclear SNV/CNV-based methods with different clone definitions that do not correspond one-to-one to mitochondrial clones
- The goal is caller performance on simulated data without known clone-source structure; use the simulation-specific selection criteria (average depth >50, VAF >1% in >5% of cells) instead

## Inputs

- Per-cell predicted clone assignments from vireoSNP (clone ID per cell barcode)
- Per-cell ground-truth source labels (e.g., PDX pRCC, PDX mRCC, Pt mRCC for the Kim ccRCC Smart-seq2 dataset)
- Per-tool variant counts (MQuad-selected informative mtDNA variants; mgatk-called mtSNVs)

## Outputs

- Confusion matrix of predicted mitochondrial clones versus source labels, per variant caller
- Assignment concordance percentages per tool (target: MQuad ≈ 93% with ≈146 variants; mgatk 64% with 312 mtSNVs)
- Summary table combining variant counts and concordance for MQuad vs mgatk

## How to apply

After assigning cells to mitochondrial clones with vireoSNP variational inference using MQuad-selected informative variants (or mgatk variants called at log10(VMR) ≥ −2 and strand correlation ≥ 0.65), collect the per-cell predicted clone IDs and the per-cell source labels. Build a confusion matrix with rows as predicted mitochondrial clones and columns as the source labels (for the Kim ccRCC dataset: PDX pRCC, PDX mRCC, Pt mRCC). Compute assignment concordance as the fraction of cells whose predicted clone maps to the dominant source label, and assemble variant counts alongside concordance into a summary table per tool. Repeat identically for the comparison caller (mgatk) so concordance and variant counts are directly comparable; the rationale is that concordance to independently known populations validates whether the mtDNA variants are clonally informative rather than noise.

## Related tools

- **vireoSNP** (Produces the per-cell predicted clone assignments (variational inference) that are cross-tabulated against source labels) — https://github.com/single-cell-genetics/vireo
- **MQuad** (Selects the high-quality informative mtDNA variants whose resulting clones are evaluated via concordance) — https://github.com/single-cell-genetics/MQuad
- **mgatk** (Comparison variant caller whose clones are evaluated with the same confusion-matrix/concordance protocol)
- **cellsnp-lite** (Piles up mtDNA alleles per cell (SNP-by-cell AD/DP matrices) feeding the clones under evaluation) — https://github.com/single-cell-genetics/cellsnp-lite

## Examples

```
cellSNP -S $BAM_list_file -I sample_list_file -o $OUT_FILE -R $REGION_VCF -p 20 --UMItag None
```

## Evaluation signals

- Concordance for the MQuad+cellsnp-lite+vireoSNP route reproduces the reported ≈93% on the Kim ccRCC dataset with ≈146 informative mtDNA variants
- Concordance for the mgatk route reproduces the reported 64% with 312 mtSNVs (log10(VMR) ≥ −2, strand correlation ≥ 0.65)
- Confusion matrix dimensions and diagonals are consistent with three source populations (PDX pRCC, PDX mRCC, Pt mRCC), and every cell appears exactly once in both label vectors
- Concordance is computed from the dominant clone-to-source mapping and falls in a plausible range (0–100%) with no unassigned cells silently dropped

## Limitations

- Concordance to source labels reflects consistency with nuclear SNV/CNV-defined populations, not proof of true lineage; distinct CNVs and nuclear SNVs among populations drive the high score
- Results are protocol- and dataset-dependent: Smart-seq2 deep coverage favors MQuad's model selection, and mtDNA-specific methods like mgatk were designed primarily for scATAC-seq
- Frequent false positive variants in error-prone rCRS regions 302–315, 513–525, 3105–3109 must be removed, or concordance can be degraded by noisy variants

## Evidence

- [discussion] vireoSNP variational inference reconstructs clones: "vireoSNP (v0.5.3)24 uses variational inference to reconstruct clonal populations based on the selected SNPs from MQuad"
- [discussion] MQuad selects informative variants with binomial mixture model: "MQuad (v0.1.6) takes this output and selects high quality informative variants with a binomial mixture model"
- [discussion] cellsnp-lite piles up reads into SNP-by-cell matrix: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23. This generates a SNP-by-cell matrix"
- [discussion] mgatk comparison cutoffs: "we used the suggested cutoffs to call variants: log10(VMR) ≥ −2 and strand correlation coefficient ≥0.65"
- [readme] cellsnp-lite output feeds MQuad then vireoSNP in recommended pipeline: "use MQuad to differentiate informative mtDNA variants from noisy backbground"
