---
name: network-based-drug-repurposing
description: Use when a trait- or cell-type-associated co-expression module or GRN
  has already been inferred (e.g. SCANet WGCNA modules like M11/M4) and you need mechanistic
  drug repurposing candidates targeting its hub/key-driver genes.
when_to_use_negative:
- You have no network module yet — run SCANet's upstream steps first (pseudo-bulk
  aggregation, modified WGCNA GCN inference, module selection, GCN→GRN conversion)
  before drug repurposing; the module/trait association must already be established
  (e.g. M11 highest positive correlation with deceased patients).
- The gene list is not from human tissue/cells — Drugst.One maps to the human interactome
  and human-focused drug–target databases, so non-human networks need prior orthology
  mapping.
- Strict bit-for-bit reproducibility across runs is required — Drugst.One drug–target
  databases update weekly, so result sets may drift across database versions.
edam_operation: http://edamontology.org/operation_4001
edam_topics:
- http://edamontology.org/topic_0202
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3308
tools:
- name: SCANet
  role: Python package whose drug repurposing function feeds module genes into Drugst.One
    and builds the gene–drug interaction network
  repo: https://github.com/oubounyt/SCANet
- name: Drugst.One
  role: Maps network proteins to a primary common identifier (UniProt ID, plus Entrez/Ensembl)
    and connects the human interactome to drug–target interaction databases
- name: NeDRex
  role: Default drug–target database queried by SCANet, merging DrugBank and DrugCentral
    entries for small molecules, biologics, approved and experimental drugs
- name: DrugBank
  role: Component of the default NeDRex database linking drugs to M11 network proteins
- name: DrugCentral
  role: Component of the default NeDRex database linking drugs to network proteins;
    also usable as an alternative standalone source
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:3303af949d18c67e21d4355afd17a1babda4f24538c0da190d9d5128bd455db1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - DrugBank
  - DrugCentral
  - Drugst.One
  - NeDRex
  - Python
  - SCANet
  edam_operation: http://edamontology.org/operation_4001
  edam_topics:
  - http://edamontology.org/topic_0202
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# network-based-drug-repurposing

## Summary

Identify repurposable drugs by mapping a trait-associated gene co-expression network (GCN) or gene regulatory network (GRN) — e.g. the M11 classical monocyte module from deceased COVID-19 patients or the M4 ISC module from obese mice — onto drug–target databases via SCANet and Drugst.One, using NeDRex (DrugBank + DrugCentral) as the default interaction source.

## When to use

Use when a trait- or cell-type-associated co-expression module or GRN has already been inferred (e.g. SCANet WGCNA modules like M11/M4) and you need mechanistic drug repurposing candidates targeting its hub/key-driver genes.

## When NOT to use

- You have no network module yet — run SCANet's upstream steps first (pseudo-bulk aggregation, modified WGCNA GCN inference, module selection, GCN→GRN conversion) before drug repurposing; the module/trait association must already be established (e.g. M11 highest positive correlation with deceased patients).
- The gene list is not from human tissue/cells — Drugst.One maps to the human interactome and human-focused drug–target databases, so non-human networks need prior orthology mapping.
- Strict bit-for-bit reproducibility across runs is required — Drugst.One drug–target databases update weekly, so result sets may drift across database versions.

## Inputs

- A SCANet-derived gene co-expression network (GCN) or GRN for a trait-associated module (e.g. M11 network of classical monocytes from deceased patients; M4 hub network from HFHSD intestinal stem cells at gene correlation ≥ 0.8)
- List of module genes / network hubs and key drivers (e.g. TFs GATA2, MAFF, FOSB and their target genes)
- Drug–target interaction database selection: default NeDRex (DrugBank + DrugCentral), or DrugCentral, ChEMBL, DGIdb

## Outputs

- A structured drug–target interaction table (e.g. m11_drug_candidates.csv, drug_repurposing_M4_ISC.csv) listing repurposing candidates (e.g. amlexanox, dexamethasone, olopatadine, abacavir for M11; lovastatin, fenofibrate, lomitapide, pravastatin, atorvastatin, methyldopa, furosemide, warfarin for M4)
- A heterogeneous gene–drug network linking the module's proteins to drug entries

