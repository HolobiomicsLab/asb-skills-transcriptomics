---
name: nucleotide-frequency-logo-analysis
description: Use when you have balanced positive/negative sequence fragments centered
  on a candidate site and need to visualize per-position nucleotide enrichment or
  depletion between the two classes.
when_to_use_negative:
- The goal is building or evaluating a predictive model (e.g., Staem5 stacking classifier)
  rather than qualitative motif inspection — use feature encoding such as PSP, k-mer,
  PCPseDNC, or PseEIIP instead
- Input sequences are not aligned with a common reference position (candidate site
  centered), since Two Sample Logo requires per-position correspondence between classes
- Positive and negative sets are highly imbalanced, which can distort per-position
  frequency comparisons without correction
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3307
tools:
- name: Two Sample Logo
  role: Web tool used to compute and render per-position nucleotide frequency differences
    between m5C-positive and m5C-negative fragments
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:1b40c9f607da41f5a03bcbf17587b41cc9a852d89d0506cf5eba9d10b46b9162
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Two Sample Logo
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nucleotide-frequency-logo-analysis

## Summary

Two Sample Logo analysis that compares per-position nucleotide frequencies between m5C-positive and m5C-negative RNA fragments to reveal whether cytosine methylation sites carry distinctive sequence motifs. Applied in Staem5 to 41-bp A. thaliana and M. musculus fragments with the candidate cytosine centered.

## When to use

Use when you have balanced positive/negative sequence fragments centered on a candidate site and need to visualize per-position nucleotide enrichment or depletion between the two classes.

## When NOT to use

- The goal is building or evaluating a predictive model (e.g., Staem5 stacking classifier) rather than qualitative motif inspection — use feature encoding such as PSP, k-mer, PCPseDNC, or PseEIIP instead
- Input sequences are not aligned with a common reference position (candidate site centered), since Two Sample Logo requires per-position correspondence between classes
- Positive and negative sets are highly imbalanced, which can distort per-position frequency comparisons without correction

## Inputs

- 41-bp m5C-positive fragment sequences in FASTA format (A. thaliana: 5,298; M. musculus: 4,563), candidate cytosine centered
- Balanced m5C-negative fragment sequences in FASTA format (equal counts per species)
- Position mapping defining upstream positions -18 to -1 and downstream positions +1 to +18 around the central cytosine

## Outputs

- Two Sample Logo image per species showing enriched/depleted nucleotides at each aligned position
- Per-position nucleotide frequency/enrichment table (nucleotide_frequencies.csv) covering every aligned position for both species

## How to apply

Load the balanced m5C-positive and m5C-negative training fragments (A. thaliana 5,298 vs 5,298; M. musculus 4,563 vs 4,563) and align the 41-bp fragments so the candidate cytosine sits at position 0, indexing upstream positions -18 to -1 and downstream positions +1 to +18 per nucleotide. Run Two Sample Logo to compute per-position nucleotide frequency differences between positive and negative sets and identify statistically enriched residues. Render the two-sample logo image per species and export the per-position frequency/enrichment values (e.g., to nucleotide_frequencies.csv). Validate by confirming the logo reproduces the reported pattern (e.g., C enrichment in upstream positions of A. thaliana fragments) and that the frequency table covers every aligned position for both species.

## Related tools

- **Two Sample Logo** (Web tool used to compute and render per-position nucleotide frequency differences between m5C-positive and m5C-negative fragments)

## Evaluation signals

- The rendered logo qualitatively matches the reported pattern, e.g., C enrichment in upstream positions of A. thaliana fragments
- nucleotide_frequencies.csv contains one frequency row per aligned position (-18 to +18) for both species
- Positive and negative input sets are balanced per species (A. thaliana 5,298 vs 5,298; M. musculus 4,563 vs 4,563)
- The central position in the alignment corresponds to the candidate cytosine (C) in every fragment
- The conclusion is consistent with the paper's finding that m5C sites in A. thaliana and M. musculus show no notable sequence motifs relative to non-m5C sites

## Limitations

- m5C sites in A. thaliana and M. musculus do not have notable sequence motifs compared with non-m5C sites, so the analysis is mainly a negative/qualitative validation rather than a discriminative feature source
- The analysis is species-specific; the article notes it could be difficult to develop a general cross-species model and species-specific analyses are necessary

## Evidence

- [full_text] Workflow details from source card: [evidence span withheld — anchor status: not_found]
