---
name: gene-regulatory-network-perturbation-scoring
description: Use when untreated scRNA-seq data from diseased tissue has a known drug
  target (e.g. Brd4 for BET inhibitors) and you must rank which cell types are most
  drug-responsive via in silico target perturbation of GRNs.
when_to_use_negative:
- The drug acts as an agonist — scRank primarily focuses on drugs acting as inhibitors,
  and agonist modeling (setting out-edge weights to their maximum value) is a preliminary
  simplifying assumption.
- The drug has multiple targets — multi-target perturbation requires specifically
  tailored methodologies not yet developed in scRank.
- Only treated-vs-untreated differential expression is available and drug target knowledge
  is not required — DEG-count or separability methods (e.g. Augur) fit that setting
  instead.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scRank
  role: R package providing rank_celltype() to build the tpGRN, compute manifold-alignment
    global distortion and 2-hop diffusion local effect, and score/rank cell types
  repo: https://github.com/ZJUFanLab/scRank
- name: R
  role: Runtime environment for scRank (R > 4.0) including devtools-based installation
- name: rTensor
  role: Dependency installed before scRank
  repo: https://github.com/rikenbit/rTensor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:0c18619372c1e1e0455c946b919b82b7e6217ff2010d52b24e32dd22d759373f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - rTensor
  - scRank
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gene-regulatory-network-perturbation-scoring

## Summary

Generates a target-perturbed gene regulatory network (tpGRN) from cell-type-specific GRNs and scores in silico drug perturbation by combining global manifold-alignment distortion with local n-hop network diffusion, ranking drug-responsive cell types in scRank.

## When to use

Use when untreated scRNA-seq data from diseased tissue has a known drug target (e.g. Brd4 for BET inhibitors) and you must rank which cell types are most drug-responsive via in silico target perturbation of GRNs.

## When NOT to use

- The drug acts as an agonist — scRank primarily focuses on drugs acting as inhibitors, and agonist modeling (setting out-edge weights to their maximum value) is a preliminary simplifying assumption.
- The drug has multiple targets — multi-target perturbation requires specifically tailored methodologies not yet developed in scRank.
- Only treated-vs-untreated differential expression is available and drug target knowledge is not required — DEG-count or separability methods (e.g. Augur) fit that setting instead.

## Inputs

- scRank S4 object created by CreateScRank with GRN reconstructed by Constr_net from expression profiles
- Assigned drug target gene (e.g. Brd4; targets discoverable via the utile_database)
- Untreated scRNA-seq count matrix split by cell type (demo dataset GSE110894)

## Outputs

- Per-cell-type perturbation scores and ranking in the obj@cell_type_rank slot
- Exported ranking table (e.g. cell_type_rank.csv)

## How to apply

Load the scRank S4 object carrying the GRN built by Constr_net (demo: GSE110894 BET inhibitor resistant vs sensitive leukaemic cells, target Brd4). Call rank_celltype(obj) to form the tpGRN by deleting the target gene node's outgoing edges (default type = antagonist), which models the drug (inhibitor) as a node deletion producing both global and local effects. The global effect is measured via manifold-alignment node distances between the untreated and target-perturbed networks; the local effect uses network diffusion over the neighborhood defined by the n_hop parameter (2-hop diffusion from the target gene). The two effects are combined into a single per-cell-type perturbation score stored in obj@cell_type_rank, which is then exported (e.g. to cell_type_rank.csv). Correct application is judged by a ranking in which the biologically expected drug-responsive cell type scores highest, validated against literature or treated-vs-untreated data.

## Related tools

- **scRank** (R package providing rank_celltype() to build the tpGRN, compute manifold-alignment global distortion and 2-hop diffusion local effect, and score/rank cell types) — https://github.com/ZJUFanLab/scRank
- **R** (Runtime environment for scRank (R > 4.0) including devtools-based installation)
- **rTensor** (Dependency installed before scRank) — https://github.com/rikenbit/rTensor

## Examples

```
devtools::install_github("ZJUFanLab/scRank"); rank_celltype(obj); write.csv(obj@cell_type_rank, "cell_type_rank.csv")
```

## Evaluation signals

- obj@cell_type_rank slot is populated and can be written to cell_type_rank.csv without errors
- The tpGRN equals the original GRN with the target gene node's outgoing edges removed (antagonist default)
- Top-ranked cell type matches literature-expected drug responsiveness (e.g. BET inhibitor-resistant leukaemic cells for Brd4 targeting)
- Validation against GRNs derived from drug-treated cells confirms the simulated tpGRN accurately captures drug effect (supplementary Figure S14)
- Ranking is robust across clustering methodologies (supplementary benchmark, Figure S15)

## Limitations

- scRank primarily focuses on drugs acting as inhibitors due to data availability; agonist tests are preliminary only.
- Agonist modeling uses a simplifying assumption of setting out-edge weights of the target gene node to their maximum possible value.
- Multi-target drugs need specifically tailored methodologies that are not yet developed (preliminary results only, Figure S17).
- Validation of tpGRN simulation against GRNs from treated cells appears only in supplementary material (Figure S14).

## Evidence
