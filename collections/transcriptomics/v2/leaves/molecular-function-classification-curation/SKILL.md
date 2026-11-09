---
name: molecular-function-classification-curation
description: Use when building or extending a protein/gene annotation database for
  intercellular communication analysis and you need to reduce hundreds of UniProt
  molecular function terms into a tractable, user-filterable set of functional classes.
when_to_use_negative:
- When the analysis goal is interaction confidence scoring or interaction filtering
  rather than functional annotation — use the STRING credibility-score/action-mode
  curation workflow instead
- When subcellular localization alone suffices to filter candidates and molecular
  function grouping would add unnecessary complexity
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0089
tools:
- name: InterCellDB
  role: R package whose gene database stores the consolidated molecular function classifications
    and subcellular locations used for user-defined gene subset selection
  repo: https://github.com/ZJUDBlab/InterCellDB
- name: UniProt
  role: Source of the 132 'Molecular Function' classifications that are remapped into
    16 consolidated functional groups
- name: Gene Ontology (GO)
  role: Supplementary source of functional classification for gene-encoded proteins
- name: COMPARTMENTS
  role: Source of the 13 subcellular location parameters annotated alongside the functional
    classes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202200045
    title: 'InterCellDB: A User‐Defined Database for Inferring Intercellular Networks'
  source_content_hash: sha256:0fed42ed18bb0bdbb5db393437cfd389aec9ccdd91e6a247c04e0aeec920d632
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUDBlab/InterCellDB
  tools:
  - COMPARTMENTS
  - Gene Ontology (GO)
  - InterCellDB
  - R
  - STRING
  - UniProt
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# molecular-function-classification-curation

## Summary

Curation workflow that maps fine-grained UniProt 'Molecular Function' classifications into a small set of consolidated functional groups (e.g., cytokine, growth factor, receptor, transducer) and attaches them, together with COMPARTMENTS subcellular localization, as annotations to every gene-coded protein in an interaction database such as InterCellDB.

## When to use

Use when building or extending a protein/gene annotation database for intercellular communication analysis and you need to reduce hundreds of UniProt molecular function terms into a tractable, user-filterable set of functional classes.

## When NOT to use

- When the analysis goal is interaction confidence scoring or interaction filtering rather than functional annotation — use the STRING credibility-score/action-mode curation workflow instead
- When subcellular localization alone suffices to filter candidates and molecular function grouping would add unnecessary complexity

## Inputs

- Gene-coded protein database derived from Ensembl human (127,750 proteins) and mouse (21,291 proteins) protein lists converted via NCBI gene references
- UniProt 'Molecular Function' classifications for each protein
- Gene Ontology (GO) functional annotations (e.g., GO: 0006955 immune response)
- COMPARTMENTS database subcellular localization records

## Outputs

- Annotated gene database with 16 consolidated functional group labels per protein
- 13 subcellular location parameters per protein from COMPARTMENTS
- Function-annotated interaction sub-libraries (InterCellDB.exp, InterCellDB.know, InterCellDB.pred) supporting user-defined gene subset selection by molecular function

## How to apply

Start from the gene-coded protein database built by converting Ensembl-sourced proteins (human 127,750; mouse 21,291) into coding genes via NCBI references. Retrieve UniProt 'Molecular Function' classifications for each protein (supplemented with Gene Ontology terms) and remap the 132 UniProt classes into 16 consolidated functional groups: actin-binding and motor protein, antimicrobial, cytokine, enzyme, growth factor, hormone, hydrolase, ion channel, protein metabolism regulator, receptor, transcription regulator, transducer, transferase, translation regulator, vasoactive, and other. In parallel, annotate each protein with 13 subcellular location parameters extracted from COMPARTMENTS (extracellular region, plasma membrane, cytosol, cytoskeleton, peroxisome, lysosome, endoplasmic reticulum, Golgi apparatus, endosome, mitochondrion, cytoplasm, nucleus, other). Attach these functional and localization annotations to every protein–protein pair in the interaction sub-libraries (InterCellDB.exp, InterCellDB.know, InterCellDB.pred) so downstream users can restrict gene subsets by molecular function before inferring cell–cell interactions. The rationale is that consolidated functional classes make the user-defined gene-subset selection tractable while preserving biological meaning (e.g., focusing on cytokines or receptors for immune-response analyses).

## Related tools

- **InterCellDB** (R package whose gene database stores the consolidated molecular function classifications and subcellular locations used for user-defined gene subset selection) — https://github.com/ZJUDBlab/InterCellDB
- **UniProt** (Source of the 132 'Molecular Function' classifications that are remapped into 16 consolidated functional groups)
- **Gene Ontology (GO)** (Supplementary source of functional classification for gene-encoded proteins)
- **COMPARTMENTS** (Source of the 13 subcellular location parameters annotated alongside the functional classes)

## Examples

```
devtools::install_github("ZJUDBlab/InterCellDB")  # then use InterCellDB gene-subset selection by molecular function (release v0.9.2)
```

## Evaluation signals

- The assembled gene database contains 18,990 human genes and 20,938 mouse genes with full functional and localization annotations, matching the counts reported in the paper
- Every protein carries one of the 16 consolidated functional group labels (no unmapped UniProt classes remain)
- All 13 COMPARTMENTS subcellular location parameters are populated for each gene-coded protein
- Functional annotations are retrievable during InterCellDB gene-subset selection (e.g., restricting to immune response GO: 0006955 genes) and return biologically coherent candidates such as interleukins and their receptors

## Limitations

- The 16 consolidated groups are a coarse remapping of 132 UniProt classes; fine-grained functional distinctions are lost and users cannot recover original UniProt class granularity from the consolidated label alone
- Functional classification depends on the March 20, 2020 snapshot of Ensembl/NCBI sources and may not cover newly annotated proteins
- Users must rely on the package-release version 0.9.2 of InterCellDB for the tutorials and figshare data to reproduce the annotations

## Evidence

- [results] cellular localization from the COMPARTMENTS database and functional classification from the Uniprot and Gene Ontology (GO) databases: "we appended annotation information for each protein including cellular localization from the COMPARTMENTS database and functional classification from the Uniprot and Gene Ontology (GO) databases"
- [results] The gene database contained 18 990 human genes and 20 938 mouse genes with a variety of annotations: "The gene database contained 18 990 human genes and 20 938 mouse genes with a variety of annotations"
- [results] we first acquired the complete human (127 750 proteins) and mouse (21 291 proteins) protein data from the Ensembl genomes database: "we first acquired the complete human (127 750 proteins) and mouse (21 291 proteins) protein data from the Ensembl genomes database"
- [readme] InterCellDB integrates gene product properties (subcellular locations, molecular functions, involving biological process): "InterCellDB integrates not only protein-protein interaction database, but also several databases on interaction properties (type, direction, effect) as well as gene product properties (subcellular"
- [readme] Gene subset selection by subcellular location, molecular function and involving biological process: "- Gene subset selection by subcellular location, molecular function and involving biological process"
