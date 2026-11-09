---
name: covid-19-therapeutic-candidate-evaluation
description: Use when a disease/trait-associated gene co-expression module or GRN
  (e.g. M11 with TFs GATA2, MAFF, FOSB) has been inferred and you need candidate repurposable
  drugs targeting its key driver proteins.
when_to_use_negative:
- The input is raw scRNA-seq counts rather than an already-inferred co-expression
  module or GRN — run SCANet's upstream module inference first
- The goal is novel drug design or de novo target identification rather than repurposing
  of existing approved/experimental drugs present in NeDRex and related databases
- The module lacks a validated association with the trait of interest (e.g. M11's
  highest positive correlation with deceased patients), since drug candidates are
  only mechanistically meaningful for driver networks
edam_operation: http://edamontology.org/operation_3938
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0154
tools:
- name: SCANet
  role: Python package whose drug repurposing function feeds module genes into Drugst.One
    to build the gene–drug network
  repo: https://github.com/oubounyt/SCANet
- name: Drugst.One
  role: Maps gene identifiers to a primary common identifier (UniProt, plus Entrez/Ensembl)
    and connects the human interactome to drug–target databases; its databases update
    weekly, affecting reproducibility
- name: NeDRex
  role: Default drug–target database used to link drugs to the M11 network proteins;
    merges DrugBank and DrugCentral
- name: Python
  role: Runtime language for SCANet; package installable from PyPI, officially supports
    Python 3.6+
  repo: https://pypi.org/project/scanet/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:763af279bad0ebaf9738dc38b7ce4611195b209bcb4625b8666205f7a7661b1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - Drugst.One
  - NeDRex
  - Python
  - SCANet
  edam_operation: http://edamontology.org/operation_3938
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# covid-19-therapeutic-candidate-evaluation

## Summary

Predict mechanistic drug repurposing candidates by mapping a trait-associated gene co-expression/regulatory network (e.g. the M11 module of classical monocytes from deceased COVID-19 patients) onto drug–target interaction databases via SCANet's Drugst.One integration, yielding a structured table of candidate drugs.

## When to use

Use when a disease/trait-associated gene co-expression module or GRN (e.g. M11 with TFs GATA2, MAFF, FOSB) has been inferred and you need candidate repurposable drugs targeting its key driver proteins.

## When NOT to use

- The input is raw scRNA-seq counts rather than an already-inferred co-expression module or GRN — run SCANet's upstream module inference first
- The goal is novel drug design or de novo target identification rather than repurposing of existing approved/experimental drugs present in NeDRex and related databases
- The module lacks a validated association with the trait of interest (e.g. M11's highest positive correlation with deceased patients), since drug candidates are only mechanistically meaningful for driver networks

## Inputs

- A trait/cell-type-specific co-expression module or GRN (e.g. the M11 network: 90-gene GCN of classical monocytes from deceased patients with TFs GATA2, MAFF, FOSB regulating pro-inflammatory TGs)
- Gene identifiers from the module (mapped internally to UniProt/Entrez/Ensembl by Drugst.One)
- Drug–target interaction database selection (default NeDRex; alternatives: DrugCentral, ChEMBL, DGIdb)

## Outputs

- A heterogeneous gene–drug network linking drugs to module proteins
- A structured table of candidate repurposing drugs (e.g. m11_drug_candidates.csv) listing drugs targeting the mechanistic key driver network

## How to apply

Take the previously inferred module network — for the fatal COVID-19 case, the M11 GRN of classical monocytes from deceased patients — and feed its genes into SCANet's drug repurposing function, which uses Drugst.One to map gene identifiers to a primary common identifier (UniProt ID, plus Entrez/Ensembl). Query the default NeDRex database, which merges DrugBank and DrugCentral (SCANet also supports DrugCentral, ChEMBL, and DGIdb as alternative sources), to link drugs to the module's proteins and form a heterogeneous gene–drug network. Extract drugs targeting the mechanistic key driver network and compile them into a structured table (e.g. m11_drug_candidates.csv). Validate by confirming the reported candidates — anti-inflammatory agents amlexanox, dexamethasone, and olopatadine and the antiviral abacavir — appear as substrings in the output table. Note that Drugst.One gene–drug databases update weekly, so reproducibility against reported results may drift.

## Related tools

- **SCANet** (Python package whose drug repurposing function feeds module genes into Drugst.One to build the gene–drug network) — https://github.com/oubounyt/SCANet
- **Drugst.One** (Maps gene identifiers to a primary common identifier (UniProt, plus Entrez/Ensembl) and connects the human interactome to drug–target databases; its databases update weekly, affecting reproducibility)
- **NeDRex** (Default drug–target database used to link drugs to the M11 network proteins; merges DrugBank and DrugCentral)
- **Python** (Runtime language for SCANet; package installable from PyPI, officially supports Python 3.6+) — https://pypi.org/project/scanet/

## Examples

```
pip install scanet; import scanet as sn  # then feed M11 module genes into SCANet's drug repurposing function (NeDRex default)
```

## Evaluation signals

- Output table contains the four reported M11 candidates: amlexanox, dexamethasone, olopatadine (anti-inflammatory) and abacavir (antiviral) — check these drug names appear as substrings in m11_drug_candidates.csv
- Gene identifiers in the input module are successfully mapped to UniProt IDs (plus Entrez/Ensembl) by Drugst.One with no unmapped key driver genes
- The drug candidates target the mechanistic key driver network of the module, not peripheral genes
- Alternative database runs (DrugCentral, ChEMBL, DGIdb) are documented alongside the default NeDRex result so database choice is traceable
- Re-run results are compared with awareness that Drugst.One gene–drug databases update weekly, so divergent candidates on newer database versions are expected

## Limitations

- Gene–drug databases in Drugst.One update weekly, impacting reproducibility; running identical analyses and parameters on newer datasets may yield different results
- A version control system for Drugst.One is still in development and not yet available, so consistent results across versions cannot be guaranteed
- Candidate drugs are inferred from network-level drug–target interactions and require experimental validation (e.g. the precise effect of erlotinib on ERRFI1 in the parallel M21 analysis 'remains uncertain... should be examined further experimentally')

## Evidence

- [methods] Default NeDRex database merges DrugBank and DrugCentral: [evidence span withheld — anchor status: not_found]
- [results] Reported M11 repurposing candidates: "we identified anti-inflammatory agents like amlexanox, dexamethasone, and olopatadine, which could modulate these aggressive immune responses. Additionally, the antiviral drug, abacavir"
- [results] Drugst.One identifier mapping: [evidence span withheld — anchor status: not_found]
- [discussion] Drugst.One weekly updates affect reproducibility: "gene–drug databases in Drugst.One update weekly, impacting reproducibility. Although old results are accessible, running identical analyses and parameters on newer datasets may yield different results"
- [readme] SCANet installation and Python support: "`SCANet` officially supports Python 3.6+."
