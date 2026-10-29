---
name: string-network-density-normalization
description: Use when quantifying how connected a protein is within the human STRING
  interaction network, or when reproducing/validating reported network density percentages
  (e.g. HSPH1 = 78% from Fig. 1C of Kahraman et al., Scientific Reports 2020).
when_to_use_negative:
- When the analysis goal is isoform switching detection or functional impact scoring
  (CanIsoNet's primary use case) rather than raw network connectivity quantification
- When the interaction network is not STRING (the weighted-sum normalization is defined
  on STRING combined interaction scores and calibrated against the full human STRING
  network maximum)
- When a percentile of node degree alone suffices — the density score uses interaction
  weights, not just edge counts
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: STRING
  role: Source of the human protein-protein interaction network and combined interaction
    scores used to compute weighted interaction sums
- name: CanIsoNet
  role: Database/webserver that displays isoform switching events and network disruptions
    on the STRING interaction network, the context in which density scores are visualized
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:bf0baca86cd6473acdbcfb1345c0826cf0dde98d89334005f45c5e8cbe2ec628
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - STRING
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# string-network-density-normalization

## Summary

Compute a per-protein network density score in the human STRING protein-protein interaction network by normalizing each protein's weighted interaction sum against the network-wide maximum, expressed as a percentile-like percentage. Used in CanIsoNet/Kahraman et al. 2020 to quantify how connected a gene (e.g. HSPH1, 78%) is relative to all other proteins.

## When to use

Use when quantifying how connected a protein is within the human STRING interaction network, or when reproducing/validating reported network density percentages (e.g. HSPH1 = 78% from Fig. 1C of Kahraman et al., Scientific Reports 2020).

## When NOT to use

- When the analysis goal is isoform switching detection or functional impact scoring (CanIsoNet's primary use case) rather than raw network connectivity quantification
- When the interaction network is not STRING (the weighted-sum normalization is defined on STRING combined interaction scores and calibrated against the full human STRING network maximum)
- When a percentile of node degree alone suffices — the density score uses interaction weights, not just edge counts

## Inputs

- Human STRING protein-protein interaction network with combined interaction scores (from the STRING database)
- Protein/gene identifier of interest (e.g. HSPH1)

## Outputs

- Per-protein network density score as a normalized percentage
- hsph1_network_density.csv with computed and reported values side by side

## How to apply

Download the human STRING protein-protein interaction network with combined interaction scores. For the protein of interest (e.g. HSPH1), extract all interactions involving it and sum their interaction weights to obtain the raw weighted interaction sum; compute the same weighted sum for every protein in the network. Take the maximum weighted sum across the entire network and normalize the protein's score by this maximum, expressing it as a percentage — this is the network density score, interpretable as having more interactions in its local neighbourhood than that percentage of all other proteins. Compare the computed percentage against any published value (e.g. 78% for HSPH1 from Fig. 1C of Kahraman et al. 2020) within a small tolerance (±1%). This metric feeds CanIsoNet's visualization of network disruptions on the STRING interaction network.

## Related tools

- **STRING** (Source of the human protein-protein interaction network and combined interaction scores used to compute weighted interaction sums)
- **CanIsoNet** (Database/webserver that displays isoform switching events and network disruptions on the STRING interaction network, the context in which density scores are visualized) — https://github.com/kahramanlab/CanIsoNet_Web

## Examples

```
import pandas as pd; net = pd.read_csv('string_human_combined.tsv', sep='\t'); sums = net.groupby('protein1')['combined_score'].sum(); density = 100 * sums['HSPH1'] / sums.max(); pd.DataFrame({'computed': [density], 'reported': [78]}).to_csv('hsph1_network_density.csv')
```

## Evaluation signals

- Computed network density score for HSPH1 equals the reported 78% from Fig. 1C of Kahraman et al., Scientific Reports 2020 within ±1% tolerance
- The maximum weighted interaction sum used for normalization is computed across every protein in the human STRING network, not a subset
- Output CSV (hsph1_network_density.csv) contains computed and reported values side by side
- Score is expressed as a percentage in [0, 100], where X% means more interactions in the local neighbourhood than X% of all other proteins

## Limitations

- The score is relative to the current human STRING network — re-computation depends on the STRING version and its combined interaction scores
- It is a whole-network normalization (max across all human proteins), so disease- or tissue-specific subnetworks will yield different percentages
- The reported 78% value originates from Kahraman et al. 2020 (Fig. 1C); the CanIsoNet paper itself documents the webserver, not the exact density-score formula

## Evidence

- [intro] STRING: "the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network"
- [readme] readme: "Exploring and visualizing network disruptions in the STRING interaction network."
- [other] workflow: [evidence span withheld — anchor status: not_found]
- [other] finding: [evidence span withheld — anchor status: not_found]
