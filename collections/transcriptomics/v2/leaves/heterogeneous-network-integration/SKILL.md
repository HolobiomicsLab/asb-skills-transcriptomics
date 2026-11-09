---
name: heterogeneous-network-integration
description: Use when you have an inferred gene network (e.g. a co-expression module
  GCN or a TF–TG GRN) and need to connect its genes/proteins to drugs to propose mechanistic
  repurposing candidates.
when_to_use_negative:
- When the goal is co-expression module inference or TF–TG GRN reconstruction rather
  than drug linking — use SCANet's upstream GCN/GRN inference steps instead
- When reproducibility across time is critical and a pinned gene–drug database version
  is required — Drugst.One's gene–drug databases update weekly and a version control
  system is still in development
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_0202
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3308
tools:
- name: SCANet
  role: Provides the drug repurposing function that takes module network genes and
    builds the heterogeneous gene–drug network
  repo: https://github.com/oubounyt/SCANet
- name: Drugst.One
  role: Maps gene identifiers to a primary common identifier (UniProt, Entrez, Ensembl)
    and connects the human interactome to drug–target databases
- name: NeDRex
  role: Default drug–target database queried by SCANet; merges DrugBank and DrugCentral
- name: DrugBank
  role: Component drug–target source within NeDRex
- name: DrugCentral
  role: Component drug–target source within NeDRex, also available as a standalone
    alternative database in SCANet
- name: ChEMBL
  role: Alternative drug–target interaction source supported by SCANet
- name: DGIdb
  role: Alternative drug–target interaction source supported by SCANet
- name: Python
  role: Implementation language; SCANet is a Python package installed via pip
  repo: https://pypi.org/project/scanet/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:ed2c76f806a0120874d19afac2326e635430d9337967c8df08c46b8def709d82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - ChEMBL
  - DGIdb
  - DrugBank
  - DrugCentral
  - Drugst.One
  - NeDRex
  - Python
  - SCANet
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_0202
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# heterogeneous-network-integration

## Summary

Integration of a gene network (GCN or GRN) with drug–target interaction databases into a heterogeneous gene–drug network, as done in SCANet to link M11/M21 module proteins to repurposing candidates via Drugst.One and the NeDRex database.

## When to use

Use when you have an inferred gene network (e.g. a co-expression module GCN or a TF–TG GRN) and need to connect its genes/proteins to drugs to propose mechanistic repurposing candidates.

## When NOT to use

- When the goal is co-expression module inference or TF–TG GRN reconstruction rather than drug linking — use SCANet's upstream GCN/GRN inference steps instead
- When reproducibility across time is critical and a pinned gene–drug database version is required — Drugst.One's gene–drug databases update weekly and a version control system is still in development

## Inputs

- Gene co-expression network (GCN) or gene regulatory network (GRN) for a selected module (e.g. M11 with TFs GATA2, MAFF, FOSB)
- Gene/protein identifiers from the module (mapped to UniProt, Entrez, Ensembl via Drugst.One)
- Drug–target interaction database choice (default NeDRex; alternatives: DrugCentral, ChEMBL, DGIdb)

## Outputs

- Heterogeneous gene–drug network linking module proteins to candidate drugs
- Structured table of repurposing drug candidates (e.g. m11_drug_candidates.csv listing amlexanox, dexamethasone, olopatadine, abacavir for M11)

## How to apply

Feed the module's network genes into SCANet's drug repurposing function, which uses the Drugst.One package to map gene identifiers to a primary common identifier (UniProt ID, plus Entrez/Ensembl) and connect the human interactome to drug–target interaction databases. Query the default NeDRex database — which merges DrugBank and DrugCentral and encompasses small molecules, biologics, approved drugs, and experimental ones; SCANet also supports DrugCentral, ChEMBL, and DGIdb as alternative sources — to link drugs to the network proteins and form the heterogeneous gene–drug network. Extract the candidate repurposing drugs targeting the mechanistic key driver network (e.g. the M11 GRN with TFs GATA2, MAFF, FOSB regulating pro-inflammatory TGs) and compile them into a structured table. Note that GCN–drug interactions were filtered at a correlation threshold (≥ 0.9 for M11, ≥ 0.8 for the ISC module M4) before linking to drugs.

