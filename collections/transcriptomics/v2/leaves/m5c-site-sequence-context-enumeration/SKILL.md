---
name: m5c-site-sequence-context-enumeration
description: Use when balanced m5C-positive/negative RNA fragment sets must be compared
  for position-specific nucleotide enrichment around a candidate cytosine, e.g., to
  test whether m5C sites carry distinctive flanking motifs before encoding features.
when_to_use_negative:
- Input fragments are variable-length or not cytosine-centered, so positions -18 to
  -1 / +1 to +18 cannot be indexed consistently across positive and negative sets
- The goal is m5C site prediction rather than motif/context characterization — use
  the Staem5 stacked ensemble (m5C_predict.py) instead of frequency-difference analysis
- The positive and negative sets are unbalanced, which would bias per-position frequency
  differences in the two-sample comparison
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Two Sample Logo
  role: Renders the two-sample sequence logo comparing nucleotide frequencies of m5C-positive
    vs. m5C-negative fragments at each aligned position
- name: Staem5
  role: Source of the balanced m5C-positive/negative training fragments for A. thaliana
    and M. musculus whose sequence contexts are enumerated
  repo: https://github.com/Cxd-626/Staem5.git
- name: PEA-m5C
  role: Related predictor that similarly derives features from flanking sequences
    of m5C modifications; provides the Ara-m5C candidate site context
  repo: https://github.com/cma2015/PEA-m5C
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:d025fef2dd2b109efc150774026e693cda48c5c09feed9c555d84f62bb2e23fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - PEA-m5C
  - Staem5
  - Two Sample Logo
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# m5c-site-sequence-context-enumeration

## Summary

Characterizes nucleotide preferences around RNA m5C sites by aligning m5C-positive and m5C-negative fragments with the candidate cytosine centered, then computing per-position nucleotide frequency differences (Two Sample Logo approach) for A. thaliana and M. musculus.

## When to use

Use when balanced m5C-positive/negative RNA fragment sets must be compared for position-specific nucleotide enrichment around a candidate cytosine, e.g., to test whether m5C sites carry distinctive flanking motifs before encoding features.

## When NOT to use

- Input fragments are variable-length or not cytosine-centered, so positions -18 to -1 / +1 to +18 cannot be indexed consistently across positive and negative sets
- The goal is m5C site prediction rather than motif/context characterization — use the Staem5 stacked ensemble (m5C_predict.py) instead of frequency-difference analysis
- The positive and negative sets are unbalanced, which would bias per-position frequency differences in the two-sample comparison

## Inputs

- Balanced m5C-positive fragment set for A. thaliana (5,298 sequences, 41 bp, candidate cytosine centered)
- Balanced m5C-negative fragment set for A. thaliana (5,298 sequences, 41 bp, cytosine centered)
- Balanced m5C-positive fragment set for M. musculus (4,563 sequences, 41 bp, cytosine centered)
- Balanced m5C-negative fragment set for M. musculus (4,563 sequences, 41 bp, cytosine centered)

## Outputs

- Two-sample sequence logo image per species (positive vs. negative nucleotide preferences)
- Per-position nucleotide frequency/enrichment table exported to nucleotide_frequencies.csv covering every aligned position (-18 to +18) for both species

## How to apply

Load the balanced m5C-positive and m5C-negative training fragments (A. thaliana: 5,298 vs 5,298; M. musculus: 4,563 vs 4,563) as 41-bp sequences with the candidate cytosine centered. Index each aligned position so upstream positions -18 to -1 and downstream positions +1 to +18 are addressable per nucleotide. Compute per-position nucleotide frequency differences between the positive and negative sets following the Two Sample Logo approach, which flags statistically enriched residues at each offset. Render a two-sample sequence logo per species for qualitative inspection and export the per-position nucleotide frequency/enrichment table to nucleotide_frequencies.csv. The rationale (per the Staem5 paper) is to verify whether m5C fragments carry sequence motifs that downstream encodings such as position-specific propensity (PSP) can exploit, or whether, as reported, the two species lack notable motifs relative to non-m5C sites.

## Related tools

- **Two Sample Logo** (Renders the two-sample sequence logo comparing nucleotide frequencies of m5C-positive vs. m5C-negative fragments at each aligned position)
- **Staem5** (Source of the balanced m5C-positive/negative training fragments for A. thaliana and M. musculus whose sequence contexts are enumerated) — https://github.com/Cxd-626/Staem5.git
- **PEA-m5C** (Related predictor that similarly derives features from flanking sequences of m5C modifications; provides the Ara-m5C candidate site context) — https://github.com/cma2015/PEA-m5C

## Examples

```
from collections import Counter; import pandas as pd; freq = {i: Counter(seq[i] for seq in frags) for i in range(41)}; pd.DataFrame(freq).to_csv('nucleotide_frequencies.csv')
```

## Evaluation signals

- The nucleotide_frequencies.csv table covers every aligned position (upstream -18 to -1 and downstream +1 to +18) for both A. thaliana and M. musculus
- Qualitative check: the rendered two-sample logo reproduces the reported pattern (e.g., C enrichment in upstream positions of A. thaliana m5C fragments)
- Positive and negative fragment counts are balanced per species (5,298 vs 5,298 for A. thaliana; 4,563 vs 4,563 for M. musculus) before computing frequency differences
- Per-position nucleotide frequencies sum to 1 across A/C/G/U(T) at each offset within each set
- Expected outcome: no prominent sequence motif should emerge, consistent with the paper's finding that m5C sites in both species lack notable motifs relative to non-m5C sites

## Limitations

- The reported result is a negative finding: m5C sites in A. thaliana and M. musculus do not have notable sequence motifs compared with non-m5C sites, so strong motif enrichment should not be expected
- Nucleotide-context patterns are species-specific; the paper notes it could be difficult to develop a general model for cross-species prediction, making species-specific analysis necessary
- Validation is largely qualitative (visual logo inspection) rather than a formal statistical significance call per position

## Evidence

- [readme] flanking-sequence features in related predictor: [evidence span withheld — anchor status: external_doc]
