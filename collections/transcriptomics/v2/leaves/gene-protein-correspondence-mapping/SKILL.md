---
name: gene-protein-correspondence-mapping
description: Use when constructing an intercellular communication database from protein-centric
  PPI sources (e.g., STRING) that must be expressed as gene pairs to match scRNA-seq
  gene expression matrices for human or mouse.
when_to_use_negative:
- Input is already a gene expression matrix keyed by gene symbols and no protein-level
  interaction database needs to be built
- The target species is neither human nor mouse, since the InterCellDB gene database
  covers only these two organisms
- The goal is downstream network inference (e.g., permutation tests on cell-cluster
  pairs) rather than database construction
tools:
- name: InterCellDB
  role: R package that consumes the gene-coded protein database; its interaction filtering
    matches gene pairs inferred from STRING protein-protein interactions to scRNA-seq
    data
  repo: https://github.com/ZJUDBlab/InterCellDB
- name: Ensembl genomes database
  role: Source of the complete human and mouse protein data used as the entry point
    of the mapping
- name: NCBI genomes database
  role: Provides the gene-protein correspondence references used to convert proteins
    into corresponding coding genes
- name: COMPARTMENTS
  role: Source of cellular localization annotations appended to each gene-coded protein
    (with adjustable confidence cut-off)
- name: UniProt
  role: Source of Molecular Function classifications consolidated into 16 functional
    groups for protein annotation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202200045
    title: 'InterCellDB: A User‐Defined Database for Inferring Intercellular Networks'
  source_content_hash: sha256:10e71ad418daebee66e5da08fbbe0678aaea7708b7cac196f64090259bd30d3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUDBlab/InterCellDB
  tools:
  - COMPARTMENTS
  - Ensembl genomes database
  - InterCellDB
  - NCBI genomes database
  - R
  - STRING
  - UniProt
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gene-protein-correspondence-mapping

## Summary

Builds a gene-coded protein database for InterCellDB by acquiring complete human (127 750) and mouse (21 291) protein data from the Ensembl genomes database and converting proteins into corresponding coding genes via NCBI genomes database references, enabling protein-protein interactions to be expressed as gene-gene pairs for scRNA-seq intercellular analysis.

## When to use

Use when constructing an intercellular communication database from protein-centric PPI sources (e.g., STRING) that must be expressed as gene pairs to match scRNA-seq gene expression matrices for human or mouse.

## When NOT to use

- Input is already a gene expression matrix keyed by gene symbols and no protein-level interaction database needs to be built
- The target species is neither human nor mouse, since the InterCellDB gene database covers only these two organisms
- The goal is downstream network inference (e.g., permutation tests on cell-cluster pairs) rather than database construction

## Inputs

- Complete human and mouse protein lists from the Ensembl genomes database (127 750 human / 21 291 mouse proteins)
- Gene-protein correspondence references from the NCBI genomes database (snapshot dated March 20, 2020)
- COMPARTMENTS database subcellular localization annotations
- UniProt Molecular Function classifications and Gene Ontology (GO) terms

## Outputs

- Gene-coded protein database: 18 990 human genes and 20 938 mouse genes with full annotation
- Human and mouse gene lists mapping each protein to its coding gene
- Annotated gene entries with 13 subcellular location parameters and 16 consolidated molecular function groups

## How to apply

Acquire the complete protein sets from the Ensembl genomes database (127 750 human proteins, 21 291 mouse proteins in the paper, using Ensembl and NCBI snapshots dated March 20, 2020). Using the references provided by the NCBI genomes database, convert each protein into its corresponding coding gene to generate human and mouse gene lists. Append annotation information for each gene-coded protein: 13 subcellular location parameters from the COMPARTMENTS database and functional classification consolidated from UniProt 'Molecular Function' (132 classes mapped to 16 groups) plus GO biological processes. Validate the mapping by confirming the assembled gene database contains 18 990 human genes and 20 938 mouse genes with full annotation. The resulting gene database is what allows STRING protein-protein interactions to be matched to statistically significant gene pairs between cell clusters.

## Related tools

- **InterCellDB** (R package that consumes the gene-coded protein database; its interaction filtering matches gene pairs inferred from STRING protein-protein interactions to scRNA-seq data) — https://github.com/ZJUDBlab/InterCellDB
- **Ensembl genomes database** (Source of the complete human and mouse protein data used as the entry point of the mapping)
- **NCBI genomes database** (Provides the gene-protein correspondence references used to convert proteins into corresponding coding genes)
- **COMPARTMENTS** (Source of cellular localization annotations appended to each gene-coded protein (with adjustable confidence cut-off))
- **UniProt** (Source of Molecular Function classifications consolidated into 16 functional groups for protein annotation)

## Examples

```
devtools::install_github("ZJUDBlab/InterCellDB")
```

## Evaluation signals

- The assembled gene database contains 18 990 human genes and 20 938 mouse genes, matching the counts reported in the paper
- The protein input sets total 127 750 human proteins and 21 291 mouse proteins acquired from the Ensembl genomes database
- Each gene-coded protein carries annotation from COMPARTMENTS (cellular localization) and UniProt/GO (functional classification)
- Figshare data packages for database generation (human 10.6084/m9.figshare.17057339, mouse 10.6084/m9.figshare.17057342) reproduce the same mapping for release v0.9.2

## Limitations

- Only human and mouse are covered; other species require building an analogous gene-protein mapping from scratch
- The mapping depends on database snapshots (Ensembl/NCBI dated March 20, 2020); newer genome annotations may change gene counts
- Some proteins may not convert to a unique coding gene, reducing the protein set (127 750 human proteins) to fewer genes (18 990 human genes)

## Evidence

- [results] we first acquired the complete human (127 750 proteins) and mouse (21 291 proteins) protein data from the Ensembl genomes database: "we first acquired the complete human (127 750 proteins) and mouse (21 291 proteins) protein data from the Ensembl genomes database"
- [results] based on the references provided by the NCBI genomes database, we converted proteins into corresponding coding genes: "based on the references provided by the NCBI genomes database, we converted proteins into corresponding coding genes"
- [results] The gene database contained 18 990 human genes and 20 938 mouse genes with a variety of annotations: "The gene database contained 18 990 human genes and 20 938 mouse genes with a variety of annotations"
- [results] cellular localization from the COMPARTMENTS database and functional classification from the Uniprot and Gene Ontology (GO) databases: "cellular localization from the COMPARTMENTS database and functional classification from the Uniprot and Gene Ontology (GO) databases"
- [readme] Covering human and mouse; Gene subset selection by subcellular location, molecular function and involving biological process: "- Covering human and mouse
- Gene subset selection by subcellular location, molecular function and involving biological process"
