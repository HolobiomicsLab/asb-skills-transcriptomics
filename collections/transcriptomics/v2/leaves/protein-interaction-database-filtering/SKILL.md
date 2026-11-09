---
name: protein-interaction-database-filtering
description: Use when building or subsetting a cell–cell interaction database from
  STRING/COMPARTMENTS/UniProt annotations, or when filtering candidate ligand–receptor
  and protein–protein pairs by localization, evidence source, action mode, or effect
  before intercellular analysis.
when_to_use_negative:
- The goal is de novo prediction of protein–protein interactions from sequence or
  structure — this skill only partitions and filters STRING-derived interaction evidence,
  it does not predict new pairs
- Only ligand–receptor pairs are needed and receptor–receptor, extracellular matrix–receptor,
  or vesicle–cell interaction types must be excluded — InterCellDB's user-defined
  scope deliberately covers these broader interaction types
- The input is already a curated, tool-specific ligand–receptor list (e.g. CellPhoneDB's
  built-in database) that requires no localization or action-mode subsetting
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: InterCellDB
  role: R package whose gene and interaction sub-libraries (exp/know/pred) implement
    the partitioning and localization/action-mode filtering workflow; gene pair matching
    and table writing via Tool.WriteTables
  repo: https://github.com/ZJUDBlab/InterCellDB
