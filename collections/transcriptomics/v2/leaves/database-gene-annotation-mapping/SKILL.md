---
name: database-gene-annotation-mapping
description: Use when a gene list (e.g., top 2000 HVGs from Seurat's FindVariableFeatures()
  on seqFISH+ or Seq-Scope data) needs binary communication-relevance labels to train
  or evaluate a gene classifier.
when_to_use_negative:
- The genes already carry communication-relevance labels from a prior annotation pass
  — re-querying adds no information
- The goal is classifying which HVGs are communication-related from expression/spatial
  data (that is the MLP classification step; this skill only supplies the ground-truth
  labels)
- Genes of interest lack entries in GeneCards or GO, since the labeling criterion
  depends entirely on predefined labels from those databases
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: GeneCards
  role: Source of predefined functional labels used to decide whether a gene is a
    strong-com (communication-related) gene
- name: Gene Ontology (GO)
  role: Second annotation database queried for predefined labels; combined with GeneCards
    to assign the binary communication-relevance criterion
- name: CPPLS-MLP
  role: Provides the search_gene script that applies the communication criterion to
    an input gene list
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:c48a235ad4d6b98843bae2278d96ee91dd5deb0b556da0b90ae23e5ba210959a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - Gene Ontology (GO)
  - GeneCards
  - Python
  - Seurat
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# database-gene-annotation-mapping

## Summary

Assigns binary communication-relevance labels to genes by manually querying GeneCards and Gene Ontology (GO): genes tagged with strong-com terms ('receptor', 'ligand', 'cell-cell adhesion', 'intercellular interaction') receive label 1, all others 0. These labels serve as training targets for the CPPLS-MLP classifier and as the standardized criterion for what counts as a cell-communication gene.

## When to use

Use when a gene list (e.g., top 2000 HVGs from Seurat's FindVariableFeatures() on seqFISH+ or Seq-Scope data) needs binary communication-relevance labels to train or evaluate a gene classifier.

## When NOT to use

- The genes already carry communication-relevance labels from a prior annotation pass — re-querying adds no information
- The goal is classifying which HVGs are communication-related from expression/spatial data (that is the MLP classification step; this skill only supplies the ground-truth labels)
- Genes of interest lack entries in GeneCards or GO, since the labeling criterion depends entirely on predefined labels from those databases

## Inputs

- List of HVG gene symbols (e.g., top 2000 HVGs from Seurat FindVariableFeatures() on the seqFISH+ mouse cortex or Seq-Scope mouse colon datasets)
- GeneCards query results with predefined functional labels per gene
- Gene Ontology (GO) annotations per gene

## Outputs

- Binary communication-relevance label per gene (1 = strong-com: receptor/ligand/cell-cell adhesion/intercellular interaction; 0 = otherwise)
- Labeled gene table feeding MLP training as strong-correlation targets

## How to apply

Take the HVG list produced upstream (CPPLS-MLP restricts to the top 2000 HVGs obtained via Seurat's FindVariableFeatures() with default threshold 2000, and to genes participating in the MIMO cell-communication network). For each gene, manually query GeneCards and GO and extract predefined functional labels; assign label 1 if the gene carries a strong-com tag — 'receptor', 'ligand', 'cell-cell adhesion', or 'intercellular interaction' — otherwise 0. The CPPLS-MLP repository provides a helper script (search_gene) that applies this criterion to a whole gene list at once, so the labels can be produced either by hand or by entering the gene list and running search_gene. The resulting binary labels are the strong-correlation training targets for the MLP classification step and the reference criterion against which predicted communication-related genes (354 on seqFISH+, 345 on Seq-Scope) are judged.

## Related tools

- **GeneCards** (Source of predefined functional labels used to decide whether a gene is a strong-com (communication-related) gene)
- **Gene Ontology (GO)** (Second annotation database queried for predefined labels; combined with GeneCards to assign the binary communication-relevance criterion)
- **CPPLS-MLP** (Provides the search_gene script that applies the communication criterion to an input gene list) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
search_gene(gene_list)
```

## Evaluation signals

- Every gene in the input list receives exactly one label in {0, 1}; no missing or non-binary labels
- Genes labeled 1 carry at least one strong-com term ('receptor', 'ligand', 'cell-cell adhesion', 'intercellular interaction') verifiable by a manual GeneCards/GO spot-check
- Restricting labels to HVGs involved in the MIMO network and applying the MLP classifier reproduces per-dataset counts of communication-related genes on the order of 354 (seqFISH+) and 345 (Seq-Scope)
- Running search_gene on the same gene list returns a label for every entered gene, matching the manual-query result

## Limitations

- Label extraction is a manual database-querying process with no scripted or automated procedure fully described in the article; search_gene partially automates it but coverage of GeneCards/GO terms is implicit
- There is no standardized classification criterion beyond the database labels, so the binary labeling depends on the chosen strong-com term list
- Only two benchmark datasets (seqFISH+ and Seq-Scope) were used, so label counts and their downstream classification results are validated on limited data

## Evidence

- [methods] categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO): "categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO)"
- [discussion] labels genes in a known database to filter out HVGs that are not involved in cell communication: "labels genes in a known database to filter out HVGs that are not involved in cell communication"
- [discussion] By manually querying genes in databases like Genecard and GO and extracting labels: "By manually querying genes in databases like Genecard and GO and extracting labels"
- [readme] Enter the list of genes and then run the search_gene to get the results under this criterion.: "Enter the list of genes and then run the search_gene to get the results under this criterion."
