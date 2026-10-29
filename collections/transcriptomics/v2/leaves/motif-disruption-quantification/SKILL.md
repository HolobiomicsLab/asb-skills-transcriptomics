---
name: motif-disruption-quantification
description: Use when a set of fine-mapped GWAS variants (e.g. linked to target genes
  via GrID-Net peak-gene links) needs to be prioritized by predicted TF binding disruption,
  using PWM-based motif scanning of alternate vs. reference alleles.
when_to_use_negative:
- Variants have not been linked to target genes (e.g. via GrID-Net peak-gene scores),
  so there is no prioritized variant set to annotate with motif disruptions
- The goal is direct experimental measurement of binding (e.g. ChIP-seq/EMSA) rather
  than PWM-based in silico prediction of affinity changes
- Input is protein-level or expression-level data with no variant coordinates on GRCh38/hg38
  from which allele-flanked sequences can be extracted
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0625
tools:
- name: FIMO
  role: Scans reference- and risk-allele flanked sequences with PWMs to obtain per-TF
    motif binding scores used to compute fold changes
- name: HOCOMOCO v11
  role: Position weight matrix collection (Full Collection) used as one of the motif
    databases for FIMO scanning
- name: JASPAR CORE (Non-Redundant) v2
  role: Position weight matrix collection used as one of the motif databases for FIMO
    scanning
- name: CIS-BP v2
  role: Position weight matrix collection used as one of the motif databases for FIMO
    scanning
- name: GrID-Net
  role: Upstream method that links SCZ variants to target genes, defining the variant
    set whose motifs are scanned
  repo: https://github.com/alexw16/gridnet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:e66e106666b3806d93fca02d29430c32ddbeaacb9b175bc3cc6ec2a14897f2db
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
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# motif-disruption-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify how variant alleles alter predicted transcription-factor binding affinity by scanning reference- and risk-allele flanked sequences (±20 bp, GRCh38/hg38) with FIMO against PWM collections (HOCOMOCO v11 Full Collection, JASPAR CORE Non-Redundant v2, CIS-BP v2) and computing per-variant, per-TF fold changes. Used here to identify GrID-Net-linked SCZ variants causing ≥20-fold binding affinity disruptions.

## When to use

Use when a set of fine-mapped GWAS variants (e.g. linked to target genes via GrID-Net peak-gene links) needs to be prioritized by predicted TF binding disruption, using PWM-based motif scanning of alternate vs. reference alleles.

## When NOT to use

- Variants have not been linked to target genes (e.g. via GrID-Net peak-gene scores), so there is no prioritized variant set to annotate with motif disruptions
- The goal is direct experimental measurement of binding (e.g. ChIP-seq/EMSA) rather than PWM-based in silico prediction of affinity changes
- Input is protein-level or expression-level data with no variant coordinates on GRCh38/hg38 from which allele-flanked sequences can be extracted

## Inputs

- Set of GrID-Net-linked SCZ variants (115 variants with variant position, reference and risk alleles)
- GRCh38/hg38 reference genome FASTA
- PWM/motif collections: HOCOMOCO v11 (Full Collection), JASPAR CORE (Non-Redundant) v2, CIS-BP v2
- FIMO motif-scanning tool

## Outputs

- tf_affinity_changes.tsv — full variant-TF table of binding affinity fold changes (risk vs. reference allele)
- List of variants with ≥20-fold TF affinity changes (e.g. 28 of 115 variants, covering 49 target genes)
- Ranked summary of top disrupted TFs (VEZF1, KLF15/KLF6, SP1/SP3/KLF16)

## How to apply

First extract, for each variant, two sequences from the GRCh38/hg38 reference genome: one carrying the reference allele and one the risk allele, each flanked by ±20 bp of genomic sequence around the variant position. Scan both sequence sets with FIMO using position weight matrices from HOCOMOCO v11 (Full Collection), JASPAR CORE (Non-Redundant) v2, and CIS-BP v2 to obtain per-TF motif binding scores. For each variant-TF pair, compute the binding-affinity fold change between the risk and reference alleles and tabulate all results. Flag variants with ≥20-fold affinity changes for any TF, then summarize the most frequently disrupted TFs (e.g. VEZF1, KLF15/KLF6, SP1/SP3/KLF16) and save the full table as tf_affinity_changes.tsv.

## Related tools

- **FIMO** (Scans reference- and risk-allele flanked sequences with PWMs to obtain per-TF motif binding scores used to compute fold changes)
- **HOCOMOCO v11** (Position weight matrix collection (Full Collection) used as one of the motif databases for FIMO scanning)
- **JASPAR CORE (Non-Redundant) v2** (Position weight matrix collection used as one of the motif databases for FIMO scanning)
- **CIS-BP v2** (Position weight matrix collection used as one of the motif databases for FIMO scanning)
- **GrID-Net** (Upstream method that links SCZ variants to target genes, defining the variant set whose motifs are scanned) — https://github.com/alexw16/gridnet

## Examples

```
fimo --thresh 1e-4 hocomoco_v11_full.meme risk_alleles.fa > fimo_risk.tsv; fimo --thresh 1e-4 hocomoco_v11_full.meme ref_alleles.fa > fimo_ref.tsv  # then compute risk/ref fold changes per variant-TF pair
```

## Evaluation signals

- Confirm that 28 of the 115 GrID-Net-linked SCZ variants exhibit ≥20-fold affinity changes for at least one TF
- Confirm those 28 variants cover 49 target genes
- Verify the top disrupted TFs match the reported ranking: VEZF1 most frequent diminished binding (5 variants), followed by KLF15/KLF6 and SP1/SP3/KLF16
- Reproduce the variant-TF relationships of Supplementary Data 2 from the output tf_affinity_changes.tsv
- Sanity-check fold changes are directionally consistent with allele swapping (diminished vs. enhanced binding per TF per variant)

## Limitations

- Predictions rely on PWM-based motif scanning and are indirect proxies — the article notes inferred causal interactions 'are likely indirect and mediated by additional biophysical events, such as transcription factor binding'
- Restricting flanks to ±20 bp around the variant position may miss disruptions of binding sites partially overlapping the window
- Results depend on the reference genome build (GRCh38/hg38) and on the specific PWM collections used (HOCOMOCO v11, JASPAR v2, CIS-BP v2)

## Evidence

- [discussion] Predict TF binding affinity changes by FIMO PWM scanning of variant-flanked sequences (±20 bp, GRCh38/hg38): [evidence span withheld — anchor status: not_found]
- [results] Workflow validation: 28 of 115 variants with ≥20-fold changes, VEZF1 most frequent: [evidence span withheld — anchor status: not_found]
