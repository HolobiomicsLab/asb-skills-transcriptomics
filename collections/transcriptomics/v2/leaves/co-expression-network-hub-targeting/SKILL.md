---
name: co-expression-network-hub-targeting
description: Use when a scRNA-seq dataset has been decomposed into co-expression gene
  modules (e.g. WGCNA-style modules like M4 or M11) and the goal is to prioritize
  hub genes inside a disease/trait-associated module as drug targets.
when_to_use_negative:
- The question is differential expression or cell-type annotation, not network-level
  target prioritization — SCANet's drug repurposing step operates on GCN/GRN modules,
  not gene lists alone
- 'You need causal regulatory evidence: GCNs rely on correlation, not causation, so
  hub status alone does not establish a causal driver'
- Reproducibility across time is critical and the exact Drugst.One database version
  cannot be fixed, since gene–drug databases update weekly
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0602
tools:
- name: SCANet
  role: Python package performing module inference (modified WGCNA), hub detection
    (top 10% most connected), and the drug repurposing function linking GCN hubs to
    drugs
  repo: https://github.com/oubounyt/SCANet
- name: Drugst.One
  role: Maps network proteins to primary identifiers (UniProt, Entrez/Ensembl) and
    provides the drug–target interaction query interface used by SCANet's repurposing
    step
- name: NeDRex
  role: Default drug–target database queried by SCANet, merging DrugBank and DrugCentral
    entries for small molecules, biologics, approved and experimental drugs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:c8745d57d985001c86a08e3d2acc90bef1135783c6f2de62c489f3c4aef421d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - DrugBank
  - DrugCentral
  - Drugst.One
  - NeDRex
  - SCANet
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# co-expression-network-hub-targeting

## Summary

Identifies hub genes (top 10% most connected) within trait/cell-type-specific co-expression modules inferred from scRNA-seq data, then maps those hubs to pharmacological targets for mechanistic drug repurposing via SCANet's Drugst.One/NeDRex integration.

## When to use

Use when a scRNA-seq dataset has been decomposed into co-expression gene modules (e.g. WGCNA-style modules like M4 or M11) and the goal is to prioritize hub genes inside a disease/trait-associated module as drug targets.

## When NOT to use

- The question is differential expression or cell-type annotation, not network-level target prioritization — SCANet's drug repurposing step operates on GCN/GRN modules, not gene lists alone
- You need causal regulatory evidence: GCNs rely on correlation, not causation, so hub status alone does not establish a causal driver
- Reproducibility across time is critical and the exact Drugst.One database version cannot be fixed, since gene–drug databases update weekly

## Inputs

- Processed scRNA-seq count matrix (e.g. GSE180578 Cillo covid-19 processed data or Aliluev et al. 2021 diet-induced obese mouse small intestine data)
- Selected trait/cell-type-associated co-expression module (e.g. M4 ISC module from HFHSD mice)
- Hub gene list (top 10% most connected genes of the module GCN)
- Drug–target interaction databases accessed via Drugst.One (NeDRex, merging DrugBank and DrugCentral)

## Outputs

- Drug–target interaction table (e.g. drug_repurposing_M4_ISC.csv) listing repurposing candidates per hub/target
- Set of pharmacological targets derived from module hubs (20 drugs for 8 targets in the M4 ISC case)
- Hub gene intersection within the module GCN at the chosen correlation threshold (≥ 0.8 for GCN–drug interactions in M4)

## How to apply

First infer co-expression modules with SCANet, which builds on a modified version of WGCNA on pseudo-bulk-aggregated single-cell counts, and select the module associated with the trait/cell type of interest using the 'Modules selection guidelines' in Supplementary File 1. Within that module, identify network hubs by intersecting the top 10% of the most connected genes in the module graph, optionally verifying the module fits scale-free topology. Restrict the GCN used for drug interaction querying to edges with high gene correlation — for the M4 intestinal stem cell (ISC) module of HFHSD obese mice the threshold was correlation ≥ 0.8. Run SCANet's drug repurposing function, which maps network proteins to primary identifiers (UniProt, plus Entrez/Ensembl) through Drugst.One and queries the default NeDRex database (merging DrugBank and DrugCentral entries for small molecules, biologics, approved and experimental drugs). The output is a drug–target interaction table, e.g. drug_repurposing_M4_ISC.csv, which in the paper's obese-mouse ISC analysis yielded 20 drugs for eight targets including cholesterol/lipid-lowering drugs (lovastatin, atorvastatin, pravastatin, fenofibrate, lomitapide, ezetimibe) and anti-hypertensives (methyldopa, furosemide, warfarin).

## Related tools

- **SCANet** (Python package performing module inference (modified WGCNA), hub detection (top 10% most connected), and the drug repurposing function linking GCN hubs to drugs) — https://github.com/oubounyt/SCANet
- **Drugst.One** (Maps network proteins to primary identifiers (UniProt, Entrez/Ensembl) and provides the drug–target interaction query interface used by SCANet's repurposing step)
- **NeDRex** (Default drug–target database queried by SCANet, merging DrugBank and DrugCentral entries for small molecules, biologics, approved and experimental drugs)

## Examples

```
pip install scanet; import scanet as sn
```

## Evaluation signals

- The output drug–target table contains the expected entry count — 20 drug entries spanning eight distinct pharmacological targets, matching the article abstract for the ISC module of obese mice
- Recovered drug classes match the reported finding: cholesterol/lipid-lowering drugs (lovastatin, fenofibrate, lomitapide, pravastatin, atorvastatin) and anti-hypertensive drugs (methyldopa, furosemide, warfarin)
- Hub set equals the intersection of the top 10% of the most connected genes in the module, and GCN–drug candidates are filtered at gene correlation ≥ 0.8 (the M4 ISC threshold)
- Every network protein is mapped to a primary identifier (UniProt ID, plus Entrez/Ensembl) before the NeDRex query succeeds
- Result CSV (e.g. drug_repurposing_M4_ISC.csv) is written and each row resolves to a drug–target pair present in NeDRex (DrugBank/DrugCentral merge)

## Limitations

- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis
- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected
- Gene–drug databases in Drugst.One update weekly, impacting reproducibility; running identical analyses and parameters on newer datasets may yield different results
- A version control system for Drugst.One is still in development and not yet available, so result sets may drift across database versions

## Evidence

- [abstract] we find 20 drugs for eight potential pharmacological targets in cellular driver mechanisms in the intestinal stem cells of obese mice: "we find 20 drugs for eight potential pharmacological targets in cellular driver mechanisms in the intestinal stem cells of obese mice"
- [methods] users can identify network hubs by looking for the intersection of the top 10% of the most connected genes in the module: "users can identify network hubs by looking for the intersection of the top 10% of the most connected genes in the module"
- [results] the same module was selected for GCN–drug interactions (correlation ≥ 0.8) (Fig. 4D): "the same module was selected for GCN–drug interactions (correlation ≥ 0.8) (Fig. 4D)"
- [methods] SCANet uses by default the database "NeDRex," which is an extensive dataset encompassing small molecules, biologics, approved drugs, and experimental ones: [evidence span withheld — anchor status: not_found]
- [discussion] gene–drug databases in Drugst.One update weekly, impacting reproducibility.: "gene–drug databases in Drugst.One update weekly, impacting reproducibility. Although old results are accessible, running identical analyses"
- [readme] a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data: "`SCANet`, a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data"
