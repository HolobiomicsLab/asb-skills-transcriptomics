---
name: regulon-membership-validation
description: 'Use when a differential network (e.g. from BoostDiff on salt/heat/ethanol
  stress vs. control) needs biological validation: check whether top hub genes belong
  to a known regulon such as SigB.'
when_to_use_negative:
- The condition under study has no curated transcription-factor regulon against which
  hub genes can be checked — the validation relies on prior regulon knowledge and
  cannot be applied blind.
- The network is still the raw BoostDiff output before filter_network filtering (p=3,
  n_top_edges=150) — unfiltered edges include genes where BoostDiff did not find a
  more predictive model for the target condition, so hub ranking would be misleading.
- The goal is differential expression testing rather than network-structure validation
  — standard differential expression analysis is a complementary but distinct check.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: BoostDiff
  role: Produces the differential networks whose filtered edge lists are mined for
    hub genes to validate against the SigB regulon; uses differential variance improvement
    as splitting criterion
  repo: https://github.com/scibiome/boostdiff_inference
- name: Python
  role: Used to compute hub genes from the filtered differential edge lists and write
    per-condition CSV hub tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:0bb4be506ff0ae25b1456fe2c487cc60304be404fa656344b848560c0d51c47d
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
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# regulon-membership-validation

## Summary

Validates a condition-specific differential network by checking whether its top-ranked hub genes are members of a known transcription-factor regulon (here, the B. subtilis SigB stress-response regulon), turning biological prior knowledge into a correctness signal for network inference methods like BoostDiff.

## When to use

Use when a differential network (e.g. from BoostDiff on salt/heat/ethanol stress vs. control) needs biological validation: check whether top hub genes belong to a known regulon such as SigB.

## When NOT to use

- The condition under study has no curated transcription-factor regulon against which hub genes can be checked — the validation relies on prior regulon knowledge and cannot be applied blind.
- The network is still the raw BoostDiff output before filter_network filtering (p=3, n_top_edges=150) — unfiltered edges include genes where BoostDiff did not find a more predictive model for the target condition, so hub ranking would be misleading.
- The goal is differential expression testing rather than network-structure validation — standard differential expression analysis is a complementary but distinct check.

## Inputs

- Filtered BoostDiff differential edge lists per condition (e.g. top 300 edges total: 150 per run with disease as target and control as target)
- Known regulon membership list (e.g. B. subtilis SigB regulon member gene names such as csbD, ohrB, yuzA, yycD)
- Per-condition hub-gene CSV tables computed from the filtered networks (salt, heat, ethanol stress contexts from GSE27219)

## Outputs

- Ranked top-ten hub gene list per stress condition (e.g. csbD, S536, ohrB, yuzA, ywjC, S1199, S1379, yfhK, yycD, ywmE, ybyB for salt stress)
- Per-condition CSV validation table recording which top hub genes are SigB regulon members and the overlap count

## How to apply

After running BoostDiff (e.g. salt vs smm with keyword 'saltvssmm', n_estimators=50, n_features=2500, n_subsamples=20, loss='exponential', regulators='all') and filtering each raw network with filter_network (p=3, n_top_edges=150 per sub-analysis, keeping only target genes with a more predictive model for the target condition), compute hub genes from the filtered differential edge lists — genes with the highest edge degree — and write per-condition CSV hub tables. Then check regulon membership: for the GSE27219 B. subtilis stress contexts, verify that the top-ten hub genes in each stress condition (e.g. csbD, ohrB, yuzA, yycD, S536, ywjC, S1199, yfhK, ywmE for salt and heat) are members of the SigB regulon. The rationale is that BoostDiff's differential variance improvement criterion should elevate genes whose condition-specific co-expression rewiring is driven by the known master regulator, so recovery of SigB regulon members as hubs confirms the network captures the intended biology rather than artifacts.

## Related tools

- **BoostDiff** (Produces the differential networks whose filtered edge lists are mined for hub genes to validate against the SigB regulon; uses differential variance improvement as splitting criterion) — https://github.com/scibiome/boostdiff_inference
- **Python** (Used to compute hub genes from the filtered differential edge lists and write per-condition CSV hub tables)

## Examples

```
python -c "import pandas as pd; edges=pd.read_csv('df_dis.txt',sep='\t'); hubs=edges['Target'].value_counts().head(10); print(hubs)"  # then check top hubs against SigB regulon list
```

## Evaluation signals

- All (or nearly all) of the top-ten hub genes in the salt-stress differential network (csbD, S536, ohrB, yuzA, ywjC, S1199, S1379, yfhK, yycD, ywmE, ybyB) are SigB regulon members.
- The top hubs for heat stress (csbD, ohrB, S536, yuzA, S1199, yycD, ywjC, yfhK, ywmE, nhaX) are likewise SigB regulon members, matching the reported result.
- Hub tables are computed only after filter_network with p=3 and n_top_edges=150 per sub-analysis (top 300 total), keeping only target genes with a more predictive model for the target condition.
- Hub recovery is consistent across the stress contexts (salt, heat, ethanol) since all target the SigB-mediated general stress response.
- Per-condition CSV hub tables exist with gene names resolvable against the SigB regulon membership list (no unmapped gene IDs).

## Limitations

- Validation is regulon-dependent: it succeeds here because B. subtilis stress response is dominated by the well-characterized SigB regulon; conditions without a dominant, well-curated regulon may lack an equivalent gold standard.
- BoostDiff input quality depends on sample size and balancing: the GSE27219 workflow used SMOTE oversampling to 48 samples per condition and a variance cutoff of 0.4, and the README recommends at least 30 samples per condition; small or unbalanced inputs may degrade hub recovery.
- BoostDiff is memory-intensive at scale — the Crohn's disease dataset was too large for the demo and required a high-performance computing cluster.

## Evidence

- [results] For the salt stress experiment, all of the top ten hub genes in the BoostDiff differential network: [evidence span withheld — anchor status: not_found]
- [methods] filter_network raw network filtering parameters: "df_dis = filter_network(file_net_dis, file_diff_dis, p=3, n_top_edges=150)"
- [methods] raw network filtered for target genes: "the raw network should be filtered for target genes in which BoostDiff found a more predictive model for the target condition"
- [abstract] BoostDiff identified SigB-enriched context-specific networks on B. subtilis stress data: "BoostDiff identifies context-specific networks that are enriched with genes of known disease-relevant pathways and complements standard differential expression analyses"
- [readme] B. subtilis datasets in GEO accession GSE27219: "The *B. subtilis* datasets can be found in GEO with the accession number GSE27219."
