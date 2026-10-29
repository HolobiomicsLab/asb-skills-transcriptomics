---
name: hvg-annotation-filtering
description: Use when a list of candidate HVGs (e.g. 16,000 genes from FindVariableFeatures(),
  variance, scannpy, M3drop) must be narrowed to genes with database-supported roles
  in cell communication before downstream modeling.
when_to_use_negative:
- The input is not a gene/HVG list but raw expression counts or spatial coordinates
  — annotation filtering applies only after HVGs have been identified
- A data-driven, expression-based criterion for communication-associated genes is
  needed; the paper explicitly notes it lacks a standardized classification criterion
  beyond database labels
- Automated, reproducible label extraction is required — the article states label
  extraction is a manual database-querying process with no scripted procedure described
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3308
tools:
- name: CPPLS-MLP
  role: Hosts the gene-labeling workflow; its search_gene script queries a gene list
    and returns genes involved in cellular communication under the defined criteria
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: GeneCards
  role: Source of predefined gene annotation tags (e.g. 'receptor', 'ligand', 'cell–cell
    adhesion') used to assign communication labels
- name: Gene Ontology (GO)
  role: Source of predefined gene annotation tags used alongside GeneCards to label
    genes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:273e317fa988430a205353a87075f8b5daa67d79510964a164761e030de07656
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - Gene Ontology (GO)
  - GeneCards
  - Python
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hvg-annotation-filtering

## Summary

Filters candidate HVGs by querying GeneCards and Gene Ontology annotations to assign communication-relevance labels (strong_com/weak_com) and a binary indicator, retaining only genes plausibly involved in cell-cell communication.

## When to use

Use when a list of candidate HVGs (e.g. 16,000 genes from FindVariableFeatures(), variance, scannpy, M3drop) must be narrowed to genes with database-supported roles in cell communication before downstream modeling.

## When NOT to use

- The input is not a gene/HVG list but raw expression counts or spatial coordinates — annotation filtering applies only after HVGs have been identified
- A data-driven, expression-based criterion for communication-associated genes is needed; the paper explicitly notes it lacks a standardized classification criterion beyond database labels
- Automated, reproducible label extraction is required — the article states label extraction is a manual database-querying process with no scripted procedure described

## Inputs

- HVG list produced by upstream variable-feature selection (e.g. 16,000 HVGs from FindVariableFeatures(), variance, scannpy and M3drop)
- GeneCards annotation records per gene
- Gene Ontology (GO) annotation records per gene

## Outputs

- hvg_communication_labels.csv containing a gene-to-label mapping with a label column (values within {strong, weak}) and a binary indicator column (1 for strongly labeled, 0 for unlabeled)
- Filtered set of genes deemed involved in cell communication (354 and 345 communication-related genes identified on the seqFISH+ and Seq-Scope datasets respectively)

## How to apply

Load the HVG list (e.g. the 16,000 HVGs selected across both datasets by FindVariableFeatures(), variance, scannpy and M3drop) and, for each gene, query GeneCards and Gene Ontology (GO) to retrieve its recorded annotation tags (11 tags in total). Assign strong_com to genes carrying at least one of the six directly communication-related tags ('receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion', 'intercellular interaction'), and weak_com to genes carrying only function-related tags such as 'surfaces of many cells and extracellular matrices', 'Participates in cellular', 'Pathway', 'regulation' or 'signal transduction'. Under the strong-correlation criterion, set the binary indicator to 1 for strongly labeled genes and 0 otherwise, then write the mapping to hvg_communication_labels.csv. Rationale: the paper lacks a standardized classification criterion for communication-associated genes, so predefined database labels serve as that criterion; the repository implements this via a search_gene script that takes a gene list and returns genes involved in cellular communication under the criteria.

## Related tools

- **CPPLS-MLP** (Hosts the gene-labeling workflow; its search_gene script queries a gene list and returns genes involved in cellular communication under the defined criteria) — https://github.com/wuzhenao/CPPLS-MLP
- **GeneCards** (Source of predefined gene annotation tags (e.g. 'receptor', 'ligand', 'cell–cell adhesion') used to assign communication labels)
- **Gene Ontology (GO)** (Source of predefined gene annotation tags used alongside GeneCards to label genes)

## Examples

```
python search_gene.py --genes hvg_list.csv  # outputs labels under the strong/weak communication criterion
```

## Evaluation signals

- hvg_communication_labels.csv exists and contains a label column whose values fall within {strong, weak}
- A binary indicator column matches the strong-correlation criterion: 1 for genes with a strong label, 0 for unlabeled genes
- Only the 11 annotation tags enumerated by the paper (6 strong, 5 weak/function-related) drive the labels — no other tag types appear
- Every input HVG appears exactly once in the output mapping (complete gene-to-label coverage)
- Downstream, the number of genes retained as communication-related should be in the same order as the paper's 354 (seqFISH+) and 345 (Seq-Scope) communication-related genes

## Limitations

- Label extraction is a manual database-querying process with no scripted or automated procedure described in the article (the repo's search_gene code only applies the criteria, not the querying)
- Only two benchmark datasets (seqFISH+ and Seq-Scope) were used to assess the labeling/filtering pipeline
- There is no standardized classification criterion in the field to determine whether a gene is associated with cell communication; the database-tag scheme is one proposal and may miss communication genes lacking receptor/ligand tags

## Evidence

- [discussion] labels genes in a known database to filter out HVGs that are not involved in cell communication: "labels genes in a known database to filter out HVGs that are not involved in cell communication"
- [methods] categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO): "categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO)"
- [readme] Enter the list of genes and then run the search_gene to get the results under this criterion.: "Enter the list of genes and then run the search_gene to get the results under this criterion."
- [methods] our CPPLS-MLP method identified 354 and 345 genes related to cell communication, respectively: "our CPPLS-MLP method identified 354 and 345 genes related to cell communication, respectively"
