---
name: protein-identifier-mapping-uniprot
description: Use when a SCANet co-expression module or GRN (e.g. M11 monocyte network
  or M4 ISC hub network at correlation ≥ 0.8) must be converted to UniProt identifiers
  before drug–target interaction queries.
when_to_use_negative:
- The analysis goal is upstream network inference (co-expression modules, hub detection,
  GRN reconstruction) rather than drug repurposing — identifier mapping to UniProt
  is only the entry point of the drug–target query step.
- The input scRNA-seq data are not from human tissue; Drugst.One links the human interactome
  to drug–target databases, so non-human gene identifiers are out of scope.
- Genes are already provided as UniProt IDs and no drug–target interaction query (NeDRex/DrugBank/DrugCentral/ChEMBL/DGIdb)
  is planned, making the mapping step redundant.
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: SCANet
  role: Python package whose drug repurposing function orchestrates the identifier
    mapping and drug–target query for a module's genes
  repo: https://github.com/oubounyt/SCANet
- name: Drugst.One
  role: Maps gene identifiers to a primary common identifier (UniProt ID, plus Entrez/Ensembl)
    and connects the human interactome to drug–target interaction databases
- name: NeDRex
  role: Default drug–target database (merging DrugBank and DrugCentral) queried with
    the UniProt-mapped proteins to form the gene–drug network
- name: DrugBank
  role: Alternative/source drug–target database underlying NeDRex for linking drugs
    to mapped targets
- name: DrugCentral
  role: Alternative drug–target database (also merged into NeDRex) selectable instead
    of the default source
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:ffd462830deaa4ae713aeacdf0d3254c5922686c5ef511c82deac5cc254c4113
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# protein-identifier-mapping-uniprot

## Summary

Maps gene symbols from a SCANet-derived co-expression module (GCN or GRN) to a primary common protein identifier — UniProt ID, plus Entrez/Ensembl — via the Drugst.One package, as the prerequisite step for querying drug–target interaction databases and extracting repurposing candidates.

## When to use

Use when a SCANet co-expression module or GRN (e.g. M11 monocyte network or M4 ISC hub network at correlation ≥ 0.8) must be converted to UniProt identifiers before drug–target interaction queries.

## When NOT to use

- The analysis goal is upstream network inference (co-expression modules, hub detection, GRN reconstruction) rather than drug repurposing — identifier mapping to UniProt is only the entry point of the drug–target query step.
- The input scRNA-seq data are not from human tissue; Drugst.One links the human interactome to drug–target databases, so non-human gene identifiers are out of scope.
- Genes are already provided as UniProt IDs and no drug–target interaction query (NeDRex/DrugBank/DrugCentral/ChEMBL/DGIdb) is planned, making the mapping step redundant.

## Inputs

- Gene list / network nodes from a SCANet co-expression module or GRN (e.g. M11 with 90 genes, M4 hub network) as gene symbols
- Human protein-coding gene–protein identifier table handled internally by Drugst.One (UniProt, Entrez, Ensembl namespaces)

## Outputs

- Identifier-mapped protein set (UniProt IDs plus Entrez/Ensembl) corresponding to the input module genes
- Heterogeneous gene–drug network feeding the repurposing candidate table (e.g. m11_drug_candidates.csv, drug_repurposing_M4_ISC.csv)

## How to apply

Run SCANet's drug repurposing function, which internally invokes Drugst.One to map the module's network proteins to a primary common identifier (UniProt ID, plus Entrez/Ensembl) and connects the human interactome to drug–target interaction databases. By default the identifier-mapped network is queried against the NeDRex database, which merges DrugBank and DrugCentral entries for small molecules, biologics, approved and experimental drugs; SCANet also supports DrugCentral, ChEMBL, and DGIdb as alternative sources. For the ISC module M4, restrict GCN–drug interaction candidates to gene correlations ≥ 0.8; for the fatal COVID-19 M11 module, feed the full GRN (TFs GATA2, MAFF, FOSB regulating pro-inflammatory TGs) into the mapping and query step. The rationale is that drug–target databases key on protein identifiers, so heterogeneous gene symbols from scRNA-seq modules must first be normalized before forming the heterogeneous gene–drug network. Because Drugst.One drug–target databases update weekly, record the database version used at mapping time.

## Related tools

- **SCANet** (Python package whose drug repurposing function orchestrates the identifier mapping and drug–target query for a module's genes) — https://github.com/oubounyt/SCANet
- **Drugst.One** (Maps gene identifiers to a primary common identifier (UniProt ID, plus Entrez/Ensembl) and connects the human interactome to drug–target interaction databases)
- **NeDRex** (Default drug–target database (merging DrugBank and DrugCentral) queried with the UniProt-mapped proteins to form the gene–drug network)
- **DrugBank** (Alternative/source drug–target database underlying NeDRex for linking drugs to mapped targets)
- **DrugCentral** (Alternative drug–target database (also merged into NeDRex) selectable instead of the default source)

## Examples

```
pip install scanet; python -c "import scanet as sn"
```

## Evaluation signals

- Every input module gene with a known human protein product carries a mapped UniProt ID (plus Entrez/Ensembl) before the drug–target query; genes lacking a protein mapping are reported rather than silently dropped.
- The heterogeneous gene–drug network is built only against the chosen database (default NeDRex = DrugBank + DrugCentral); verify the source parameter matches the intended NeDRex/DrugCentral/ChEMBL/DGIdb selection.
- For the M11 COVID-19 module, the candidate table contains amlexanox, dexamethasone, olopatadine, and abacavir as drug-name substrings in m11_drug_candidates.csv.
- For the M4 ISC module (gene correlation ≥ 0.8), the drug–target table contains 20 drug entries spanning eight distinct pharmacological targets, including lovastatin, atorvastatin, pravastatin, fenofibrate, lomitapide, ezetimibe, methyldopa, furosemide, and warfarin.
- Database version is recorded alongside results, since Drugst.One gene–drug databases update weekly and result sets may drift across versions.

## Limitations

- Gene–drug databases in Drugst.One update weekly, impacting reproducibility; running identical analyses and parameters on newer database versions may yield different results.
- A version control system for Drugst.One is still in development and not yet available, so consistent results across SCANet versions cannot be guaranteed.
- The GCN-to-drug step relies on correlation-based module membership (e.g. correlation ≥ 0.8 for M4), which can produce false positives, and GRN conversion may add false negatives.

## Evidence

- [methods] SCANet uses by default the database "NeDRex," which is an extensive dataset encompassing small molecules, biologics, approved drugs, and experimental ones: [evidence span withheld — anchor status: not_found]
- [methods] link the respective drug targets in databases such as NeDRex (Sadegh et al. 2021), DrugBank (Wishart et al. 2018): "link the respective drug targets in databases such as NeDRex (Sadegh et al. 2021), DrugBank (Wishart et al. 2018)"
- [discussion] gene–drug databases in Drugst.One update weekly, impacting reproducibility. Although old results are accessible, running identical analyses and parameters on newer datasets may yield different results: "gene–drug databases in Drugst.One update weekly, impacting reproducibility. Although old results are accessible, running identical analyses and parameters on newer datasets may yield different"
- [readme] a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data and a complete analysis of the identified modules through trait and cell type associations, hub genes detection: "a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data and a complete analysis of the identified modules through trait and cell t"
- [readme] Finally, it should be possible to import drugstone to your python script.: "Finally, it should be possible to import drugstone to your python script."
