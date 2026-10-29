---
name: gene-database-label-extraction
description: Use when a list of candidate HVGs (e.g. from FindVariableFeatures(),
  variance, scannpy, M3drop) must be classified as communication-related or not, using
  database-derived gene tags as the labeling criterion.
when_to_use_negative:
- When the goal is to classify genes by CPPLS-MLP's own model output rather than by
  database annotation — that is a separate MLP classification step (divide folder
  notebooks), not database label extraction
- When no curated gene-annotation source (GeneCards/GO) is reachable, since the article
  notes label extraction was a manual database-querying process with no automated
  scripted procedure beyond the search_gene code
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: CPPLS-MLP
  role: Pipeline whose gene-labeling filter consumes the database-derived strong/weak
    labels to filter out HVGs not involved in cell communication
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: GeneCards
  role: Source of predefined gene labels (e.g. receptor, ligand, cell–cell adhesion)
    used to classify communication relevance
- name: Gene Ontology (GO)
  role: Source of predefined gene labels used alongside GeneCards for the strong_com/weak_com
    classification
- name: Python
  role: Implementation language for the search_gene labeling code provided in the
    repository
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:9cdaa3f77c62e7adb68af1980d67bb3311cc2497d7b5e10269a3d58e05f009ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CCPLS
  - CPPLS-MLP
  - Gene Ontology (GO)
  - GeneCards
  - Python
  - R
  - Seurat
  - pls
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gene-database-label-extraction

## Summary

Assigns communication-relevance labels to candidate HVGs by querying GeneCards and Gene Ontology annotations: genes carrying one of six directly communication-related tags are labeled strong_com (value 1), genes with only function-related tags are labeled weak_com (value 0). This filter removes HVGs not involved in cell communication before CPPLS-MLP analysis.

## When to use

Use when a list of candidate HVGs (e.g. from FindVariableFeatures(), variance, scannpy, M3drop) must be classified as communication-related or not, using database-derived gene tags as the labeling criterion.

## When NOT to use

- When the goal is to classify genes by CPPLS-MLP's own model output rather than by database annotation — that is a separate MLP classification step (divide folder notebooks), not database label extraction
- When no curated gene-annotation source (GeneCards/GO) is reachable, since the article notes label extraction was a manual database-querying process with no automated scripted procedure beyond the search_gene code

## Inputs

- List of candidate HVGs (gene symbols), e.g. 16,000 HVGs from FindVariableFeatures(), variance, scannpy and M3drop
- GeneCards annotation records for each gene
- Gene Ontology (GO) annotation records for each gene

## Outputs

- hvg_communication_labels.csv: gene-to-label mapping with a label column in {strong, weak} and a binary strong-correlation indicator (1/0)
- Filtered HVG set restricted to communication-relevant (strong_com) genes

## How to apply

Take the HVG list (e.g. the 16,000 HVGs extracted across seqFISH+ and Seq-Scope) and for each gene query GeneCards and GO to retrieve one of 11 recorded annotation tags. Assign strong_com when a gene carries at least one of the six directly communication-related tags ('receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion', 'intercellular interaction'), and weak_com when it carries only function-related tags such as 'Pathway', 'regulation' or 'signal transduction'. Under the strong-correlation criterion encode labeled genes as 1 and unlabeled as 0; the repo's search_gene code applies this criterion to a full gene list. The binary indicator is then used to filter out HVGs not involved in cell communication before CPPLS-MLP stability scoring.

## Related tools

- **CPPLS-MLP** (Pipeline whose gene-labeling filter consumes the database-derived strong/weak labels to filter out HVGs not involved in cell communication) — https://github.com/wuzhenao/CPPLS-MLP
- **GeneCards** (Source of predefined gene labels (e.g. receptor, ligand, cell–cell adhesion) used to classify communication relevance)
- **Gene Ontology (GO)** (Source of predefined gene labels used alongside GeneCards for the strong_com/weak_com classification)
- **Python** (Implementation language for the search_gene labeling code provided in the repository) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
Enter the list of genes and then run the search_gene to get the results under this criterion
```

## Evaluation signals

- hvg_communication_labels.csv exists and its label column contains only values in {strong, weak}
- The binary indicator column matches the strong-correlation criterion exactly: 1 for genes labeled 'receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion' or 'intercellular interaction', 0 otherwise
- Every input gene has exactly one label row (mapping covers the full HVG list)
- Downstream Y/N HVG splits (communication-related vs non-related genes per cell type) can be reproduced from the indicator column

## Limitations

- The article describes the labeling as a manual database-querying process with no fully scripted or automated extraction procedure
- There is no standardized classification criterion in the field for whether a gene is associated with cell communication; the strong/weak tag sets are the authors' own criterion
- Gene labeling depends on the completeness and currency of GeneCards/GO annotations, which may miss genes with uncharacterized communication roles

## Evidence

- [discussion] labels genes in a known database to filter out HVGs that are not involved in cell communication: "labels genes in a known database to filter out HVGs that are not involved in cell communication"
- [methods] predefined labels from databases such as GeneCards and Gene Ontology (GO): "categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO)"
- [discussion] By manually querying genes in databases like Genecard and GO and extracting labels: "By manually querying genes in databases like Genecard and GO and extracting labels"
- [readme] Enter the list of genes and then run the search_gene to get the results under this criterion: "Enter the list of genes and then run the search_gene to get the results under this criterion"
