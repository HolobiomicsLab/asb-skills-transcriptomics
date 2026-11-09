---
name: tf-binding-affinity-pwm-scanning
description: Use when you have fine-mapped GWAS/SCZ variants with genomic coordinates
  and need to predict which transcription factors' binding affinities are disrupted
  by the risk allele versus the reference allele, specifically to prioritize variants
  with large (≥20-fold) motif affinity changes.
when_to_use_negative:
- When the question is linking variants to genes they regulate (use GrID-Net peak-gene
  linking instead) rather than predicting within-locus TF binding changes.
- When variants are not fine-mapped or lack confident genomic coordinates in GRCh38/hg38,
  since allele-specific sequence extraction requires exact variant positions.
- When you only have gene-level expression or chromatin accessibility data and no
  variant-level genomic coordinates.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3125
- http://edamontology.org/topic_3511
tools:
- name: FIMO
  role: Scans risk- and reference-allele flanked sequences against PWM libraries to
    compute per-TF motif binding scores for fold-change analysis
- name: HOCOMOCO v11 (Full Collection)
  role: One of three PWM libraries used as motif sources for FIMO scanning
- name: JASPAR CORE (Non-Redundant) v2
  role: Second PWM library used as a motif source for FIMO scanning
- name: CIS-BP v2
  role: Third PWM library used as a motif source for FIMO scanning
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:5e64837ecc22bb8bd4be5ab0de040d347889bcc9da2fd158c30bc9bd8f888a96
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  tools:
  - CIS-BP
  - CIS-BP v2
  - FIMO
  - GrID-Net
  - HOCOMOCO v11
  - HOCOMOCO v11 (Full Collection)
  - JASPAR CORE (Non-Redundant) v2
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3125
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# tf-binding-affinity-pwm-scanning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts allele-specific transcription-factor binding affinity changes by scanning risk and reference allele sequences (flanked by ±20 bp genomic context from GRCh38/hg38) with FIMO against PWM libraries (HOCOMOCO v11 Full Collection, JASPAR CORE Non-Redundant v2, CIS-BP v2), then computing risk/reference fold changes per variant–TF pair to identify ≥20-fold binding disruptions in noncoding SCZ variants.

## When to use

Use when you have fine-mapped GWAS/SCZ variants with genomic coordinates and need to predict which transcription factors' binding affinities are disrupted by the risk allele versus the reference allele, specifically to prioritize variants with large (≥20-fold) motif affinity changes.

## When NOT to use

- When the question is linking variants to genes they regulate (use GrID-Net peak-gene linking instead) rather than predicting within-locus TF binding changes.
- When variants are not fine-mapped or lack confident genomic coordinates in GRCh38/hg38, since allele-specific sequence extraction requires exact variant positions.
- When you only have gene-level expression or chromatin accessibility data and no variant-level genomic coordinates.

## Inputs

- Fine-mapped GWAS variant list with genomic coordinates (GRCh38/hg38) and reference/risk alleles
- GRCh38/hg38 reference genome sequence
- Position weight matrices from HOCOMOCO v11 (Full Collection)
- Position weight matrices from JASPAR CORE (Non-Redundant) v2
- Position weight matrices from CIS-BP v2

## Outputs

- Per-variant-TF table of binding affinity fold changes (tf_affinity_changes.tsv)
- Summary of top disrupted TFs ranked by number of variants with diminished binding
- Count of variants exceeding the ≥20-fold affinity change threshold and their associated target genes

## How to apply

For each variant, extract two 41-bp sequences from the GRCh38/hg38 reference genome: the reference allele and the risk allele, each flanked by ±20 bp of genomic sequence around the variant position. Scan both allele sequence sets with FIMO using position weight matrices from HOCOMOCO v11 (Full Collection), JASPAR CORE (Non-Redundant) v2, and CIS-BP v2 to obtain per-TF motif binding scores. For each variant–TF pair, compute the fold change in binding affinity between the risk and reference alleles and tabulate the results. Identify variants with ≥20-fold TF affinity changes and summarize the top disrupted TFs (e.g., VEZF1, KLF15/KLF6, SP1/SP3/KLF16). Validate by confirming that the expected number of variants and target genes exhibit ≥20-fold changes and that the top disrupted TFs match the reported ranking.

## Related tools

- **FIMO** (Scans risk- and reference-allele flanked sequences against PWM libraries to compute per-TF motif binding scores for fold-change analysis)
- **HOCOMOCO v11 (Full Collection)** (One of three PWM libraries used as motif sources for FIMO scanning)
- **JASPAR CORE (Non-Redundant) v2** (Second PWM library used as a motif source for FIMO scanning)
- **CIS-BP v2** (Third PWM library used as a motif source for FIMO scanning)

## Examples

```
fimo --oc fimo_out HOCOMOCOv11_full.meme risk_alleles.fasta && fimo --oc fimo_out_ref HOCOMOCOv11_full.meme reference_alleles.fasta
```

## Evaluation signals

- Confirm that 28 of the 115 GrID-Net-linked SCZ variants (covering 49 target genes) exhibit ≥20-fold affinity changes for at least one TF.
- Check that the top disrupted TF ranking matches the reported order (VEZF1 with the most frequent diminished binding, 5 variants; then KLF15/KLF6, then SP1/SP3/KLF16).
- Verify that tf_affinity_changes.tsv reproduces the variant–TF relationships of Supplementary Data 2.
- Ensure each variant yields two sequences (reference and risk allele) each with exactly ±20 bp of flanking genomic context from GRCh38/hg38.
- Confirm fold changes are computed as risk-allele affinity divided by reference-allele affinity for every variant–TF pair across all three PWM libraries.

## Limitations

- Predicted TF binding affinity changes are indirect causal hypotheses; the article notes causal interactions are 'likely indirect and mediated by additional biophysical events, such as transcription factor binding' and would need perturbation-based validation.
- The analysis is limited to variants that overlap ATAC peaks from the 10x Multiome human corticogenesis dataset with PP≥0.05; variants outside this filtered set are not scanned.
- The ±20 bp flanking window captures only local sequence context; variants affecting binding at greater distances, or effects involving 3D chromatin structure (e.g., Hi-C/CTCF topology), are not modeled.

## Evidence

- [discussion] Predict TF binding affinity changes by FIMO PWM scanning of variant-flanked sequences (±20 bp, GRCh38/hg38): [evidence span withheld — anchor status: not_found]
