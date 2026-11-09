---
name: drug-target-gene-ranking-and-downregulation-validation
description: Use when a drug's direct target is known (or a drug-responsive cell type
  has been ranked) and you must prioritize additional candidate target genes from
  the cell-type-specific GRN and confirm them against treated-vs-untreated expression
  data.
when_to_use_negative:
- The drug acts as an agonist or has multiple/unknown targets requiring tailored multi-target
  methodology (scRank's agonist handling and multi-target extension are preliminary
  only).
- You only have disease-treatment paired data and want simple DEG/separability-based
  response analysis rather than GRN-based in silico perturbation of untreated samples.
- Genes already rank highly by simple correlation analysis is sufficient — the paper
  notes these top targets were not identifiable through simple correlation.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scRank
  role: 'Core method: builds the target-perturbed GRN, ranks drug-responsive cell
    types, and supports gene-level ranking of candidate targets within the top cell
    type''s GRN'
  repo: https://github.com/ZJUFanLab/scRank
- name: R
  role: Runtime environment for the scRank workflow and downstream validation/statistics
- name: devtools
  role: Used to install scRank from GitHub (devtools::install_github)
- name: rTensor
  role: Dependency required for scRank installation
  repo: https://github.com/rikenbit/rTensor
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:ddfc6acc1fc59d8addae913c6e108d5ea01df92719a70ed730fe27599fa7647e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - devtools
  - rTensor
  - scRank
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# drug-target-gene-ranking-and-downregulation-validation

## Summary

Ranks candidate drug-target genes by scRank perturbation scores within a top-ranked cell-type-specific GRN (e.g., tanshinone IIA targets in the M0-5 macrophage GRN beyond the known direct target Ctsk), then validates targets by significant post-treatment downregulation.

## When to use

Use when a drug's direct target is known (or a drug-responsive cell type has been ranked) and you must prioritize additional candidate target genes from the cell-type-specific GRN and confirm them against treated-vs-untreated expression data.

## When NOT to use

- The drug acts as an agonist or has multiple/unknown targets requiring tailored multi-target methodology (scRank's agonist handling and multi-target extension are preliminary only).
- You only have disease-treatment paired data and want simple DEG/separability-based response analysis rather than GRN-based in silico perturbation of untreated samples.
- Genes already rank highly by simple correlation analysis is sufficient — the paper notes these top targets were not identifiable through simple correlation.

## Inputs

- Single-cell expression matrix (genes x cells) from untreated diseased tissue, e.g., GSE163465 cardiac immune single-cell data
- Cell metadata with a cell-type annotation column (e.g., M0-5 macrophage subtype labels)
- Known drug target gene symbol (e.g., Ctsk for tanshinone IIA) or drug name resolvable via utile_database
- Treated vs untreated expression data for downstream target-gene downregulation testing

## Outputs

- Cell-type perturbation ranking (obj@cell_type_rank)
- Ranked candidate target gene table (top five: Ctsb, Ctsd, Fcgr1, Lgmn, Cd68)
- scranks_ctsk_celltype_ranking.csv and scrank_tanshinone_IIA_targets.csv output files

## How to apply

Build an S4 scRank object with CreateScRank(input=<expression matrix>, meta=<cell metadata>, cell_type=<annotation column>, species='mouse', target='Ctsk'), supplying the target gene directly when the drug-to-target mapping is known (as for tanshinone IIA). Reconstruct the cell-type-specific gene regulatory network with Constr_net(obj), then score in silico target perturbation across all cell types using rank_celltype(obj) and read obj@cell_type_rank to confirm the expected top population (M0-5 macrophage subtype in the GSE163465 myocardial infarction dataset). Restrict analysis to the top-ranked cell type's GRN and run scRank's gene-ranking mode to order candidate targets by perturbation effect, extracting the top five genes (Ctsb, Ctsd, Fcgr1, Lgmn, Cd68 in the reference run). Validate each recovered target by testing for statistically significant downregulation in treated versus untreated macrophages, since ranking alone does not establish direction of effect. Write the cell-type ranking and ranked target table to CSV (scrank_ctsk_celltype_ranking.csv, scrank_tanshinone_IIA_targets.csv) for reproducibility.

## Related tools

- **scRank** (Core method: builds the target-perturbed GRN, ranks drug-responsive cell types, and supports gene-level ranking of candidate targets within the top cell type's GRN) — https://github.com/ZJUFanLab/scRank
- **R** (Runtime environment for the scRank workflow and downstream validation/statistics)
- **devtools** (Used to install scRank from GitHub (devtools::install_github))
- **rTensor** (Dependency required for scRank installation) — https://github.com/rikenbit/rTensor

## Examples

```
library(scRank); obj <- CreateScRank(input=expr, meta=meta, cell_type='celltype', species='mouse', target='Ctsk'); obj <- Constr_net(obj); obj <- rank_celltype(obj)
```

## Evaluation signals

- M0-5 macrophage subtype appears as the top-ranked cell type in obj@cell_type_rank after rank_celltype().
- The top-five gene-ranking output matches the reported signature Ctsb, Ctsd, Fcgr1, Lgmn, Cd68.
- Each of the five candidate targets shows statistically significant downregulation in treated versus untreated macrophages.
- The targets are not recoverable by simple correlation analysis, providing a discriminative check that GRN-based ranking (not correlation) is being used.
- Both CSV outputs (cell-type ranking and ranked target table) are written with the expected schemas.

## Limitations

- scRank primarily focuses on drugs acting as inhibitors; agonist modeling uses a simplifying assumption (setting out-edge weights of the target gene node to maximum value) and validation is preliminary.
- Multi-target perturbation requires specifically tailored methodologies not yet developed in scRank.
- Validation of tpGRN simulation against GRNs from treated cells and robustness across clustering methodologies are reported only in supplementary figures (S14, S15).
- Gene-level ranking identifies candidate targets but requires independent expression-based validation (downregulation testing) to confirm direction of effect.

## Evidence

- [results] gene-ranking mode yields top targets: [evidence span withheld — anchor status: not_found]
