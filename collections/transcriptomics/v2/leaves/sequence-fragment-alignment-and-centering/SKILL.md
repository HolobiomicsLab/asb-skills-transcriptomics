---
name: sequence-fragment-alignment-and-centering
description: Use when you need to test whether m5C-containing fragments show distinctive
  sequence motifs vs non-m5C fragments by comparing per-position nucleotide enrichment
  across cytosine-centered, equal-length flanking sequences.
when_to_use_negative:
- Fragments are variable-length or lack a shared anchor, so no cytosine-centered coordinate
  frame (e.g., -18 to -1 upstream indexing) can be defined across sequences
- The goal is m5C site prediction rather than motif characterization — in that case
  use the Staem5 predictor (m5C_predict.py) instead of descriptive logo analysis
- Input is already a per-position feature table or encoded features (PSP, k-mer, PCPseDNC,
  PseEIIP); alignment/centering has already been done
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Two Sample Logo
  role: Computes per-position nucleotide frequency differences between m5C-positive
    and m5C-negative fragments and renders the two-sample logo
- name: Staem5
  role: Source of the balanced 41-bp m5C-positive/negative training fragments for
    A. thaliana and M. musculus that are aligned and centered
  repo: https://github.com/Cxd-626/Staem5.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:d0b2047b75871b2237e88308d78e2368241d6bdfcdf76a57727683bf878140fc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - Staem5
  - Two Sample Logo
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# sequence-fragment-alignment-and-centering

## Summary

Aligns m5C-positive and m5C-negative RNA fragments (e.g., the 41-bp Staem5 training windows for A. thaliana and M. musculus) with the candidate cytosine centered, so per-position nucleotide frequency differences can be computed and rendered as two-sample logos.

## When to use

Use when you need to test whether m5C-containing fragments show distinctive sequence motifs vs non-m5C fragments by comparing per-position nucleotide enrichment across cytosine-centered, equal-length flanking sequences.

## When NOT to use

- Fragments are variable-length or lack a shared anchor, so no cytosine-centered coordinate frame (e.g., -18 to -1 upstream indexing) can be defined across sequences
- The goal is m5C site prediction rather than motif characterization — in that case use the Staem5 predictor (m5C_predict.py) instead of descriptive logo analysis
- Input is already a per-position feature table or encoded features (PSP, k-mer, PCPseDNC, PseEIIP); alignment/centering has already been done

## Inputs

- Balanced m5C-positive FASTA fragment set (41-bp windows centered on the m5C cytosine), e.g. 5,298 A. thaliana / 4,563 M. musculus sequences
- Size-matched m5C-negative FASTA fragment set (same 41-bp window centered on a non-m5C cytosine), e.g. 5,298 A. thaliana / 4,563 M. musculus sequences

## Outputs

- Two-sample sequence logo image per species (A. thaliana and M. musculus) showing enriched/depleted residues relative to the non-m5C set
- Per-position nucleotide frequency/enrichment table (e.g., nucleotide_frequencies.csv) covering every aligned position for both species

## How to apply

Load the balanced m5C-positive and m5C-negative training fragments (A. thaliana: 5,298 vs 5,298; M. musculus: 4,563 vs 4,563) and align each 41-bp fragment with the candidate cytosine at the center so upstream positions (-18 to -1) and downstream positions can be indexed per nucleotide. Compute per-position nucleotide frequency differences between the positive and negative sets using the Two Sample Logo approach to flag statistically enriched residues at each aligned position. Render one two-sample logo image per species and export the per-position frequency/enrichment values to a table such as nucleotide_frequencies.csv. The rationale is descriptive validation: the paper's Two Sample Logo analysis concluded that m5C sites lack strong motifs, so the logo mainly confirms the absence of large position-specific biases (with limited exceptions such as upstream C enrichment in A. thaliana).

## Related tools

- **Two Sample Logo** (Computes per-position nucleotide frequency differences between m5C-positive and m5C-negative fragments and renders the two-sample logo)
- **Staem5** (Source of the balanced 41-bp m5C-positive/negative training fragments for A. thaliana and M. musculus that are aligned and centered) — https://github.com/Cxd-626/Staem5.git

## Evaluation signals

- Every 41-bp fragment is aligned with the candidate cytosine at the same centered coordinate (upstream positions indexable as -18 to -1) before frequency computation
- Positive and negative sets remain size-balanced (5,298 vs 5,298 A. thaliana; 4,563 vs 4,563 M. musculus) at logo computation time
- The exported frequency table covers every aligned position for both species with no missing rows/columns
- The rendered logo qualitatively reproduces the reported pattern, e.g., C enrichment in upstream positions of A. thaliana fragments
- Overall enrichment magnitudes stay small, consistent with the paper's finding that m5C sites show no notable sequence motifs vs non-m5C sites

## Limitations

- The analysis is descriptive validation rather than a predictive feature: the paper found m5C sites in A. thaliana and M. musculus do not have notable sequence motifs compared with non-m5C sites, so logos mainly confirm weak/no enrichment
- Centering requires all fragments to share the same length and anchor (candidate cytosine at window center); this skill cannot align heterogeneous or unanchored sequences

## Evidence

- [other] Cytosine-centered alignment of 41-bp fragments: [evidence span withheld — anchor status: not_found]
