---
name: communication-relevance-label-classification
description: Use when a gene list (e.g., the 16,000 HVGs from FindVariableFeatures(),
  variance, scanpy/scranpy and M3drop) must be filtered to genes involved in cell–cell
  communication, and no standardized communication-relevance labels exist yet.
when_to_use_negative:
- When the analysis goal is constructing the cell–cell communication network itself
  (CPPLS regression modeling) rather than labeling/filtering individual HVGs
- When genes already carry curated ligand–receptor or communication annotations from
  another source, making the manual GeneCards/GO tag query redundant
- When the gene list is not drawn from a spatial/single-cell HVG pipeline (e.g., it
  is already a feature table of cell-type-specific markers)
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: CPPLS-MLP
  role: Workflow whose HVG-filtering stage consumes these communication-relevance
    labels to filter out HVGs not involved in cell communication
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: GeneCards
  role: Source of predefined gene annotation tags (e.g., 'receptor', 'ligand') used
    to derive strong/weak communication labels
- name: Gene Ontology (GO)
  role: Source of predefined gene labels used alongside GeneCards for the two-class
    communication-relevance classification
- name: Python (>= 3.7)
  role: Implementation language for the search_gene labeling code and CSV output of
    gene-to-label mappings
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:d3a637d7685d9352eae7e13fa3393d31cf219822addcfbc1bd0bd6cb4ffe9262
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - Gene Ontology (GO)
  - GeneCards
  - Python
  - Python (>= 3.7)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# communication-relevance-label-classification

## Summary

Classifies candidate highly variable genes (HVGs) as communication-relevant or not by querying GeneCards and Gene Ontology (GO) for annotation tags (11 total), assigning 'strong_com' or 'weak_com' labels and a binary strong-correlation indicator used by CPPLS-MLP to filter out genes not involved in cell communication.

## When to use

Use when a gene list (e.g., the 16,000 HVGs from FindVariableFeatures(), variance, scanpy/scranpy and M3drop) must be filtered to genes involved in cell–cell communication, and no standardized communication-relevance labels exist yet.

## When NOT to use

- When the analysis goal is constructing the cell–cell communication network itself (CPPLS regression modeling) rather than labeling/filtering individual HVGs
- When genes already carry curated ligand–receptor or communication annotations from another source, making the manual GeneCards/GO tag query redundant
- When the gene list is not drawn from a spatial/single-cell HVG pipeline (e.g., it is already a feature table of cell-type-specific markers)

## Inputs

- List of candidate HVGs (16,000 genes selected via FindVariableFeatures(), variance, scanpy/scranpy and M3drop)
- GeneCards database annotations per gene
- Gene Ontology (GO) database annotations per gene

## Outputs

- hvg_communication_labels.csv mapping each gene to a label in {strong_com, weak_com}
- Binary indicator column (1 = strong communication label, 0 = unlabeled)

## How to apply

For each HVG in the candidate list, query GeneCards and Gene Ontology (GO) to retrieve its recorded annotation tags, which span 11 tags including 'receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion' and 'intercellular interaction'. Assign the label 'strong_com' if the gene carries at least one of the six directly communication-related tags ('receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion', 'intercellular interaction'); assign 'weak_com' if it carries only function-related tags such as 'surfaces of many cells and extracellular matrices', 'Participates in cellular', 'Pathway', 'regulation' or 'signal transduction'. Under the strong-correlation criterion, set a binary indicator of 1 for genes with a strong label and 0 otherwise, then write the gene-to-label mapping to hvg_communication_labels.csv. Note the article describes this as a manual database-querying process; the repository provides a search_gene code to apply the same criteria to an entered gene list.

## Related tools

- **CPPLS-MLP** (Workflow whose HVG-filtering stage consumes these communication-relevance labels to filter out HVGs not involved in cell communication) — https://github.com/wuzhenao/CPPLS-MLP
- **GeneCards** (Source of predefined gene annotation tags (e.g., 'receptor', 'ligand') used to derive strong/weak communication labels)
- **Gene Ontology (GO)** (Source of predefined gene labels used alongside GeneCards for the two-class communication-relevance classification)
- **Python (>= 3.7)** (Implementation language for the search_gene labeling code and CSV output of gene-to-label mappings) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
run search_gene  # after entering the HVG gene list, per the CPPLS-MLP README step (4)
```

## Evaluation signals

- hvg_communication_labels.csv exists and contains a label column whose values fall within {strong, weak}
- A binary indicator column matches the strong-correlation criterion exactly (1 for strongly labeled genes, 0 for unlabeled genes)
- Every gene in the input HVG list appears exactly once in the output CSV mapping
- Labeling is grounded in retrievable GeneCards/GO tags; the six strong tags ('receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion', 'intercellular interaction') are the only triggers for label 1
- Downstream sanity range: the paper's pipeline yielded 354 communication-related genes on seqFISH+ and 345 on Seq-Scope, so filtered counts should be in a comparable magnitude, not thousands

## Limitations

- Label extraction is a manual database-querying process with no scripted or automated procedure described in the article
- The article notes there is no standardized classification criterion to determine whether a gene is associated with cell communication; this skill's 11-tag scheme is the authors' proposed criterion
- Coverage depends entirely on GeneCards and GO annotations — unannotated genes default to unlabeled (0), and database tag inconsistencies propagate into labels

## Evidence

- [methods] categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO): "categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO)"
- [discussion] labels genes in a known database to filter out HVGs that are not involved in cell communication: "labels genes in a known database to filter out HVGs that are not involved in cell communication"
- [discussion] By manually querying genes in databases like Genecard and GO and extracting labels: "By manually querying genes in databases like Genecard and GO and extracting labels"
- [readme] Enter the list of genes and then run the search_gene to get the results under this criterion.: "Enter the list of genes and then run the search_gene to get the results under this criterion."
