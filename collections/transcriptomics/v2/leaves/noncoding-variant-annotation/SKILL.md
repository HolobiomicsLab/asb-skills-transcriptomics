---
name: noncoding-variant-annotation
description: Use when fine-mapped noncoding GWAS variants (e.g., SCZ variants linked
  to genes by GrID-Net) need functional annotation of allele-specific TF binding disruption
  in accessible regulatory peaks.
when_to_use_negative:
- Variants are coding or splice-site; FIMO PWM scanning of ±20 bp flanks only models
  TF motif disruption, not protein or splicing effects
- No allele-specific or positional variant information is available (only aggregate
  locus-level signals) — fold-change computation per variant-TF pair cannot be performed
- The variant lies outside assayed accessible chromatin (ATAC peaks); the article
  restricts to variants overlapping peaks from the 10x Multiome human corticogenesis
  dataset
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: FIMO
  role: Scans reference- and risk-allele ±20 bp flanked sequences against PWMs to
    produce per-TF motif binding scores used to compute risk/reference fold changes
- name: HOCOMOCO v11
  role: Full Collection position weight matrices supplying one of the three TF motif
    motif databases for FIMO scanning
- name: JASPAR CORE (Non-Redundant) v2
  role: Non-redundant PWM collection used alongside HOCOMOCO and CIS-BP for motif
    scoring
- name: CIS-BP v2
  role: Third PWM collection scanned by FIMO to maximize TF coverage of predicted
    binding disruptions
- name: GrID-Net
  role: Upstream tool that links fine-mapped SCZ variants to target genes (distance-dependent
    scores, top 50% of peak-gene pairs, up to 3 genes per peak), providing the variant
    set annotated here
  repo: https://github.com/alexw16/gridnet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:cdd41d0edc605a5b72f660d6c98b23a1a14f5fad4b327c7fc6d1b3e6c61024ce
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  repo_url: https://github.com/alexw16/gridnet
  tools:
  - CIS-BP
  - CIS-BP v2
  - FIMO
  - GrID-Net
  - HOCOMOCO v11
  - JASPAR CORE (Non-Redundant) v2
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# noncoding-variant-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates noncoding genetic variants with predicted transcription-factor binding affinity changes by scanning reference- and risk-allele flanked sequences with FIMO against PWM collections, quantifying per-TF fold changes to prioritize variants with large regulatory disruptions.

## When to use

Use when fine-mapped noncoding GWAS variants (e.g., SCZ variants linked to genes by GrID-Net) need functional annotation of allele-specific TF binding disruption in accessible regulatory peaks.

## When NOT to use

- Variants are coding or splice-site; FIMO PWM scanning of ±20 bp flanks only models TF motif disruption, not protein or splicing effects
- No allele-specific or positional variant information is available (only aggregate locus-level signals) — fold-change computation per variant-TF pair cannot be performed
- The variant lies outside assayed accessible chromatin (ATAC peaks); the article restricts to variants overlapping peaks from the 10x Multiome human corticogenesis dataset

## Inputs

- Fine-mapped noncoding variants with reference and risk alleles and GRCh38/hg38 genomic positions (e.g., SCZ variants with PP >= 0.05 overlapping ATAC peaks)
- Variant-to-target-gene links (e.g., 115 GrID-Net-linked SCZ variants with associated target genes)
- GRCh38/hg38 reference genome sequence
- PWM motif collections: HOCOMOCO v11 (Full Collection), JASPAR CORE (Non-Redundant) v2, CIS-BP v2

## Outputs

- tf_affinity_changes.tsv: full per-variant, per-TF binding affinity fold-change table
- List of variants with >=20-fold TF affinity changes (e.g., 28 of 115 variants covering 49 target genes)
- Ranked summary of most frequently disrupted TFs (e.g., VEZF1 with 5 variants showing diminished binding; KLF15/KLF6, SP1/SP3/KLF16)

## How to apply

For each variant, extract two 41-bp sequences from the GRCh38/hg38 reference genome: one carrying the reference allele and one the risk allele, each flanked by ±20 bp of genomic sequence around the variant position. Scan both allele sequence sets with FIMO using position weight matrices from HOCOMOCO v11 (Full Collection), JASPAR CORE (Non-Redundant) v2, and CIS-BP v2 to obtain per-TF motif binding scores. For each variant-TF pair, compute the binding affinity fold change between the risk and reference alleles and tabulate all results. Filter for variants causing >=20-fold TF affinity changes and rank TFs by frequency of disrupted binding (e.g., VEZF1, KLF15/KLF6, SP1/SP3/KLF16). Rationale: restricting to variants already linked to target genes (via GrID-Net peak-gene scoring, PP>=0.05 fine-mapped variants overlapping ATAC peaks) ties the motif-level prediction to causal locus-gene hypotheses rather than nearest-gene annotations.

## Related tools

- **FIMO** (Scans reference- and risk-allele ±20 bp flanked sequences against PWMs to produce per-TF motif binding scores used to compute risk/reference fold changes)
- **HOCOMOCO v11** (Full Collection position weight matrices supplying one of the three TF motif motif databases for FIMO scanning)
- **JASPAR CORE (Non-Redundant) v2** (Non-redundant PWM collection used alongside HOCOMOCO and CIS-BP for motif scoring)
- **CIS-BP v2** (Third PWM collection scanned by FIMO to maximize TF coverage of predicted binding disruptions)
- **GrID-Net** (Upstream tool that links fine-mapped SCZ variants to target genes (distance-dependent scores, top 50% of peak-gene pairs, up to 3 genes per peak), providing the variant set annotated here) — https://github.com/alexw16/gridnet

## Examples

```
fimo --thresh 1e-4 hocomoco_v11_full.meme risk_alleles.fa > risk_fimo.txt; fimo --thresh 1e-4 hocomoco_v11_full.meme ref_alleles.fa > ref_fimo.txt
```

## Evaluation signals

- Replication check: 28 of the 115 GrID-Net-linked SCZ variants (covering 49 target genes) show >=20-fold affinity change for at least one TF
- Ranking check: VEZF1 emerges as the TF with the most frequent diminished binding (5 variants), followed by KLF15/KLF6 and SP1/SP3/KLF16
- tf_affinity_changes.tsv contains a fold-change value for every variant-TF pair across all three PWM collections (HOCOMOCO v11, JASPAR CORE v2, CIS-BP v2)
- Sequence construction check: each extracted allele sequence is 41 bp (variant position ± 20 bp) from GRCh38/hg38 with the correct reference or risk allele at the center
- Fold changes are computed as risk-allele score relative to reference-allele score per variant-TF pair, with direction (diminished vs. increased) recorded

## Limitations

- Predicted binding disruptions are indirect, biophysical hypotheses: 'such causal interactions are likely indirect and mediated by additional biophysical events, such as transcription factor binding' and require perturbation-based validation
- Only variants overlapping ATAC peaks from the corticogenesis dataset are annotated; noncoding variants outside assayed accessible peaks are missed
- PWM scanning captures only single-motif affinity and ignores chromatin context, 3D genome structure (Hi-C/CTCF topology), and cooperative TF binding

## Evidence

- [discussion] FIMO PWM scanning of variant-flanked sequences: [evidence span withheld — anchor status: not_found]
- [discussion] Variant retention filter: [evidence span withheld — anchor status: not_found]