## How to apply

Feed the module's genes (e.g. the M11 GRN with TFs GATA2, MAFF, FOSB regulating pro-inflammatory TGs, or M4 ISC hub genes selected at gene correlation ≥ 0.8) into SCANet's drug repurposing function, which uses Drugst.One to map gene identifiers to a primary common identifier (UniProt ID, plus Entrez/Ensembl) and connect the human interactome to drug–target interaction databases. Query the default NeDRex database (merging DrugBank and DrugCentral; DrugCentral, ChEMBL, and DGIdb are supported alternatives) to link drugs to the network proteins and form the heterogeneous gene–drug network. Extract candidate drugs targeting the mechanistic key drivers and write them to a structured table (e.g. m11_drug_candidates.csv or drug_repurposing_M4_ISC.csv). Validate by checking the presence of expected drug names as substrings in the output CSV and, for the ISC case, by confirming the expected count (20 drugs across eight distinct pharmacological targets).

## Related tools

- **SCANet** (Python package whose drug repurposing function feeds module genes into Drugst.One and builds the gene–drug interaction network) — https://github.com/oubounyt/SCANet
- **Drugst.One** (Maps network proteins to a primary common identifier (UniProt ID, plus Entrez/Ensembl) and connects the human interactome to drug–target interaction databases)
- **NeDRex** (Default drug–target database queried by SCANet, merging DrugBank and DrugCentral entries for small molecules, biologics, approved and experimental drugs)
- **DrugBank** (Component of the default NeDRex database linking drugs to M11 network proteins)
- **DrugCentral** (Component of the default NeDRex database linking drugs to network proteins; also usable as an alternative standalone source)

## Examples

```
pip install scanet; import scanet as sn  # run sn's drug repurposing on the M11/M4 module genes via Drugst.One + NeDRex
```

## Evaluation signals

- For the M11 reproduction: confirm the output table (m11_drug_candidates.csv) contains the drug names amlexanox, dexamethasone, olopatadine (anti-inflammatory) and abacavir (antiviral) as substrings.
- For the ISC M4 reproduction: the output table contains exactly 20 drug entries spanning eight distinct pharmacological targets, matching the count reported in the article abstract.
- Gene identifiers were successfully mapped to a primary common identifier (UniProt ID, plus Entrez/Ensembl) via Drugst.One before database query.
- The queried database is the default NeDRex (DrugBank + DrugCentral) unless an alternative (DrugCentral, ChEMBL, DGIdb) was explicitly selected.
- Drugs returned target the module's mechanistic key driver network proteins, not merely any module gene.

## Limitations

- Drugst.One gene–drug databases update weekly, impacting reproducibility: running identical analyses and parameters on newer database versions may yield different results, although old results are accessible.
- A version control system for Drugst.One is still in development and not yet available, so consistent results across SCANet versions cannot be guaranteed.
- Drug repurposing candidates are network-based predictions; e.g. the precise effect of erlotinib on ERRFI1 (M21) remains uncertain and should be examined experimentally.

## Evidence

- [results] Drugst.One maps to UniProt: [evidence span withheld — anchor status: not_found]
- [methods] NeDRex default database: [evidence span withheld — anchor status: not_found]
- [results] M11 repurposing candidates: "we identified anti-inflammatory agents like amlexanox, dexamethasone, and olopatadine, which could modulate these aggressive immune responses. Additionally, the antiviral drug, abacavir"
- [abstract] 20 drugs for eight targets: "we find 20 drugs for eight potential pharmacological targets in cellular driver mechanisms in the intestinal stem cells of obese mice"
- [discussion] Database drift caveat: "gene–drug databases in Drugst.One update weekly, impacting reproducibility. Although old results are accessible, running identical analyses and parameters on newer datasets may yield different results"
- [results] ISC correlation threshold: "the same module was selected for GCN–drug interactions (correlation ≥ 0.8) (Fig. 4D)"
- [readme] README usage pointer: "we explored all aspects of gene coexpression networks (GCNs) using `SCANet` through a full analysis of the 3k PBMCs from 10x Genomics"
