---
name: cross-dataset-gene-classification-benchmarking
description: Use when you need to classify which of the top-2000 Seurat FindVariableFeatures()
  HVGs participate in cell-cell communication, and compare the classified gene counts
  across independent spatial transcriptomics datasets.
when_to_use_negative:
- The gene list has not first been restricted to HVGs involved in the MIMO communication
  network — the criterion applies only to network-participating HVGs, not all genes
- Your labels come from an automated annotation pipeline rather than manual GeneCards/GO
  querying — the article's criterion is defined by strong-com database labels, so
  different label sources break comparability
- You need an MLP with documented architecture, hyperparameters, or training procedure
  — the article does not provide these implementation details
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CPPLS-MLP
  role: Integrated method whose MLP component performs the binary classification of
    HVGs as communication-related
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Python
  role: Runtime (>= 3.7) in which the MLP classification notebooks are executed
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Seurat
  role: Source of the top-2000 HVG input feature set via FindVariableFeatures()
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: GeneCards
  role: External database queried to assign binary communication-relevance labels
    to genes
- name: Gene Ontology (GO)
  role: External database queried to assign binary communication-relevance labels
    to genes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:5db055f606e346bf3d2d0a8577c42e20dc1523546771c0b1dcdce87e98838995
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cross-dataset-gene-classification-benchmarking

## Summary

Classifies highly variable genes (HVGs) as cell-communication-related using a binary label criterion (receptor, ligand, cell-cell adhesion, intercellular interaction from GeneCards/GO) and an MLP classifier applied to HVGs involved in the MIMO communication network, benchmarked across two spatial transcriptomics datasets (seqFISH+ mouse cortex and Seq-Scope mouse colon).

## When to use

Use when you need to classify which of the top-2000 Seurat FindVariableFeatures() HVGs participate in cell-cell communication, and compare the classified gene counts across independent spatial transcriptomics datasets.

## When NOT to use

- The gene list has not first been restricted to HVGs involved in the MIMO communication network — the criterion applies only to network-participating HVGs, not all genes
- Your labels come from an automated annotation pipeline rather than manual GeneCards/GO querying — the article's criterion is defined by strong-com database labels, so different label sources break comparability
- You need an MLP with documented architecture, hyperparameters, or training procedure — the article does not provide these implementation details

## Inputs

- Top 2000 HVGs per dataset extracted with Seurat FindVariableFeatures() (packaged in compare_hvg.zip)
- Gene communication-relevance labels from GeneCards and Gene Ontology (GO) queries (strong-com tags: 'receptor', 'ligand', 'cell-cell adhesion', 'intercellular interaction')
- HVGs involved in the MIMO cell-communication network from the upstream CPPLS regression and MIMO-graph step (task_003)
- seqFISH+ mouse cortex and Seq-Scope mouse colon spatial transcriptomics datasets

## Outputs

- hvg_communication_classification.csv with per-cell-type rows of (gene, class label, classification score in [0, 1])
- Per-dataset counts of communication-related HVGs (354 for seqFISH+, 345 for Seq-Scope)
- Per-cell-type HVG overlap ratios (0.31 Seq-Scope, 0.25 seqFISH+)

## How to apply

Load the top 2000 HVGs extracted with Seurat's FindVariableFeatures() from each dataset (compare_hvg.zip in the CPPLS-MLP GitHub repo packages these for seqFISH+ mouse cortex and Seq-Scope mouse colon). Assign binary communication-relevance labels by querying GeneCards and Gene Ontology: genes tagged 'receptor', 'ligand', 'cell-cell adhesion', or 'intercellular interaction' receive label 1, all others 0 — this manual database-query step has no scripted procedure. Restrict classification to only the HVGs involved in the MIMO cell-communication network produced by the upstream CPPLS regression and MIMO-graph construction step. Train the MLP (Python >= 3.7) on the labeled genes using the strong-correlation labels as targets, then predict communication-related class membership per HVG per cell type and write gene, class label, and score to hvg_communication_classification.csv. Benchmark across datasets by comparing the number of communication-related genes (354 for seqFISH+ vs 345 for Seq-Scope) and the per-cell-type HVG overlap ratios (0.25 seqFISH+, 0.31 Seq-Scope).

## Related tools

- **CPPLS-MLP** (Integrated method whose MLP component performs the binary classification of HVGs as communication-related) — https://github.com/wuzhenao/CPPLS-MLP
- **Python** (Runtime (>= 3.7) in which the MLP classification notebooks are executed) — https://github.com/wuzhenao/CPPLS-MLP
- **Seurat** (Source of the top-2000 HVG input feature set via FindVariableFeatures()) — https://github.com/wuzhenao/CPPLS-MLP
- **GeneCards** (External database queried to assign binary communication-relevance labels to genes)
- **Gene Ontology (GO)** (External database queried to assign binary communication-relevance labels to genes)

## Examples

```
cd divide && jupyter nbconvert --to notebook --execute Seqfish+_Find_var.ipynb
```

## Evaluation signals

- hvg_communication_classification.csv contains an explicit class-label column and per-gene scores constrained to [0, 1]
- Row counts of class-1 genes reproduce the reported 354 (seqFISH+) and 345 (Seq-Scope) communication-related genes
- Classification is restricted to HVGs present in the MIMO communication network, not the full top-2000 HVG list
- Per-cell-type HVG overlap ratios fall near the reported 0.25 (seqFISH+) and 0.31 (Seq-Scope)
- Every labeled gene traces to a GeneCards or GO strong-com tag ('receptor', 'ligand', 'cell-cell adhesion', 'intercellular interaction')

## Limitations

- Label extraction is a manual database-querying process with no scripted or automated procedure described
- No MLP implementation details (architecture, hyperparameters, training procedure) are given in the article
- Only two benchmark datasets (seqFISH+ and Seq-Scope) were used, limiting generalizability
- No accession IDs or download sources are provided for the Seq-Scope and seqFISH+ datasets

## Evidence

- [methods] CPPLS-MLP identified 354 and 345 cell-communication-related genes on the seqFISH+ and Seq-Scope datasets respectively: "our CPPLS-MLP method identified 354 and 345 genes related to cell communication, respectively"
- [methods] Classification labels derive from GeneCards and GO strong-com tags: "categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO)"
- [methods] Top 2000 HVG filtering via FindVariableFeatures() defines the candidate gene set: "CPPLS-MLP initially filters the top 2000 HVGs using the FindVariableFeatures() method"
- [readme] MLP classification results are reproducible from the divide folder notebooks in the repository: "If you want to reproduce the results of MLP's classification of genes, you can get it in the divide folder"
- [readme] compare_hvg.zip packages the HVG extraction with four methods including FindVariableFeatures(): "we extracted the HVGs on the dataset based on four methods: FindVariableFeatures(), variance, scannpy, and M3drop"
- [methods] Average per-cell-type HVG overlap ratio reported per dataset: "the average overlap ratio of HVGs between these cell types was 0.31"
