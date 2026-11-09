---
name: subcellular-location-annotation
description: Use when constructing an intercellular communication interaction database
  or when filtering candidate protein-protein/ligand-receptor pairs so that only proteins
  with compatible subcellular localizations (e.g., extracellular region, plasma membrane)
  are paired.
when_to_use_negative:
- The task is de novo prediction of protein localization rather than annotation-based
  filtering of interaction candidates
- The analysis does not involve intercellular communication from scRNA-seq data, so
  localization-compatibility filtering of gene pairs is unnecessary
- Input is already a filtered, experimentally validated ligand-receptor list where
  COMPARTMENTS-based localization filtering could inappropriately remove secreted/membrane
  partners
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: InterCellDB
  role: R package whose gene database stores COMPARTMENTS-derived subcellular location
    annotations and applies the localization cut-off to filter unlikely interactions
  repo: https://github.com/ZJUDBlab/InterCellDB
- name: COMPARTMENTS
  role: Source database of gene-encoded protein cellular localization information
    used to filter out biologically unlikely interactions
- name: R
  role: Runtime environment in which InterCellDB data calculation and visualization
    (including localization-based filtering) are performed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202200045
    title: 'InterCellDB: A User‐Defined Database for Inferring Intercellular Networks'
  source_content_hash: sha256:d763c2afaf39b09a1ccc73fbfbd6fd00ddd09d271a01d2c98d17413a3b552718
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUDBlab/InterCellDB
  tools:
  - COMPARTMENTS
  - InterCellDB
  - R
  - STRING
  - UniProt
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# subcellular-location-annotation

## Summary

Annotate gene-coded proteins with subcellular location parameters from the COMPARTMENTS database and use them, with an adjustable confidence score cut-off, to filter out biologically unlikely cell-cell interactions in InterCellDB.

## When to use

Use when constructing an intercellular communication interaction database or when filtering candidate protein-protein/ligand-receptor pairs so that only proteins with compatible subcellular localizations (e.g., extracellular region, plasma membrane) are paired.

## When NOT to use

- The task is de novo prediction of protein localization rather than annotation-based filtering of interaction candidates
- The analysis does not involve intercellular communication from scRNA-seq data, so localization-compatibility filtering of gene pairs is unnecessary
- Input is already a filtered, experimentally validated ligand-receptor list where COMPARTMENTS-based localization filtering could inappropriately remove secreted/membrane partners

## Inputs

- Gene-coded protein database built from Ensembl (127750 human, 21291 mouse proteins) and NCBI genome database references
- Subcellular location records from the COMPARTMENTS database (with confidence scores)
- Functional classification annotations from UniProt Molecular Function and Gene Ontology (GO)

## Outputs

- Annotated gene database (18990 human, 20938 mouse genes) with cellular localization parameters per protein
- Filtered set of candidate cell-cell interactions in which biologically unlikely pairs (incompatible localizations below the cut-off) are removed
- User-adjustable localization cut-off setting for downstream intercellular network analyses

## How to apply

Extract subcellular location information for every gene-encoded protein from the COMPARTMENTS database and attach it as annotation alongside functional classification from UniProt and GO. In InterCellDB, this localization annotation underpins a 13-category scheme (extracellular region, plasma membrane, cytosol, cytoskeleton, peroxisome, lysosome, endoplasmic reticulum, Golgi apparatus, endosome, mitochondrion, cytoplasm, nucleus, other) used for gene subset selection. When matching statistically significant gene pairs to the interaction database, apply the localization filter with a confidence score cut-off from COMPARTMENTS to discard biologically unlikely interactions. The cut-off value is deliberately user-adjustable so it can be tuned to the experimental context. Validate the result by confirming the gene database retains full annotation for its gene counts (18990 human, 20938 mouse genes in the paper).

## Related tools

- **InterCellDB** (R package whose gene database stores COMPARTMENTS-derived subcellular location annotations and applies the localization cut-off to filter unlikely interactions) — https://github.com/ZJUDBlab/InterCellDB
- **COMPARTMENTS** (Source database of gene-encoded protein cellular localization information used to filter out biologically unlikely interactions)
- **R** (Runtime environment in which InterCellDB data calculation and visualization (including localization-based filtering) are performed)

## Examples

```
devtools::install_github("ZJUDBlab/InterCellDB")
```

## Evaluation signals

- Gene database annotation completeness matches the paper: 18990 human genes and 20938 mouse genes with a variety of annotations including localization
- Protein inputs reconcile with Ensembl counts: 127750 human and 21291 mouse proteins before conversion to coding genes
- The COMPARTMENTS confidence score cut-off is exposed as a user-adjustable parameter rather than hardcoded
- Filtered interaction output contains only gene pairs whose localization annotations are compatible (e.g., ligand in extracellular region/plasma membrane vs receptor on plasma membrane)
- Package-release version 0.9.2 is used, matching the README's stated requirement for running released codes

## Limitations

- The localization cut-off value must be chosen per analysis; a fixed default may not suit all experimental requirements
- Version information for some compared tools (e.g., NicheNet) is not stated, complicating reproducible cross-tool localization filtering comparisons

## Evidence

- [results] To filter out biologically unlikely interactions, we incorporated the cellular localization information of all gene‐encoded proteins from the COMPARTMENTS database: "To filter out biologically unlikely interactions, we incorporated the cellular localization information of all gene‐encoded proteins from the COMPARTMENTS database"
- [results] Users can adjust this cut‐off value depending on their experimental requirements.: "Users can adjust this cut‐off value depending on their experimental requirements."
- [results] cellular localization from the COMPARTMENTS database and functional classification from the Uniprot and Gene Ontology (GO) databases: "cellular localization from the COMPARTMENTS database and functional classification from the Uniprot and Gene Ontology (GO) databases"
- [readme] Gene subset selection by subcellular location, molecular function and involving biological process: "Gene subset selection by subcellular location, molecular function and involving biological process"
