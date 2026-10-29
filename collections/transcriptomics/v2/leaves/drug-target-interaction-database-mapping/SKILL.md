---
name: drug-target-interaction-database-mapping
description: Use when a gene co-expression or regulatory network module has driver/hub
  genes and you need to retrieve known or experimental drugs targeting them for mechanistic
  drug repurposing.
when_to_use_negative:
- The module lacks identified hub/driver genes or the network proteins cannot be mapped
  to UniProt/Entrez/Ensembl identifiers via Drugst.One.
- The goal is de novo drug discovery or QSAR modeling rather than lookup of known
  drug–target interactions in DrugBank/DrugCentral-derived databases.
- Strict bitwise reproducibility of the drug list is required and no pinned Drugst.One/NeDRex
  database version can be used, since the databases update weekly.
edam_operation: http://edamontology.org/operation_4001
edam_topics:
- http://edamontology.org/topic_0208
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0202
tools:
- name: SCANet
  role: Runs the drug repurposing function that integrates the human interactome with
    drug–target interaction databases on the module hub network
  repo: https://github.com/oubounyt/SCANet
- name: Drugst.One
  role: Maps network proteins to primary common identifiers (UniProt, Entrez, Ensembl)
    and queries drug–target databases
- name: NeDRex
  role: Default drug–target database queried by Drugst.One; merges DrugBank and DrugCentral
    entries for small molecules, biologics, approved and experimental drugs
- name: DrugBank
  role: Source database of drug–target interactions merged into NeDRex
- name: DrugCentral
  role: Source database of drug–target interactions merged into NeDRex
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:4b8a865969e8e7c20fa3134cdb0ec2bed3dc7b26f05f92046b70f026ddf174a0
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
  edam_operation: http://edamontology.org/operation_4001
  edam_topics:
  - http://edamontology.org/topic_0208
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# drug-target-interaction-database-mapping

## Summary

Maps hub-gene proteins from a SCANet-derived co-expression module (e.g. the M4 ISC module of HFHSD obese mice) to drug–target interaction databases via Drugst.One/NeDRex to yield repurposing candidates such as lovastatin, fenofibrate, methyldopa, and warfarin.

## When to use

Use when a gene co-expression or regulatory network module has driver/hub genes and you need to retrieve known or experimental drugs targeting them for mechanistic drug repurposing.

## When NOT to use

- The module lacks identified hub/driver genes or the network proteins cannot be mapped to UniProt/Entrez/Ensembl identifiers via Drugst.One.
- The goal is de novo drug discovery or QSAR modeling rather than lookup of known drug–target interactions in DrugBank/DrugCentral-derived databases.
- Strict bitwise reproducibility of the drug list is required and no pinned Drugst.One/NeDRex database version can be used, since the databases update weekly.

## Inputs

- Gene co-expression network (GCN) module from SCANet with hub genes (e.g. M4 ISC module from HFHSD mouse scRNA-seq data)
- Network protein/gene list requiring identifier mapping (to UniProt, Entrez, Ensembl)
- Gene correlation filter threshold (e.g. ≥ 0.8 for GCN–drug interactions)

## Outputs

- Drug–target interaction table (e.g. drug_repurposing_M4_ISC.csv) listing candidate repurposable drugs per pharmacological target
- Set of repurposing candidates grouped by drug class (e.g. 20 drugs for eight targets: statins, fibrates, anti-hypertensives)

## How to apply

First load the module's hub network from the SCANet GCN analysis, selecting GCN–drug interaction candidates at gene correlation ≥ 0.8 (as used for the ISC module M4). Then run SCANet's drug repurposing function, which integrates the human interactome with drug–target interaction databases through the Drugst.One package. Map the network's proteins to a primary common identifier (UniProt ID, plus Entrez/Ensembl) via Drugst.One and query the default NeDRex database, which merges DrugBank and DrugCentral entries for small molecules, biologics, approved and experimental drugs. Finally, retrieve drugs targeting the identified pharmacological targets and write the drug–target interaction table (e.g. drug_repurposing_M4_ISC.csv); validate by counting distinct drugs and distinct targets against the reported results.

## Related tools

- **SCANet** (Runs the drug repurposing function that integrates the human interactome with drug–target interaction databases on the module hub network) — https://github.com/oubounyt/SCANet
- **Drugst.One** (Maps network proteins to primary common identifiers (UniProt, Entrez, Ensembl) and queries drug–target databases)
- **NeDRex** (Default drug–target database queried by Drugst.One; merges DrugBank and DrugCentral entries for small molecules, biologics, approved and experimental drugs)
- **DrugBank** (Source database of drug–target interactions merged into NeDRex)
- **DrugCentral** (Source database of drug–target interactions merged into NeDRex)

## Examples

```
import scanet as sn; sn.drug_repurposing(network='M4_ISC_hubs', correlation_threshold=0.8, database='NeDRex', out='drug_repurposing_M4_ISC.csv')
```

## Evaluation signals

- Output table contains the expected number of drug entries (e.g. 20 drug entries spanning eight distinct pharmacological targets, matching the count reported in the article abstract for ISC of obese mice).
- Every drug–target row resolves to a pharmacological target that is a hub gene of the input module selected at correlation ≥ 0.8.
- All network proteins were successfully mapped to a primary common identifier (UniProt ID, plus Entrez/Ensembl) before querying NeDRex.
- Identified drugs belong to biologically plausible classes for the trait (e.g. cholesterol/lipid-lowering drugs and anti-hypertensive drugs for the obesogenic M4 ISC module).
- Record the Drugst.One/NeDRex database version/date used, since the underlying drug–target databases update weekly and result sets may drift.

## Limitations

- Gene–drug databases in Drugst.One update weekly, impacting reproducibility; running identical analyses and parameters on newer datasets may yield different results.
- A version control system for Drugst.One is still in development and not yet available, so consistent results across SCANet versions cannot be guaranteed.
- Drug–target interactions retrieved are database annotations (correlation-based network links), not experimentally validated effects; e.g. erlotinib's effect on ERRFI1 remains uncertain and should be examined experimentally.

## Evidence

- [other] Run SCANet's drug repurposing function, which integrates the human interactome with drug–target interaction databases through the Drugst.One package.: [evidence span withheld — anchor status: not_found]
- [methods] SCANet uses by default the database "NeDRex," which is an extensive dataset encompassing small molecules, biologics, approved drugs, and experimental ones: [evidence span withheld — anchor status: not_found]
- [results] the same module was selected for GCN–drug interactions (correlation ≥ 0.8) (Fig. 4D): "the same module was selected for GCN–drug interactions (correlation ≥ 0.8) (Fig. 4D)"
- [abstract] we find 20 drugs for eight potential pharmacological targets in cellular driver mechanisms in the intestinal stem cells of obese mice: "we find 20 drugs for eight potential pharmacological targets in cellular driver mechanisms in the intestinal stem cells of obese mice"
- [discussion] gene–drug databases in Drugst.One update weekly, impacting reproducibility.: "gene–drug databases in Drugst.One update weekly, impacting reproducibility."