## Related tools

- **SCANet** (Provides the drug repurposing function that takes module network genes and builds the heterogeneous gene–drug network) — https://github.com/oubounyt/SCANet
- **Drugst.One** (Maps gene identifiers to a primary common identifier (UniProt, Entrez, Ensembl) and connects the human interactome to drug–target databases)
- **NeDRex** (Default drug–target database queried by SCANet; merges DrugBank and DrugCentral)
- **DrugBank** (Component drug–target source within NeDRex)
- **DrugCentral** (Component drug–target source within NeDRex, also available as a standalone alternative database in SCANet)
- **ChEMBL** (Alternative drug–target interaction source supported by SCANet)
- **DGIdb** (Alternative drug–target interaction source supported by SCANet)
- **Python** (Implementation language; SCANet is a Python package installed via pip) — https://pypi.org/project/scanet/

## Examples

```
import scanet as sn; sn.drug_repurposing(module_genes=['GATA2','MAFF','FOSB'], network='M11', database='NeDRex')
```

## Evaluation signals

- Output table reproduces the reported candidate list — anti-inflammatory agents amlexanox, dexamethasone, and olopatadine and antiviral drug abacavir — verified by checking presence of these drug names as substrings in m11_drug_candidates.csv
- Gene identifiers are successfully mapped to a primary common identifier (UniProt ID, plus Entrez/Ensembl) before drug linking
- The heterogeneous network links drugs to the mechanistic key driver network proteins (e.g. M11 GRN TFs and TGs), not to unrelated genes
- The chosen drug database matches the intended source (NeDRex by default; DrugCentral, ChEMBL, or DGIdb if specified)
- Module genes entering the drug-linking step respect the correlation threshold used upstream (≥ 0.9 for the M11 GCN, ≥ 0.8 for the ISC M4 GCN–drug interactions)

## Limitations

- Gene–drug databases in Drugst.One update weekly, impacting reproducibility; running identical analyses and parameters on newer datasets may yield different results
- A version control system for Drugst.One is still in development and not yet available
- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis that propagate into drug–target links
- Converting GCNs into GRNs may increase the risk of false negatives, causing some subtle true relationships between genes to be undetected

## Evidence

- [other] Feed the M11 network genes into SCANet's drug repurposing function, which uses the Drugst.One package to map gene identifiers to a primary common identifier (UniProt ID, plus Entrez/Ensembl) and connect the human interactome to drug–target interaction databases.: [evidence span withheld — anchor status: not_found]
- [methods] SCANet uses by default the database NeDRex, which is an extensive dataset encompassing small molecules, biologics, approved drugs, and experimental ones.: [evidence span withheld — anchor status: not_found]
- [methods] DrugCentral (Avram et al. 2021), ChEMBL (Davies et al. 2015), and DGIdb (Freshour et al. 2021) are alternative drug–target sources supported by SCANet.: "DrugCentral (Avram et al. 2021), ChEMBL (Davies et al. 2015), and DGIdb (Freshour et al. 2021)"
- [results] Anti-inflammatory agents amlexanox, dexamethasone, and olopatadine, and antiviral drug abacavir were identified as potential repurposing candidates for the M11 network.: "we identified anti-inflammatory agents like amlexanox, dexamethasone, and olopatadine, which could modulate these aggressive immune responses"
- [discussion] Gene–drug databases in Drugst.One update weekly, impacting reproducibility.: "gene–drug databases in Drugst.One update weekly, impacting reproducibility. Although old results are accessible, running identical analyses and parameters on newer datasets may yield different results"
- [readme] SCANet is a Python package for gene co-expression network inference and drug–gene interaction identification.: "deciphering of co-regulatory signals in co-expression, and drug-gene interactions identification. This will likely accelerate network analysis pipelines"
