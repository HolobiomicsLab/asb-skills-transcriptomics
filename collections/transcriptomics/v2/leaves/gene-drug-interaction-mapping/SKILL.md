---
name: gene-drug-interaction-mapping
description: Use when a trait- or cell-type-associated co-expression module or GRN
  (e.g. the M11 network with TFs GATA2, MAFF, FOSB) has already been inferred and
  needs to be linked to repurposable drugs through drug-target interaction databases.
when_to_use_negative:
- Input is raw scRNA-seq data — co-expression modules and GRNs (GCN inference, module
  detection, RcisTarget-validated GRN conversion) must be produced by upstream SCANet
  steps before drug mapping
- No gene identifiers in the module can be mapped to UniProt/Entrez/Ensembl identifiers
  required by Drugst.One
- Strictly reproducible results are required across time — gene-drug databases in
  Drugst.One update weekly and no version control is yet available, so identical reruns
  on newer databases may yield different drugs
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: SCANet
  role: Hosts the drug repurposing function that takes the inferred GRN/GCN module
    genes and builds the gene-drug interaction analysis
  repo: https://github.com/oubounyt/SCANet
- name: Drugst.One
  role: Maps gene identifiers to a primary common identifier (UniProt, plus Entrez/Ensembl)
    and connects the human interactome to drug-target interaction databases
- name: NeDRex
  role: Default drug-target database queried by SCANet (merging DrugBank and DrugCentral)
    to link drugs to M11 network proteins
- name: Python
  role: Language of the SCANet package used to run the repurposing function and compile
    candidates into a structured CSV table
  repo: https://pypi.org/project/scanet/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:a3f46f8dc14d8e667ba8f2762c6afa6e2c596b368a2eaa20a9b71b9d74d0a204
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - Drugst.One
  - NeDRex
  - Python
  - SCANet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# gene-drug-interaction-mapping

## Summary

Maps genes of a mechanistic key driver network (e.g. the SCANet M11 GRN of classical monocytes from deceased COVID-19 patients) onto drug-target databases via Drugst.One to extract repurposing candidates. This is SCANet's final pipeline stage, turning TF–TG regulatory networks into structured drug candidate tables.

## When to use

Use when a trait- or cell-type-associated co-expression module or GRN (e.g. the M11 network with TFs GATA2, MAFF, FOSB) has already been inferred and needs to be linked to repurposable drugs through drug-target interaction databases.

## When NOT to use

- Input is raw scRNA-seq data — co-expression modules and GRNs (GCN inference, module detection, RcisTarget-validated GRN conversion) must be produced by upstream SCANet steps before drug mapping
- No gene identifiers in the module can be mapped to UniProt/Entrez/Ensembl identifiers required by Drugst.One
- Strictly reproducible results are required across time — gene-drug databases in Drugst.One update weekly and no version control is yet available, so identical reruns on newer databases may yield different drugs

## Inputs

- Gene co-expression network (GCN) or gene regulatory network (GRN) module gene list, e.g. the M11 GRN of classical monocytes (TFs GATA2, MAFF, FOSB and pro-inflammatory target genes)
- Human interactome linkage provided by Drugst.One
- Drug-target interaction database selection: NeDRex (default, merging DrugBank and DrugCentral), or DrugCentral, ChEMBL, DGIdb

## Outputs

- Structured table of candidate repurposing drugs targeting the module's mechanistic key driver network (e.g. m11_drug_candidates.csv listing amlexanox, dexamethasone, olopatadine, abacavir)
- Heterogeneous gene-drug network connecting module proteins to drug targets

## How to apply

Feed the module gene list (e.g. M11 network genes, 90 genes with 16 exhibiting correlation >= 0.9) into SCANet's drug repurposing function, which uses the Drugst.One package to map gene identifiers to a primary common identifier (UniProt ID, plus Entrez/Ensembl) and connect the human interactome to drug-target interaction databases. Query the default NeDRex database (merging DrugBank and DrugCentral) to link drugs to the network proteins and form the heterogeneous gene-drug network; DrugCentral, ChEMBL, and DGIdb can be selected as alternative sources. Extract the candidate repurposing drugs targeting the mechanistic key driver network and compile them into a structured CSV table (e.g. m11_drug_candidates.csv). Rationale: the goal is mechanistic repurposing — drugs whose targets intersect the drivers of the disease mechanotype (e.g. the pro-inflammatory M11 GRN regulating TGs of highly pro-inflammatory reactions) rather than generic gene-set enrichment.

## Related tools

- **SCANet** (Hosts the drug repurposing function that takes the inferred GRN/GCN module genes and builds the gene-drug interaction analysis) — https://github.com/oubounyt/SCANet
- **Drugst.One** (Maps gene identifiers to a primary common identifier (UniProt, plus Entrez/Ensembl) and connects the human interactome to drug-target interaction databases)
- **NeDRex** (Default drug-target database queried by SCANet (merging DrugBank and DrugCentral) to link drugs to M11 network proteins)
- **Python** (Language of the SCANet package used to run the repurposing function and compile candidates into a structured CSV table) — https://pypi.org/project/scanet/

## Examples

```
pip install scanet; import scanet as sn
```

## Evaluation signals

- Presence of expected drug names as substrings in the candidate output table (e.g. 'amlexanox', 'dexamethasone', 'olopatadine', 'abacavir' in m11_drug_candidates.csv)
- Identifier mapping succeeded: module genes resolved to primary UniProt IDs (plus Entrez/Ensembl) by Drugst.One
- Gene-drug network is heterogeneous, linking module proteins (drivers of the pro-inflammatory mechanotype) to drugs from the chosen database
- The queried database and any alternative source selection (NeDRex vs DrugCentral/ChEMBL/DGIdb) is recorded with the output
- Only drugs targeting the module's mechanistic key driver network are returned, not an unfiltered whole-genome drug list

## Limitations

- Gene-drug databases in Drugst.One update weekly, impacting reproducibility; running identical analyses and parameters on newer datasets may yield different results
- A version control system for Drugst.One is still in development and not yet available, so consistent results across versions cannot be guaranteed
- Upstream co-expression networks rely on correlation, not causation, potentially leading to false positives that propagate into drug candidates; GCN-to-GRN conversion may also cause false negatives for subtle true relationships

## Evidence

- [methods] SCANet's drug repurposing uses Drugst.One to map identifiers and query NeDRex: [evidence span withheld — anchor status: not_found]
- [results] M11 repurposing candidates found: "we identified anti-inflammatory agents like amlexanox, dexamethasone, and olopatadine, which could modulate these aggressive immune responses"
- [discussion] Reproducibility caveat from Drugst.One weekly updates: "gene–drug databases in Drugst.One update weekly, impacting reproducibility. Although old results are accessible, running identical analyses and parameters on newer datasets may yield different results"
- [readme] README installation of the SCANet package: "To install `SCANet`.
console
pip install scanet
"
