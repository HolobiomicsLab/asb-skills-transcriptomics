---
name: hub-gene-identification-from-networks
description: Use when a filtered differential network (e.g. BoostDiff output with
  p=3, n_top_edges=150 per sub-analysis) is available and top hub genes must be ranked
  per condition and checked against a known regulon or pathway.
when_to_use_negative:
- The network is a raw (unfiltered) BoostDiff output — hubs computed from it would
  include edges not predictive for the target condition; filter first with filter_network.
- The goal is differential expression ranking rather than network topology — BoostDiff
  complements, but does not replace, standard differential expression analyses.
- There is no reference regulon or pathway to validate hubs against, and only a qualitative
  sanity check is needed.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: BoostDiff
  role: Produces the raw differential networks whose filtered edge lists are the basis
    for hub-gene computation
  repo: https://github.com/scibiome/boostdiff_inference
- name: Python
  role: Used to compute hub genes from the filtered differential edge lists and write
    per-condition CSV tables
- name: pySMOTE
  role: Oversamples the minority stress condition (to 48 samples per condition) before
    network inference so hub rankings are computed on balanced classes
  repo: https://github.com/akazs/pySMOTE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:8fc6fe9c8e0ecc4b2a57d28f0a3dcd7c181b9ff8cc4af1787780acc5e88a9050
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/boostdiff_inference
  tools:
  - BoostDiff
  - GEOquery
  - Python
  - R
  - dplyr
  - pySMOTE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hub-gene-identification-from-networks

## Summary

Compute hub genes (highly connected nodes) from BoostDiff-filtered differential edge lists and validate them against known regulons, e.g. confirming that the top-ten hubs in B. subtilis salt/heat stress networks are SigB regulon members.

## When to use

Use when a filtered differential network (e.g. BoostDiff output with p=3, n_top_edges=150 per sub-analysis) is available and top hub genes must be ranked per condition and checked against a known regulon or pathway.

## When NOT to use

- The network is a raw (unfiltered) BoostDiff output — hubs computed from it would include edges not predictive for the target condition; filter first with filter_network.
- The goal is differential expression ranking rather than network topology — BoostDiff complements, but does not replace, standard differential expression analyses.
- There is no reference regulon or pathway to validate hubs against, and only a qualitative sanity check is needed.

## Inputs

- Filtered differential edge list per condition (BoostDiff raw network filtered with filter_network, p=3, n_top_edges=150 per sub-analysis, top 300 total)
- Target-gene lists indicating which genes BoostDiff found more predictive for the target condition
- Reference regulon membership (e.g. SigB regulon member list for B. subtilis)

## Outputs

- Per-condition hub-gene ranked CSV tables (e.g. hub genes for saltvssmm, heatvssmm, ethavssmm)
- Validation report mapping top-ten hubs to SigB regulon membership

## How to apply

Run BoostDiff twice per context (disease as target, then control as target) and filter each raw network with filter_network(p=3, n_top_edges=150), keeping only target genes with a more predictive model for the target condition (top 300 total edges per comparison). Compute node degree/edge counts from the filtered differential edge lists and rank genes to produce per-condition hub-gene tables (e.g. per-condition CSVs for salt, heat, ethanol from GSE27219). Judge correctness by checking whether top-ranked hubs are known regulon members: for salt stress the top ten hubs (csbD, S536, ohrB, yuzA, ywjC, S1199, S1379, yfhK, yycD, ywmE, ybyB) should all belong to the SigB regulon, and heat-stress hubs (csbD, ohrB, S536, yuzA, S1199, yycD, ywjC, yfhK, ywmE, nhaX) should likewise be SigB members.

## Related tools

- **BoostDiff** (Produces the raw differential networks whose filtered edge lists are the basis for hub-gene computation) — https://github.com/scibiome/boostdiff_inference
- **Python** (Used to compute hub genes from the filtered differential edge lists and write per-condition CSV tables)
- **pySMOTE** (Oversamples the minority stress condition (to 48 samples per condition) before network inference so hub rankings are computed on balanced classes) — https://github.com/akazs/pySMOTE

## Examples

```
df_dis = filter_network(file_net_dis, file_diff_dis, p=3, n_top_edges=150)
```

## Evaluation signals

- All top-ten hub genes for the salt stress condition (csbD, S536, ohrB, yuzA, ywjC, S1199, S1379, yfhK, yycD, ywmE, ybyB) are SigB regulon members.
- Top hubs for heat stress (csbD, ohrB, S536, yuzA, S1199, yycD, ywjC, yfhK, ywmE, nhaX) are also SigB regulon members.
- Hub tables are derived from networks filtered with filter_network p=3 and n_top_edges=150 per sub-analysis (top 300 total edges).
- Filtered networks retain only target genes for which BoostDiff found a more predictive model for the target condition.
- A per-condition hub-gene CSV is written for each of the three stress-vs-control contexts.

## Limitations

- Hub recovery depends on upstream parameters (SMOTE oversampling to 48 samples per condition, variance cutoff 0.4, n_estimators=50, n_features=2500, n_subsamples=20, loss='exponential'); changing these can alter which genes become hubs.
- Validation is regulon-dependent — recovery of SigB members is demonstrated for B. subtilis stress contexts and may not generalize to conditions without a well-defined regulon.
- BoostDiff works best with at least ~30 samples per condition; smaller datasets may yield unstable hub rankings.

## Evidence

- [results] task_007 salt/heat hub finding: [evidence span withheld — anchor status: not_found]
- [methods] filter step: "df_dis = filter_network(file_net_dis, file_diff_dis, p=3, n_top_edges=150)"
- [methods] filter rationale: "the raw network should be filtered for target genes in which BoostDiff found a more predictive model for the target condition"
- [readme] input format: "Each input is a tab-separated file. The rows correspond to genes, while the columns correspond to samples. Note that the two inputs should have the same set of features, where the first column should"
