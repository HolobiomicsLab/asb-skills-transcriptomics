---
name: reported-value-benchmark-verification
description: Use when a published numeric result (e.g., a network statistic like a
  network density score reported in Fig. 1C of Kahraman et al., Scientific Reports
  2020) must be independently reproduced from raw network data to validate a pipeline
  or database-derived value.
when_to_use_negative:
- The analysis goal is predicting or browsing the pathogenic impact of isoform switching
  events across the 31 disease types in CanIsoNet, rather than verifying a published
  numeric benchmark.
- The quantity of interest depends on sample-level isoform switching calls (e.g.,
  per-sample expression values or dMDTs) rather than a static whole-network interaction
  statistic.
- The protein is absent from the human STRING network, so a normalized network density
  score cannot be defined.
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: STRING
  role: Source of the human protein-protein interaction network and combined interaction
    scores used to compute weighted interaction sums and the normalized network density
    score
- name: CanIsoNet
  role: Disease Specific Isoform Interaction Network database that displays isoform
    switching events on STRING interaction networks; source of the Kahraman et al.
    2020 benchmark context
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:d0d68616775308773425ad86d0f3c50cf3c94065056bfcdac2985c32d3faf1a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - STRING
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# reported-value-benchmark-verification

## Summary

A benchmarking skill that independently recomputes a published quantitative statistic — here, the HSPH1 network density score of 78% from Fig. 1C of Kahraman et al., Scientific Reports 2020 — from raw source data (the human STRING interaction network) and checks the recomputed value against the reported one within a defined tolerance.

## When to use

Use when a published numeric result (e.g., a network statistic like a network density score reported in Fig. 1C of Kahraman et al., Scientific Reports 2020) must be independently reproduced from raw network data to validate a pipeline or database-derived value.

## When NOT to use

- The analysis goal is predicting or browsing the pathogenic impact of isoform switching events across the 31 disease types in CanIsoNet, rather than verifying a published numeric benchmark.
- The quantity of interest depends on sample-level isoform switching calls (e.g., per-sample expression values or dMDTs) rather than a static whole-network interaction statistic.
- The protein is absent from the human STRING network, so a normalized network density score cannot be defined.

## Inputs

- Human STRING protein-protein interaction network with combined interaction scores
- Protein identifier of interest (HSPH1)
- Reference (reported) value: 78% network density score from Fig. 1C of Kahraman et al., Scientific Reports 2020

## Outputs

- hsph1_network_density.csv containing computed and reported network density percentages side by side
- Computed HSPH1 network density score as a percentage of the network-wide maximum weighted interaction sum

## How to apply

Download the human STRING protein-protein interaction network including its combined interaction scores, which CanIsoNet uses as its underlying interaction display layer. For every protein in the network, extract all interactions and sum their interaction weights to get a raw weighted interaction sum; for HSPH1 this is the sum of weights of its STRING partners. Take the maximum weighted interaction sum across all proteins in the network and divide the HSPH1 sum by this maximum, expressing the result as a percentage — this percentage is the network density score, defined as having more interactions in the local neighbourhood than a given percent of all other proteins. Accept the reproduction if the computed percentage matches the reported 78% within a small numerical tolerance (±1%), and write computed and reported values side by side to hsph1_network_density.csv for auditability. The verification anchors on the Kahraman et al. 2020 isoform-specific interaction network analysis, the precursor work to the CanIsoNet database.

## Related tools

- **STRING** (Source of the human protein-protein interaction network and combined interaction scores used to compute weighted interaction sums and the normalized network density score)
- **CanIsoNet** (Disease Specific Isoform Interaction Network database that displays isoform switching events on STRING interaction networks; source of the Kahraman et al. 2020 benchmark context) — https://github.com/kahramanlab/CanIsoNet_Web

## Examples

```
python -c "import pandas as pd; g=pd.read_csv('string_hsa_combined.tsv',sep='\t'); s=g.groupby('protein2').weight.sum(); pct=s['HSPH1']/s.max()*100; print(pct)"
```

## Evaluation signals

- Computed HSPH1 network density score equals the reported 78% within ±1% numerical tolerance.
- Normalization denominator is the maximum weighted interaction sum across every protein in the human STRING network, not a per-neighbourhood statistic.
- hsph1_network_density.csv contains both the computed and reported percentages side by side for direct comparison.
- HSPH1 interactions were extracted exclusively from the STRING combined-score edges, matching the CanIsoNet/STRING network basis.
- Ranking interpretation holds: HSPH1 has more interactions in its local neighbourhood than ~78% of all other proteins in the human STRING network.

## Limitations

- The reported 78% value is tied to a specific STRING network version and interaction-score set; changes to the STRING release or combined-score cutoffs may shift the recomputed percentage.
- The score is a relative (network-normalized) metric, so it is sensitive to the composition of the full human STRING network, not just HSPH1's immediate partners.
- Verification depends on the original figure (Fig. 1C of Kahraman et al., Scientific Reports 2020) as the ground truth; only a ±1% tolerance is specified, not an exact re-derivation of the paper's filtering steps.

## Evidence

- [readme] CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network.: "CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network."
- [readme] Kahraman et al. Scientific Reports 2020 citation: "Kahraman et al., [Pathogenic impact of transcript isoform switching in 1,209 cancer samples covering 27 cancer types using an isoform-specific interaction network Scientific Reports, 2020]"
- [abstract] CanIsoNet availability: "CanIsoNet is freely available at https://www.caniso.net."
- [readme] CanIsoNet coverage: "It holds annotations for 11,528 isoforms and 14,357 unique isoform switching events across 31 diseases."
