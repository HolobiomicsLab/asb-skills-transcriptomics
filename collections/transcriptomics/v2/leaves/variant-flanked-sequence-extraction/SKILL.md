---
name: variant-flanked-sequence-extraction
description: Use when you have a set of fine-mapped variant positions plus REF/ALT
  alleles and need allele-specific sequence windows centered on each variant for motif
  scanning (e.g., FIMO PWM analysis of binding-affinity disruption).
when_to_use_negative:
- Variants lack both reference and alternate alleles mapped to a single GRCh38/hg38
  position (e.g., only summary statistics without alleles)
- The goal is gene-level or locus-level attribution rather than allele-specific motif
  disruption analysis
edam_operation: http://edamontology.org/operation_2402
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3175
tools:
- name: GrID-Net
  role: Upstream method that produces the 115 fine-mapped SCZ variant-target-gene
    links whose flanking sequences are extracted here
  repo: https://github.com/alexw16/gridnet
- name: FIMO
  role: Downstream motif scanner that consumes the extracted allele sequences with
    PWMs from HOCOMOCO v11, JASPAR CORE v2, and CIS-BP v2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:c0b74d2f7b471ca5fda0141d299834389bf180911f3cefe459d55c6eb9baf63b
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
  - FIMO
  - GrID-Net
  - HOCOMOCO v11
  - JASPAR CORE (Non-Redundant) v2
  edam_operation: http://edamontology.org/operation_2402
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# variant-flanked-sequence-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts, for each GrID-Net-linked schizophrenia variant, two ±20 bp-flanked allele sequences (reference and risk allele) from the GRCh38/hg38 reference genome so they can be scored for transcription-factor binding affinity changes with FIMO PWM scans.

## When to use

Use when you have a set of fine-mapped variant positions plus REF/ALT alleles and need allele-specific sequence windows centered on each variant for motif scanning (e.g., FIMO PWM analysis of binding-affinity disruption).

## When NOT to use

- Variants lack both reference and alternate alleles mapped to a single GRCh38/hg38 position (e.g., only summary statistics without alleles)
- The goal is gene-level or locus-level attribution rather than allele-specific motif disruption analysis

## Inputs

- 115 GrID-Net-linked fine-mapped SCZ variant positions (GRCh38/hg38 coordinates) with reference and risk alleles
- GRCh38/hg38 reference genome sequence
- GrID-Net peak-gene linkage results (variant-to-target-gene assignments, e.g., 49 target genes)

## Outputs

- FASTA of reference-allele sequences, each with ±20 bp genomic flank around the variant position
- FASTA of risk-allele sequences, each with ±20 bp genomic flank around the variant position
- Per-variant sequence records feeding downstream FIMO scans and the tf_affinity_changes.tsv fold-change table

## How to apply

Start from the GrID-Net-linked SCZ variants (115 variants retained as fine-mapped variants with PP≥0.05 overlapping peaks from the 10x Multiome human corticogenesis dataset, linked to up to 3 highest-scoring genes per peak using distance-dependent GrID-Net scores in the top 50% of peak-gene scores). For each variant, fetch the reference genome sequence at the variant position from GRCh38/hg38 and construct two sequences: one carrying the reference allele and one carrying the risk allele, each flanked by ±20 bp of genomic sequence around the variant position. Write both allele sequence sets out as FASTA so both can be scanned identically by FIMO against PWM collections (HOCOMOCO v11 Full Collection, JASPAR CORE Non-Redundant v2, CIS-BP v2), enabling a per-variant-TF fold-change computation between alleles. The ±20 bp window balances capturing motif context around the variant position against keeping the scanned window small enough that affinity changes are attributable to the variant itself.

## Related tools

- **GrID-Net** (Upstream method that produces the 115 fine-mapped SCZ variant-target-gene links whose flanking sequences are extracted here) — https://github.com/alexw16/gridnet
- **FIMO** (Downstream motif scanner that consumes the extracted allele sequences with PWMs from HOCOMOCO v11, JASPAR CORE v2, and CIS-BP v2)

## Examples

```
samtools faidx GRCh38.fa chr:start-20:end-20 > ref_allele.fa; samtools faidx GRCh38.fa --region-with-ALT-substituted chr:start-20:end+20 > risk_allele.fa
```

## Evaluation signals

- Extracted sequence count equals 115 variants × 2 alleles (reference and risk sequences per variant)
- Each extracted sequence is 41 bp (variant position plus ±20 bp flank) and matches GRCh38/hg38 at flanking positions
- All 28 variants with ≥20-fold TF affinity changes (covering 49 target genes) trace back to extracted sequence pairs, reproducing Supplementary Data 2 variant-TF relationships
- Downstream FIMO fold-change ranking reproduces the reported top disrupted TFs (VEZF1 most frequent with 5 variants; KLF15/KLF6; SP1/SP3/KLF16)

## Limitations

- The ±20 bp window may miss longer motifs whose disruption depends on extended flanking context; the paper's predictions are computationally inferred and require perturbation-based validation
- Analysis is limited to variants overlapping peaks from the 10x Multiome human corticogenesis dataset, which may explain only a subset of SCZ phenotypes

## Evidence

- [discussion] Flanked variant alleles for FIMO scanning: [evidence span withheld — anchor status: not_found]
- [discussion] Variant filter that defines the input variant set: [evidence span withheld — anchor status: not_found]
- [abstract] Downstream result being validated: [evidence span withheld — anchor status: not_found]
