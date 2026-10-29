---
name: macrophage-subtype-prioritization-in-myocardial-infarction
description: Use when ranking diseased cell subpopulations by responsiveness to a
  drug from untreated scRNA-seq data — e.g. identifying the tanshinone IIA-responsive
  macrophage M∅-5 subpopulation in myocardial infarction — and prioritizing target
  genes beyond the known direct target.
when_to_use_negative:
- When the drug acts as an agonist and no agonist-mode refinement is available — scRank's
  agonist handling relies on a simplifying assumption (setting target-gene out-edge
  weights to maximum value) and is preliminary
- When the drug has multiple interacting targets — multi-target perturbation requires
  specifically tailored methodologies that are not yet developed in scRank
- When the analysis goal is simply differential-expression testing between treated
  and untreated cells; this skill is for perturbation-based ranking from untreated
  data using prior target knowledge
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scRank
  role: 'Core method: builds the target-perturbed GRN (tpGRN) and ranks drug-responsive
    cell types and candidate target genes via in silico perturbation (CreateScRank,
    Constr_net, rank_celltype)'
  repo: https://github.com/ZJUFanLab/scRank
- name: rTensor
  role: R dependency installed alongside scRank for tensor operations used in GRN
    reconstruction
  repo: https://github.com/rikenbit/rTensor
- name: devtools
  role: Used to install scRank from GitHub via devtools::install_github
  repo: https://github.com/rikenbit/rTensor
- name: tensorly
  role: Python module integrated in the Constr_net() function for gene regulatory
    network reconstruction
- name: R
  role: Runtime environment (R >4.0) for the scRank workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:be62eafc3f2144a2eff51b7643c7bd5902166eaf2a2b76afbb6c3af0870e8211
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - devtools
  - rTensor
  - scRank
  - tensorly
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# macrophage-subtype-prioritization-in-myocardial-infarction

## Summary

Prioritizes drug-responsive cell subpopulations and candidate drug targets from untreated scRNA-seq data by scoring in silico perturbation of a known drug target (e.g. Ctsk for tanshinone IIA) within cell-type-specific target-perturbed gene regulatory networks (tpGRNs), using scRank.

## When to use

Use when ranking diseased cell subpopulations by responsiveness to a drug from untreated scRNA-seq data — e.g. identifying the tanshinone IIA-responsive macrophage M∅-5 subpopulation in myocardial infarction — and prioritizing target genes beyond the known direct target.

## When NOT to use

- When the drug acts as an agonist and no agonist-mode refinement is available — scRank's agonist handling relies on a simplifying assumption (setting target-gene out-edge weights to maximum value) and is preliminary
- When the drug has multiple interacting targets — multi-target perturbation requires specifically tailored methodologies that are not yet developed in scRank
- When the analysis goal is simply differential-expression testing between treated and untreated cells; this skill is for perturbation-based ranking from untreated data using prior target knowledge

## Inputs

- Untreated diseased single-cell RNA-seq expression matrix (genes x cells), e.g. GSE163465 myocardial infarction cardiac immune dataset
- Cell metadata table with a cell-type/subpopulation annotation column (e.g. macrophage subtypes such as M0-5)
- Known drug target gene name (e.g. Ctsk for tanshinone IIA) or drug name resolvable via the utile_database
- Species specification (e.g. 'mouse')

## Outputs

- Cell-type perturbation ranking in obj@cell_type_rank identifying the top-ranked drug-responsive subpopulation (e.g. M0-5 macrophages)
- Ranked target-gene table from the gene-ranking mode on the top-ranked GRN (e.g. Ctsb, Ctsd, Fcgr1, Lgmn, Cd68)
- CSV output files such as scrank_ctsk_celltype_ranking.csv and scrank_tanshinone_IIA_targets.csv
- Optional network/dimension-reduction visualizations via plot_net and plot_dim, and GSEA of drug effects via scRank_GSEA and plot_drug_function

## How to apply

Load the untreated diseased scRNA-seq expression matrix and metadata (e.g. the GSE163465 myocardial infarction cardiac immune dataset) into R and build an S4 object with CreateScRank(input=<expression>, meta=<metadata>, cell_type=<column>, species='mouse', target='Ctsk'), supplying the target gene directly when the drug's target is known (no drug name needed). Reconstruct the cell-type-specific GRN from expression profiles with Constr_net(obj), which integrates the Python module 'tensorly' and depends on rTensor for tensor operations. Score in silico drug perturbation of the target-gene node across all cell types with rank_celltype(obj) and read rankings from obj@cell_type_rank; drug perturbation is modeled as deletion of the target gene node in the tpGRN with both global and local effects via network diffusion. To find candidate targets for a natural product like tanshinone IIA, restrict to the top-ranked subpopulation's GRN (M∅-5) and run scRank's gene-ranking mode, which is effective for drugs whose mechanisms of action are unknown. Finally, validate by checking that targets (e.g. Ctsb, Ctsd, Fcgr1, Lgmn, Cd68) are significantly downregulated in treated versus untreated macrophages, and write rankings to CSV.

## Related tools

- **scRank** (Core method: builds the target-perturbed GRN (tpGRN) and ranks drug-responsive cell types and candidate target genes via in silico perturbation (CreateScRank, Constr_net, rank_celltype)) — https://github.com/ZJUFanLab/scRank
- **rTensor** (R dependency installed alongside scRank for tensor operations used in GRN reconstruction) — https://github.com/rikenbit/rTensor
- **devtools** (Used to install scRank from GitHub via devtools::install_github) — https://github.com/rikenbit/rTensor
- **tensorly** (Python module integrated in the Constr_net() function for gene regulatory network reconstruction)
- **R** (Runtime environment (R >4.0) for the scRank workflow)

## Examples

```
obj <- CreateScRank(input=expr, meta=metadata, cell_type='celltype', species='mouse', target='Ctsk'); Constr_net(obj); rank_celltype(obj); obj@cell_type_rank
```

## Evaluation signals

- The known drug-target-sensitive subpopulation (M0-5 macrophages) is the top-ranked cell type in obj@cell_type_rank after rank_celltype
- The top five genes ranked by perturbation score in the M0-5 GRN match the expected signature: Ctsb, Ctsd, Fcgr1, Lgmn, Cd68
- The recovered target genes were not identifiable through simple correlation analysis, confirming the ranking adds signal beyond correlation
- The top-five targets show statistically significant downregulation in tanshinone IIA-treated versus untreated macrophages
- Both output CSVs (scrank_ctsk_celltype_ranking.csv and scrank_tanshinone_IIA_targets.csv) are written with complete rankings

## Limitations

- scRank primarily focuses on drugs acting as inhibitors, partly due to the nature of available data; agonist tests are preliminary only
- Agonist perturbation modeling uses a simplifying assumption (setting out-edge weights of the target gene node to their maximum possible value)
- Multi-target drug perturbation requires specifically tailored methodologies not yet developed; only preliminary results exist
- Validation of tpGRN simulation against GRNs from treated cells and adaptability across clustering methodologies are only shown in supplementary figures (S14, S15)

## Evidence
