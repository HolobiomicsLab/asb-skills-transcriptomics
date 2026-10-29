---
name: cross-database-gene-label-mapping
description: Use when you have a candidate HVG list (e.g. ~16,000 genes from FindVariableFeatures(),
  variance, scannpy, M3drop) and need database-derived communication-relevance labels
  to retain only genes involved in cell-cell communication.
when_to_use_negative:
- The input is raw expression data or spatial coordinates rather than an already-computed
  HVG candidate list — labeling presupposes genes from the HVG selection step
- The organism or gene symbols are absent from GeneCards/GO, since the skill depends
  entirely on predefined database labels rather than expression-based inference
- You need de novo identification of communication genes from data; this skill only
  filters using predefined database annotations, which the article notes lack a standardized
  classification criterion
edam_operation: http://edamontology.org/operation_2422
tools:
- name: Python
  role: Runs the search_gene lookup script that applies the labeling criterion to
    a gene list
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: GeneCards
  role: Source database providing predefined gene annotation tags (e.g. 'receptor',
    'ligand') used for labeling
- name: Gene Ontology (GO)
  role: Source database providing functional annotation tags used to distinguish strong_com
    from weak_com labels
- name: CPPLS-MLP
  role: Overall workflow in which the label filter removes HVGs not involved in cell
    communication
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:6953af4e7b29b9afa43c1ceb7c95daaed0d9a2196296741f40acf47c55bef558
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - Gene Ontology (GO)
  - GeneCards
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-database-gene-label-mapping

## Summary

Labels each HVG as communication-related by cross-referencing GeneCards and Gene Ontology (GO) annotation tags, assigning strong_com, weak_com, or unlabeled classes that are then binarized (1/0) to filter HVGs not involved in cell communication.

## When to use

Use when you have a candidate HVG list (e.g. ~16,000 genes from FindVariableFeatures(), variance, scannpy, M3drop) and need database-derived communication-relevance labels to retain only genes involved in cell-cell communication.

## When NOT to use

- The input is raw expression data or spatial coordinates rather than an already-computed HVG candidate list — labeling presupposes genes from the HVG selection step
- The organism or gene symbols are absent from GeneCards/GO, since the skill depends entirely on predefined database labels rather than expression-based inference
- You need de novo identification of communication genes from data; this skill only filters using predefined database annotations, which the article notes lack a standardized classification criterion

## Inputs

- HVG list produced by FindVariableFeatures(), variance, scannpy and M3drop (16,000 HVGs across the seqFISH+ and Seq-Scope datasets)
- GeneCards annotation records per gene
- Gene Ontology (GO) annotation records per gene

## Outputs

- hvg_communication_labels.csv containing a gene-to-label mapping with a label column in {strong, weak} and a binary indicator column (1 = strong/labeled, 0 = unlabeled)
- Set of communication-related HVGs (CPPLS-MLP identified 354 on seqFISH+ and 345 on Seq-Scope)

## How to apply

Load the HVG list (in CPPLS-MLP this is the 16,000 HVGs from task_004) and query GeneCards and Gene Ontology (GO) for each gene's recorded annotation tags, of which 11 tags are used. Assign strong_com to genes carrying at least one of the six directly communication-related tags ('receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion', 'intercellular interaction'), and weak_com to genes carrying only function-related tags such as 'surfaces of many cells and extracellular matrices', 'Participates in cellular', 'Pathway', 'regulation', or 'signal transduction'. Under the strong-correlation criterion, set a binary indicator to 1 for genes with a strong label and 0 for genes without one. Write the gene-to-label mapping to hvg_communication_labels.csv; the repo's search_gene script automates this lookup once a gene list is supplied.

## Related tools

- **Python** (Runs the search_gene lookup script that applies the labeling criterion to a gene list) — https://github.com/wuzhenao/CPPLS-MLP
- **GeneCards** (Source database providing predefined gene annotation tags (e.g. 'receptor', 'ligand') used for labeling)
- **Gene Ontology (GO)** (Source database providing functional annotation tags used to distinguish strong_com from weak_com labels)
- **CPPLS-MLP** (Overall workflow in which the label filter removes HVGs not involved in cell communication) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
search_gene(hvg_list)  # enter the HVG list from compare_hvg.zip; output: communication labels under the strong/weak criterion
```

## Evaluation signals

- hvg_communication_labels.csv exists and contains a label column whose values fall within {strong, weak}
- A binary indicator column matches the strong-correlation criterion: 1 for labeled/strong genes, 0 for unlabeled genes
- Every gene in the input 16,000-HVG list appears exactly once in the output mapping (no dropped or duplicated gene keys)
- Count of strong-labeled genes is in the same order as the article's reported hits (354 genes on seqFISH+, 345 on Seq-Scope)
- Each labeled gene can be traced to at least one of the 11 GeneCards/GO tags, with strong labels restricted to the six communication-related tags

## Limitations

- Label extraction is a manual database-querying process with no scripted or automated procedure described in the article; the README's search_gene script only partially automates it
- There is no standardized classification criterion to determine whether a gene is associated with cell communication, so labels are only as good as GeneCards/GO coverage and the chosen 11 tags
- The criterion was developed and evaluated on only two datasets (seqFISH+ mouse cortex and Seq-Scope mouse colon), so generalization to other tissues/species is untested

## Evidence

- [other] Assign the label strong_com to genes carrying at least one of the six directly communication-related tags ('receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion', 'intercellular interaction'): [evidence span withheld — anchor status: not_found]
- [readme] readme search_gene: "Enter the list of genes and then run the search_gene to get the results under this criterion."
- [discussion] discussion filter rationale: "labels genes in a known database to filter out HVGs that are not involved in cell communication"
- [discussion] discussion manual querying: "By manually querying genes in databases like Genecard and GO and extracting labels"
- [methods] methods database classes: "categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO)"
