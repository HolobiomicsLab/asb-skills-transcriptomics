---
name: drug-target-annotation-mapping
description: Use when applying scRank to a drug perturbation analysis and only a drug
  name (not a target gene) is available, or when deciding between antagonist (inhibitor)
  and agonist MOA settings for CreateScRank.
when_to_use_negative:
- The drug acts as an agonist and reliable single-cell agonist data are required —
  scRank's agonist modeling (maximal out-edge weights on the target node) is a simplifying
  assumption and only preliminary
- The drug has multiple targets requiring target-to-target interaction modeling —
  specifically tailored methodologies for multi-target perturbation are not yet developed
  beyond preliminary results (Figure S17)
- The analysis needs a standalone drug-target annotation tool — this skill only covers
  mapping within the scRank tpGRN workflow, not general pharmacological annotation
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: scRank
  role: Provides the `utile_database` for drug-name-to-target resolution and the MOA
    parameter of CreateScRank that sets the perturbation mode of the tpGRN
  repo: https://github.com/ZJUFanLab/scRank
- name: devtools
  role: Used to install scRank (and rTensor dependency) from GitHub so `utile_database`
    and CreateScRank are available in R
  repo: https://github.com/ZJUFanLab/scRank
- name: rTensor
  role: Dependency of scRank required before installation; supports the tensor-based
    GRN reconstruction underlying the target-perturbed network
  repo: https://github.com/rikenbit/rTensor
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:bb4f2e09744c8e3539f52480216b8b9ed8b2141082a20e79b026b875a08d684d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - Seurat
  - beyondcell
  - devtools
  - rTensor
  - scDEAL
  - scRank
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# drug-target-annotation-mapping

## Summary

Resolves drug names to their canonical targets and mechanism of action (MOA) within the scRank workflow, using the package's `utile_database` so that a target-perturbed gene regulatory network (tpGRN) can be constructed for the correct target gene with the appropriate antagonist (or agonist) perturbation mode.

## When to use

Use when applying scRank to a drug perturbation analysis and only a drug name (not a target gene) is available, or when deciding between antagonist (inhibitor) and agonist MOA settings for CreateScRank.

## When NOT to use

- The drug acts as an agonist and reliable single-cell agonist data are required — scRank's agonist modeling (maximal out-edge weights on the target node) is a simplifying assumption and only preliminary
- The drug has multiple targets requiring target-to-target interaction modeling — specifically tailored methodologies for multi-target perturbation are not yet developed beyond preliminary results (Figure S17)
- The analysis needs a standalone drug-target annotation tool — this skill only covers mapping within the scRank tpGRN workflow, not general pharmacological annotation

## Inputs

- Drug name (must be resolvable via scRank's `utile_database`) or a known drug target gene symbol
- Drug mechanism-of-action class (antagonist default; agonist supported preliminarily)
- Gene expression profile and cell type/line labels supplied to CreateScRank

## Outputs

- Resolved drug-target pairing and MOA used to parameterize the tpGRN perturbation node
- scRank S4 object (from CreateScRank) carrying the drug annotation for downstream Constr_net and rank_celltype steps

## How to apply

When calling CreateScRank for a cell-line or disease scRNA-seq dataset, supply either the known drug target gene directly, or pass the drug name and let scRank resolve it via the package's internal `utile_database`; keep the default antagonist MOA for inhibitor drugs since scRank's tpGRN models inhibitors as deletion of the drug target gene node in the GRN. The resolved target and MOA determine which gene node is perturbed during network reconstruction (Constr_net) and in silico perturbation scoring (rank_celltype), so a wrong target/MOA mapping invalidates the entire drug-responsive cell-type ranking. For natural-product drugs with unknown mechanisms of action (e.g., tanshinone IIA), use scRank's target-prioritization capability to identify and rank candidate targets before fixing the perturbation node. For agonist drugs, apply only the preliminary mode that sets the out-edge weights of the target gene node to their maximum value, and treat results as provisional.

## Related tools

- **scRank** (Provides the `utile_database` for drug-name-to-target resolution and the MOA parameter of CreateScRank that sets the perturbation mode of the tpGRN) — https://github.com/ZJUFanLab/scRank
- **devtools** (Used to install scRank (and rTensor dependency) from GitHub so `utile_database` and CreateScRank are available in R) — https://github.com/ZJUFanLab/scRank
- **rTensor** (Dependency of scRank required before installation; supports the tensor-based GRN reconstruction underlying the target-perturbed network) — https://github.com/rikenbit/rTensor

## Examples

```
devtools::install_github("ZJUFanLab/scRank")
```

## Evaluation signals

- The supplied drug name resolves in `utile_database` (CreateScRank does not error on the `drug` argument); if it does not, fall back to supplying the target gene directly
- For inhibitor drugs, MOA remains the default 'antagonist', matching the paper's modeling of inhibitors as deletion of the drug target gene node
- The resolved target gene corresponds to the biologically expected target (e.g., BRD4 for BET inhibitors in the GSE110894 leukaemia demo)
- Downstream rank_celltype output (obj@cell_type_rank) changes sensibly with the resolved target, and benchmark accuracy against known sensitive/responsive labels (e.g., scRank 77.8% vs beyondcell 72.2% and scDEAL 64.8% on cancer cell line datasets) is reproducible

## Limitations

- scRank primarily focuses on drugs acting as inhibitors (antagonists), partly due to the nature of available data; agonist support is preliminary
- Agonist perturbation is modeled only by a simplifying assumption (setting out-edge weights of the target gene node to their maximum possible value)
- Multi-target drugs require specifically tailored methodologies that are not yet developed; only preliminary extensions exist (Figure S17)

## Evidence
