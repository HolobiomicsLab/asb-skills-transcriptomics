---
name: heterogeneous-drug-target-network-construction
description: Use when a detected gene co-expression module (GCN) or gene regulatory
  network (GRN) from scRNA-seq analysis must be linked to repurposable drugs — i.e.,
  after module selection and hub gene detection, when pharmacological targeting of
  driver mechanisms is the goal.
when_to_use_negative:
- Input is a raw single-cell expression matrix or pseudo-bulk aggregate — co-expression
  module inference and module selection must happen first; this skill covers only
  the drug-linking stage.
- The goal is transcription factor–target validation (RcisTarget cis-regulatory motif
  enrichment) rather than drug repurposing — GRN reconstruction is a separate upstream
  skill.
- Network proteins cannot be mapped to UniProt/Entrez/Ensembl identifiers, which Drugst.One
  requires as primary identifiers for NeDRex queries.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0091
tools:
- name: SCANet
  role: Provides the drug repurposing function that links the module GCN/GRN to drugs
    via Drugst.One
  repo: https://github.com/oubounyt/SCANet
- name: Drugst.One
  role: Maps network proteins to primary identifiers (UniProt, Entrez/Ensembl) and
    queries drug–target databases; updated weekly
- name: NeDRex
  role: Default drug–target database queried by SCANet, merging DrugBank and DrugCentral
    entries for small molecules, biologics, approved and experimental drugs
- name: DrugBank
  role: Source database merged into NeDRex for drug–target links
- name: DrugCentral
  role: Source database merged into NeDRex for drug–target links
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:f25e979b6b529bdb83267b6bad1ee73352b92d33a48fdce1f0d4363efb3577ce
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# heterogeneous-drug-target-network-construction

## Summary

Builds a drug–target interaction network from a SCANet-derived co-expression/regulatory module by mapping its proteins to primary identifiers (UniProt, Entrez/Ensembl) via Drugst.One and querying the NeDRex database (merging DrugBank and DrugCentral). Applied to the M4 intestinal stem cell module of HFHSD obese mice, it yields 20 repurposable drugs for eight pharmacological targets.

## When to use

Use when a detected gene co-expression module (GCN) or gene regulatory network (GRN) from scRNA-seq analysis must be linked to repurposable drugs — i.e., after module selection and hub gene detection, when pharmacological targeting of driver mechanisms is the goal.

## When NOT to use

- Input is a raw single-cell expression matrix or pseudo-bulk aggregate — co-expression module inference and module selection must happen first; this skill covers only the drug-linking stage.
- The goal is transcription factor–target validation (RcisTarget cis-regulatory motif enrichment) rather than drug repurposing — GRN reconstruction is a separate upstream skill.
- Network proteins cannot be mapped to UniProt/Entrez/Ensembl identifiers, which Drugst.One requires as primary identifiers for NeDRex queries.

## Inputs

- SCANet-derived co-expression module (GCN) or gene regulatory network (GRN), e.g. the M4 hub network from HFHSD intestinal stem cells
- List of network proteins/genes with gene correlation values (candidates filtered at correlation ≥ 0.8 for GCN–drug interactions)
- Drug–target interaction database accessed through Drugst.One (default NeDRex, which merges DrugBank and DrugCentral)

## Outputs

- Drug–target interaction table written to drug_repurposing_M4_ISC.csv
- List of repurposable drugs per pharmacological target (20 drugs across eight targets for the M4 ISC module)

## How to apply

Load the selected module's hub network from the SCANet GCN analysis, filtering GCN–drug interaction candidates at gene correlation ≥ 0.8 as used for the ISC module M4. Run SCANet's drug repurposing function, which integrates the human interactome with drug–target databases through the Drugst.One package. Map the network's proteins to a primary common identifier (UniProt ID, plus Entrez/Ensembl) via Drugst.One and query the default NeDRex database, which merges DrugBank and DrugCentral entries for small molecules, biologics, approved and experimental drugs. Retrieve drugs targeting the pharmacological targets of the network — for the M4 ISC module this returned 20 drugs for eight targets, including cholesterol/lipid-lowering drugs (lovastatin, atorvastatin, pravastatin, fenofibrate, lomitapide, ezetimibe) and anti-hypertensive drugs (methyldopa, furosemide, warfarin) — and write the table to drug_repurposing_M4_ISC.csv.

## Related tools

- **SCANet** (Provides the drug repurposing function that links the module GCN/GRN to drugs via Drugst.One) — https://github.com/oubounyt/SCANet
- **Drugst.One** (Maps network proteins to primary identifiers (UniProt, Entrez/Ensembl) and queries drug–target databases; updated weekly)
- **NeDRex** (Default drug–target database queried by SCANet, merging DrugBank and DrugCentral entries for small molecules, biologics, approved and experimental drugs)
- **DrugBank** (Source database merged into NeDRex for drug–target links)
- **DrugCentral** (Source database merged into NeDRex for drug–target links)

## Examples

```
import scanet as sn
```

## Evaluation signals

- The output table contains exactly 20 drug entries spanning eight distinct pharmacological targets, matching the count reported in the article abstract for intestinal stem cells of obese mice.
- Every network protein carries a primary common identifier (UniProt ID, plus Entrez/Ensembl) before the NeDRex query is issued.
- The GCN–drug candidate set was filtered at gene correlation ≥ 0.8 for the ISC module M4 (mirroring the M11 GCN threshold of ≥ 0.9 only where the paper specifies it).
- The retrieved drug set includes cholesterol/lipid-lowering drugs (lovastatin, atorvastatin, pravastatin, fenofibrate, lomitapide, ezetimibe) and anti-hypertensive drugs (methyldopa, furosemide, warfarin).
- Re-run variability is expected and acceptable within limits: Drugst.One drug–target databases update weekly, so result sets may drift across database versions.

## Limitations

- Gene–drug databases in Drugst.One update weekly, impacting reproducibility: although old results are accessible, running identical analyses and parameters on newer datasets may yield different results.
- A version control system for Drugst.One is still in development and not yet available, so consistent results across SCANet versions cannot yet be guaranteed.
- Upstream GCNs rely on correlation, not causation, so drug targets inferred from co-expression hubs may include false positives.

## Evidence

- [methods] SCANet uses by default the database "NeDRex," which is an extensive dataset encompassing small molecules, biologics, approved drugs, and experimental ones: [evidence span withheld — anchor status: not_found]
- [results] the same module was selected for GCN–drug interactions (correlation ≥ 0.8) (Fig. 4D): "the same module was selected for GCN–drug interactions (correlation ≥ 0.8) (Fig. 4D)"
- [abstract] we find 20 drugs for eight potential pharmacological targets in cellular driver mechanisms in the intestinal stem cells of obese mice: "we find 20 drugs for eight potential pharmacological targets in cellular driver mechanisms in the intestinal stem cells of obese mice"
- [discussion] gene–drug databases in Drugst.One update weekly, impacting reproducibility: "gene–drug databases in Drugst.One update weekly, impacting reproducibility"
- [readme] deciphering of co-regulatory signals in co-expression, and drug-gene interactions identification: "deciphering of co-regulatory signals in co-expression, and drug-gene interactions identification"
