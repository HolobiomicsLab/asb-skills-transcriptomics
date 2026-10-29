---
name: genecards-gene-annotation-querying
description: Use when a gene list (e.g. 16,000 HVGs from FindVariableFeatures(), variance,
  scranpy and M3drop) must be classified by database annotation into communication-related
  vs. function-only categories before downstream filtering.
when_to_use_negative:
- When genes have already been assigned communication labels (e.g. an existing labeled
  CSV) — re-querying duplicates the annotation step
- When the goal is filtering based on expression statistics (e.g. zero-expression
  genes or coefficient-equal-to-0 HVGs) rather than database annotation tags
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GeneCards
  role: Annotation database queried for predefined gene labels used to classify communication
    relevance
- name: Gene Ontology (GO)
  role: Annotation database queried for predefined gene labels alongside GeneCards
- name: Python
  role: Implementation language for the search_gene script and writing hvg_communication_labels.csv
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: CPPLS-MLP
  role: Pipeline whose HVG filtering step consumes the database-derived strong/weak
    communication labels
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:39966568730a881e389c07d71ca09b5f786cb567ee139301ff678e83878703ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - Gene Ontology (GO)
  - GeneCards
  - Python
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# genecards-gene-annotation-querying

## Summary

Queries GeneCards and Gene Ontology (GO) annotations for each highly variable gene (HVG) and maps recorded tags (e.g. 'receptor', 'ligand', 'cell–cell adhesion') to strong/weak communication-relevance labels, enabling filtering of HVGs not involved in cell communication in the CPPLS-MLP workflow.

## When to use

Use when a gene list (e.g. 16,000 HVGs from FindVariableFeatures(), variance, scranpy and M3drop) must be classified by database annotation into communication-related vs. function-only categories before downstream filtering.

## When NOT to use

- When genes have already been assigned communication labels (e.g. an existing labeled CSV) — re-querying duplicates the annotation step
- When the goal is filtering based on expression statistics (e.g. zero-expression genes or coefficient-equal-to-0 HVGs) rather than database annotation tags

## Inputs

- HVG list produced upstream by FindVariableFeatures(), variance, scranpy and M3drop (16,000 HVGs across both datasets)
- GeneCards database annotation records per gene
- Gene Ontology (GO) database annotation records per gene

## Outputs

- hvg_communication_labels.csv with a gene-to-label mapping (strong/weak) and a binary indicator column (1 = strong label, 0 = unlabeled)

## How to apply

For each gene in the input HVG list, query the GeneCards and GO databases to retrieve its recorded annotation tags (11 tags in total, including 'receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion' and 'intercellular interaction'). Assign the label strong_com to genes carrying at least one of the six directly communication-related tags, and weak_com to genes carrying only function-related tags such as 'surfaces of many cells and extracellular matrices', 'Participates in cellular', 'Pathway', 'regulation' or 'signal transduction'. Under the strong-correlation criterion, assign value 1 to genes with a strong label and 0 otherwise, then write the gene-to-label mapping to hvg_communication_labels.csv. Note that in the source work this label extraction is a manual database-querying process with no scripted or automated procedure described; the repository's search_gene code applies the criteria to a supplied gene list.

## Related tools

- **GeneCards** (Annotation database queried for predefined gene labels used to classify communication relevance)
- **Gene Ontology (GO)** (Annotation database queried for predefined gene labels alongside GeneCards)
- **Python** (Implementation language for the search_gene script and writing hvg_communication_labels.csv) — https://github.com/wuzhenao/CPPLS-MLP
- **CPPLS-MLP** (Pipeline whose HVG filtering step consumes the database-derived strong/weak communication labels) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
run search_gene
```

## Evaluation signals

- hvg_communication_labels.csv exists and contains a label column whose values fall within {strong, weak}
- The CSV contains a binary indicator column matching the strong-correlation criterion (1 for labeled, 0 for unlabeled)
- Genes labeled strong_com carry at least one of the six directly communication-related tags ('receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion', 'intercellular interaction')
- Genes labeled weak_com carry only function-related tags (e.g. 'Pathway', 'regulation', 'signal transduction') and none of the six direct tags
- Applied downstream, the paper's method identified 354 and 345 cell-communication-related genes on the seqFISH+ and Seq-Scope datasets respectively — a comparable order of magnitude is expected

## Limitations

- Label extraction is a manual database-querying process with no scripted or automated procedure described in the article
- There is no standardized classification criterion in the field to determine whether a gene is associated with cell communication; the six-tag scheme is this paper's own criterion
- The README's search_gene code applies the authors' specific criteria and requires the input gene list to be supplied by the user

## Evidence

- [methods] categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO): "categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO)"
- [discussion] labels genes in a known database to filter out HVGs that are not involved in cell communication: "labels genes in a known database to filter out HVGs that are not involved in cell communication"
- [discussion] By manually querying genes in databases like Genecard and GO and extracting labels: "By manually querying genes in databases like Genecard and GO and extracting labels"
- [readme] Enter the list of genes and then run the search_gene to get the results under this criterion.: "Enter the list of genes and then run the search_gene to get the results under this criterion."