- name: STRING
  role: Source of all protein–protein interactions and evidence labels ("experiments
    > 0", "database > 0", low-evidence predicted sources) used to partition the database
- name: COMPARTMENTS
  role: Provides the 13 subcellular location parameters and the user-adjustable localization
    cut-off used to filter out biologically unlikely interactions
- name: UniProt
  role: Molecular Function classifications mapped into 16 consolidated functional
    groups for gene subset selection
- name: R
  role: Runtime environment for InterCellDB data calculation and visualization; installed
    via devtools/BiocManager
  repo: https://github.com/ZJUDBlab/InterCellDB
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1002/advs.202200045
    title: 'InterCellDB: A User‐Defined Database for Inferring Intercellular Networks'
  source_content_hash: sha256:4f19c34a91cf8a7e9bd0e7ad130ef20137839d031a330739908d2c6ceacc2e84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUDBlab/InterCellDB
  tools:
  - COMPARTMENTS
  - CellChat
  - CellPhoneDB
  - Ensembl genomes database
  - InterCellDB
  - NCBI genomes database
  - NicheNet
  - R
  - STRING
  - SingleCellSignalR
  - UniProt
  - iTALK
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 24
---

# protein-interaction-database-filtering

## Summary

Construct and partition a protein–protein interaction database (STRING v11) into experiment validated, pathway curated, and predicted sub-libraries, then filter biologically unlikely interactions using COMPARTMENTS subcellular localization, credibility scores, and action mode/effect annotations for user-defined intercellular communication analysis with InterCellDB.

## When to use

Use when building or subsetting a cell–cell interaction database from STRING/COMPARTMENTS/UniProt annotations, or when filtering candidate ligand–receptor and protein–protein pairs by localization, evidence source, action mode, or effect before intercellular analysis.

## When NOT to use

- The goal is de novo prediction of protein–protein interactions from sequence or structure — this skill only partitions and filters STRING-derived interaction evidence, it does not predict new pairs
- Only ligand–receptor pairs are needed and receptor–receptor, extracellular matrix–receptor, or vesicle–cell interaction types must be excluded — InterCellDB's user-defined scope deliberately covers these broader interaction types
- The input is already a curated, tool-specific ligand–receptor list (e.g. CellPhoneDB's built-in database) that requires no localization or action-mode subsetting

## Inputs

- STRING v11 protein–protein interaction records (https://version-11-0.string-db.org)
- COMPARTMENTS database subcellular localization annotations
- UniProt 'Molecular Function' classifications and Gene Ontology (GO) terms (e.g. GO: 0006955 immune response)
- Human (127750 proteins) and mouse (21291 proteins) protein lists from the Ensembl genomes database (snapshot March 20, 2020)
- NCBI genomes database gene–protein correspondence references
- Statistically significant gene pairs computed from scRNA-seq cluster comparisons

## Outputs

- Three partitioned sub-libraries: InterCellDB.exp (experiment validated), InterCellDB.know (pathway curated), InterCellDB.pred (predicted), each retaining evidence source, credibility score, action mode, and action effect from STRING
- Gene-coded protein database with 18990 human and 20938 mouse fully annotated genes (13 localization parameters, 16 functional groups)
- Filtered set of matched protein/gene pairs between two target cell clusters (e.g. endotheliocyte–microglia), written via InterCellDB's Tool.WriteTables

## How to apply

Download human and mouse protein data from the Ensembl genomes database (127750 human and 21291 mouse proteins) and convert proteins to coding genes via NCBI references, yielding 18990 human and 20938 mouse annotated genes. Extract all protein–protein interactions from STRING v11 and partition them into experiment validated (STRING label "experiments > 0"), pathway curated (label "database > 0"), and predicted (score > 0 in low-evidence sources such as neighborhood, fusion, cooccurence, homology, coexpression, textmining) subsets, attaching credibility scores (highest ≥ 900, high ≥ 700, medium ≥ 400, low < 400), action mode (activation, binding, catalysis, expression, inhibition, ptmod, reaction, other), and action effect (positive, negative, unspecified, undirected). Annotate each protein with 13 COMPARTMENTS subcellular location parameters and map the 132 UniProt Molecular Function classifications into 16 consolidated functional groups. To filter out biologically unlikely interactions, apply a user-adjustable COMPARTMENTS localization cut-off and restrict candidate pairs by action mode (e.g. "binding" so factors released from cells on the y-axis can bind receptors on the x-axis) and effect (e.g. "activation" + "positive" between CAF1 and myeloid cells). Match statistically significant gene pairs from scRNA-seq analysis against the filtered database — only matched gene pairs are retained as potential interactions between two target cells.

## Related tools

- **InterCellDB** (R package whose gene and interaction sub-libraries (exp/know/pred) implement the partitioning and localization/action-mode filtering workflow; gene pair matching and table writing via Tool.WriteTables) — https://github.com/ZJUDBlab/InterCellDB
- **STRING** (Source of all protein–protein interactions and evidence labels ("experiments > 0", "database > 0", low-evidence predicted sources) used to partition the database)
- **COMPARTMENTS** (Provides the 13 subcellular location parameters and the user-adjustable localization cut-off used to filter out biologically unlikely interactions)
- **UniProt** (Molecular Function classifications mapped into 16 consolidated functional groups for gene subset selection)
- **R** (Runtime environment for InterCellDB data calculation and visualization; installed via devtools/BiocManager) — https://github.com/ZJUDBlab/InterCellDB

## Examples

```
BiocManager::install(pkgs = "GO.db"); devtools::install_github("ZJUDBlab/InterCellDB")
```

## Evaluation signals

- Assembled gene database contains exactly 18990 human genes and 20938 mouse genes with full annotation, matching the counts reported in the paper
- Protein input counts match Ensembl figures: 127750 human and 21291 mouse proteins
- Every interaction pair carries non-empty evidence source, credibility score, action mode, and action effect fields inherited from STRING
- Only gene pairs matched to the filtered interaction database are reported as potential interactions; unmatched significant pairs are absent from the output table
- Localization cut-off is user-adjustable — changing the COMPARTMENTS threshold changes the size of the retained pair set in the expected direction

## Limitations

- Predicted sub-library relies on low-evidence STRING sources (neighborhood, fusion, cooccurence, homology, coexpression, textmining and their transferred variants), so pairs in InterCellDB.pred carry the lowest credibility (< 400)
- STRING v11 snapshot and Ensembl/NCBI data dated March 20, 2020 — newer protein identifiers or interaction evidence are not captured without rebuilding the database
- No version information is stated for some comparison tools (e.g. NicheNet), which complicates exact reproduction of cross-tool benchmarks

## Evidence

- [methods] The first subset contained those protein interactions that were validated by experiments. These protein interactions were labelled with "experiments > 0" in the STRING database.: [evidence span withheld — anchor status: not_found]
- [results] To filter out biologically unlikely interactions, we incorporated the cellular localization information of all gene‐encoded proteins from the COMPARTMENTS database: "To filter out biologically unlikely interactions, we incorporated the cellular localization information of all gene‐encoded proteins from the COMPARTMENTS database"
- [methods] Action mode of protein pairs was set as "binding" to ensure that factors released from cells of the y‐axis could bind to their receptors in cells of the x‐axis.: [evidence span withheld — anchor status: not_found]
- [results] The gene database contained 18 990 human genes and 20 938 mouse genes with a variety of annotations: "The gene database contained 18 990 human genes and 20 938 mouse genes with a variety of annotations"
- [readme] Please use **package-release version 0.9.2** to run all the codes.: "Please use **package-release version 0.9.2** to run all the codes."
