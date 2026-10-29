---
name: network-metric-reimplementation
description: Use when a paper reports a per-protein network statistic (e.g. HSPH1's
  network density score of 78%) that must be independently recomputed from the underlying
  interaction network rather than taken on trust.
when_to_use_negative:
- The target metric is a topology measure defined differently (e.g. degree centrality
  or clustering coefficient) rather than the maximum-normalized weighted interaction
  sum defined by Kahraman et al. — recomputing this recipe would give the wrong quantity.
- The network of interest is a disease-specific isoform interaction network from CanIsoNet
  (caniso.net) rather than the full human STRING network — density scores computed
  on the full network will not match scores on a filtered disease subnetwork.
- You only need to browse precomputed network disruptions or isoform switching events
  — CanIsoNet already serves these interactively and no recomputation is needed.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: STRING
  role: Source of the human protein-protein interaction network with combined interaction
    scores used to compute weighted interaction sums and the network density score
- name: CanIsoNet
  role: Webserver whose underlying methodology (Kahraman et al. 2020) defines the
    network density score; used to cross-check reported values and view isoform switching
    disruptions on the STRING network
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:159ff5f7b35c624ecdd862764e92d2a0f90e132cee0452970fdeb7f4135a3d39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - STRING
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# network-metric-reimplementation

## Summary

Reimplement a published per-protein network metric — here the 'network density score' of Kahraman et al. (Scientific Reports 2020, Fig. 1C), the basis of CanIsoNet's interaction-network displays — by recomputing weighted interaction sums over the human STRING network and normalizing against the network-wide maximum.

## When to use

Use when a paper reports a per-protein network statistic (e.g. HSPH1's network density score of 78%) that must be independently recomputed from the underlying interaction network rather than taken on trust.

## When NOT to use

- The target metric is a topology measure defined differently (e.g. degree centrality or clustering coefficient) rather than the maximum-normalized weighted interaction sum defined by Kahraman et al. — recomputing this recipe would give the wrong quantity.
- The network of interest is a disease-specific isoform interaction network from CanIsoNet (caniso.net) rather than the full human STRING network — density scores computed on the full network will not match scores on a filtered disease subnetwork.
- You only need to browse precomputed network disruptions or isoform switching events — CanIsoNet already serves these interactively and no recomputation is needed.

## Inputs

- Human STRING protein-protein interaction network edge list with combined interaction scores (e.g. 9606.protein.links files from the STRING database)
- Protein identifier for the target (HSPH1), resolved to its STRING/ENSP ID
- Reported metric value to validate against (network density score of 78% from Fig. 1C, Kahraman et al., Scientific Reports 2020)

## Outputs

- Computed network density score for the target protein (percentage)
- Weighted interaction sums for all proteins in the human STRING network
- Side-by-side comparison table of computed vs. reported values (e.g. hsph1_network_density.csv)

## How to apply

Download the human STRING protein-protein interaction network with combined interaction scores (the weighted edges CanIsoNet displays isoform switching disruptions on). For the protein of interest (HSPH1), extract all incident interactions and sum their combined interaction weights to obtain its raw weighted interaction sum; compute the identical sum for every other protein in the network. Normalize the protein's sum by the network-wide maximum and express it as a percentage — this is the network density score. Compare the computed percentage against the reported value (HSPH1: 78%, Fig. 1C of Kahraman et al. 2020) within a ±1% tolerance, and write computed and reported values side by side (e.g. to hsph1_network_density.csv). The rationale is that normalizing by the maximum places every protein on a 0–100 percentile-like scale so local interaction richness is comparable across the whole human STRING network.

## Related tools

- **STRING** (Source of the human protein-protein interaction network with combined interaction scores used to compute weighted interaction sums and the network density score)
- **CanIsoNet** (Webserver whose underlying methodology (Kahraman et al. 2020) defines the network density score; used to cross-check reported values and view isoform switching disruptions on the STRING network) — https://github.com/kahramanlab/CanIsoNet_Web

## Examples

```
python -c "import pandas as pd; L=pd.read_csv('9606.protein.links.v11.5.txt',sep=' '); s=pd.concat([L.groupby('protein1').combined_score.sum(),L.groupby('protein2').combined_score.sum()]).groupby(level=0).sum(); print(100*s['<HSPH1_ENSP>']/s.max())"
```

## Evaluation signals

- Computed network density score for HSPH1 equals the reported 78% within ±1% numerical tolerance
- The network-wide maximum weighted interaction sum is reproduced consistently when recomputed from the same STRING release (identical edge set and combined_score column)
- All computed density scores fall in the 0–100% range by construction (normalization by the maximum)
- Output CSV (hsph1_network_density.csv) contains computed and reported values side by side and both are populated
- The recomputation uses the same STRING release / combined interaction scores that Kahraman et al. and CanIsoNet rely on, not a newer or filtered network version

## Limitations

- The score is defined relative to the maximum weighted interaction sum in the human STRING network, so results are sensitive to the STRING version/release used and will shift as interaction scores are updated.
- The metric is a normalized weighted interaction sum, not a percentile of degree, so naive reimplementations using unweighted degree counts will not reproduce the reported 78%.
- The article text and README do not document the exact numeric procedure for the density score; the workflow relies on the definition given in Kahraman et al., Scientific Reports 2020 (Fig. 1C).

## Evidence

- [other] HSPH1 network density of 78% (task finding): [evidence span withheld — anchor status: not_found]
- [intro] STRING as CanIsoNet's network layer: "the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network"
- [readme] CanIsoNet scope statement (README): "It holds annotations for 11,528 isoforms and 14,357 unique isoform switching events across 31 diseases."
- [abstract] CanIsoNet combines STRING networks with isoform expression and ClinVar: "CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events"
