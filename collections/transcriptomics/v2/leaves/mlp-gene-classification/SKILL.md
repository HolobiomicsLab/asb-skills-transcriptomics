---
name: mlp-gene-classification
description: Use when spatial transcriptomics HVGs (e.g. top 2000 from Seurat FindVariableFeatures())
  restricted to a MIMO cell-communication network must be assigned binary communication-relevance
  labels and scored, in the CPPLS-MLP pipeline after network construction.
when_to_use_negative:
- No MIMO communication network has been constructed yet — the CPPLS regression and
  MIMO-graph step (upstream in the CPPLS-MLP pipeline) must run first, since classification
  is restricted to network-involved HVGs
- The analysis goal is constructing the cell-cell communication network itself (CPPLS
  regression on RMD files), not classifying genes against that network
- The classification criterion must be based on something other than GeneCards/GO
  strong-communication database labels — the article notes these methods 'lack a standardized
  classification criterion' beyond such database labeling
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CPPLS-MLP
  role: Integrated pipeline providing the MIMO network-restricted HVG set and the
    divide-folder notebooks (e.g. Seqfish+_Find_var.ipynb) that run MLP classification
    of genes
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Python
  role: Runtime for the MLP classification (Python >= 3.7) and the Seqfish+_Find_var
    notebooks in the divide folder
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Seurat
  role: Source of the top 2000 HVGs via FindVariableFeatures() that form the input
    gene set
- name: GeneCards
  role: Source of predefined strong-com labels ('receptor', 'ligand', 'cell-cell adhesion',
    'intercellular interaction') used to assign binary training targets
- name: Gene Ontology (GO)
  role: Second database supplying predefined labels for communication-relevance classification
    of genes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:097630fa960ff6d01bbc3c0a1b22deae347c68fe652529c11afa538155907438
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
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mlp-gene-classification

## Summary

Trains an MLP neural network (Python >= 3.7) on HVGs from the CPPLS-MLP MIMO communication network, using GeneCards/GO strong-communication labels ('receptor', 'ligand', 'cell-cell adhesion', 'intercellular interaction') as binary targets, to classify top-2000 HVGs as cell-communication-related on spatial transcriptomics datasets such as seqFISH+ mouse cortex and Seq-Scope mouse colon.

## When to use

Use when spatial transcriptomics HVGs (e.g. top 2000 from Seurat FindVariableFeatures()) restricted to a MIMO cell-communication network must be assigned binary communication-relevance labels and scored, in the CPPLS-MLP pipeline after network construction.

## When NOT to use

- No MIMO communication network has been constructed yet — the CPPLS regression and MIMO-graph step (upstream in the CPPLS-MLP pipeline) must run first, since classification is restricted to network-involved HVGs
- The analysis goal is constructing the cell-cell communication network itself (CPPLS regression on RMD files), not classifying genes against that network
- The classification criterion must be based on something other than GeneCards/GO strong-communication database labels — the article notes these methods 'lack a standardized classification criterion' beyond such database labeling

## Inputs

- Top 2000 HVGs from seqFISH+ mouse cortex and Seq-Scope mouse colon datasets (packaged in compare_hvg.zip from github.com/wuzhenao/CPPLS-MLP)
- Binary communication-relevance labels derived from GeneCards and Gene Ontology (GO) database queries
- Set of HVGs involved in the MIMO cell-communication network from the upstream CPPLS regression and MIMO-graph construction step

## Outputs

- hvg_communication_classification.csv with per-cell-type columns: gene, class label, classification score in [0, 1]
- Counts of communication-related genes per dataset (expected: 354 on seqFISH+, 345 on Seq-Scope)

## How to apply

Load the top 2000 HVGs extracted with Seurat's FindVariableFeatures() (default threshold 2000) from compare_hvg.zip of the CPPLS-MLP repository. Assign binary labels by querying GeneCards and Gene Ontology: genes tagged with strong-com labels ('receptor', 'ligand', 'cell-cell adhesion', 'intercellular interaction') receive 1, all others 0. Restrict training to the HVGs involved in the MIMO cell-communication network produced by the upstream CPPLS regression and MIMO-graph construction step, then train the MLP on these strong-correlation labels to exploit its nonlinear mapping capability for the classification task. Predict communication-related class membership per HVG within each cell type and record the classification score. Write per-cell-type results (gene, class label, score) to hvg_communication_classification.csv; the repo implements this in the 'divide' folder via the Seqfish+ _Find_var notebooks.

## Related tools

- **CPPLS-MLP** (Integrated pipeline providing the MIMO network-restricted HVG set and the divide-folder notebooks (e.g. Seqfish+_Find_var.ipynb) that run MLP classification of genes) — https://github.com/wuzhenao/CPPLS-MLP
- **Python** (Runtime for the MLP classification (Python >= 3.7) and the Seqfish+_Find_var notebooks in the divide folder) — https://github.com/wuzhenao/CPPLS-MLP
- **Seurat** (Source of the top 2000 HVGs via FindVariableFeatures() that form the input gene set)
- **GeneCards** (Source of predefined strong-com labels ('receptor', 'ligand', 'cell-cell adhesion', 'intercellular interaction') used to assign binary training targets)
- **Gene Ontology (GO)** (Second database supplying predefined labels for communication-relevance classification of genes)

## Examples

```
run Seqfish+_Find_var.ipynb  # in the divide folder of github.com/wuzhenao/CPPLS-MLP, after unzipping compare_hvg.zip
```

## Evaluation signals

- hvg_communication_classification.csv contains a class-label column alongside gene identifiers and scores
- Row counts are consistent with the reported 354 communication-related genes on seqFISH+ and 345 on Seq-Scope
- All classification scores fall within the [0, 1] range
- Training targets agree with GeneCards/GO strong-com labeling: 'receptor', 'ligand', 'cell-cell adhesion', 'intercellular interaction' genes labeled 1, others 0
- Classified genes are a subset of the top 2000 HVGs extracted with FindVariableFeatures() and restricted to the MIMO communication network

## Limitations

- No implementation details for the MLP (architecture, hyperparameters, training procedure) are given in the article
- Label extraction is a manual GeneCards/GO querying process with no scripted or automated procedure described
- Validated on only two datasets (seqFISH+ mouse cortex and Seq-Scope mouse colon), and no accession IDs or download sources are provided for them

## Evidence

- [methods] CPPLS-MLP identified 354 and 345 cell-communication-related genes on the seqFISH+ and Seq-Scope datasets respectively: "our CPPLS-MLP method identified 354 and 345 genes related to cell communication, respectively"
- [methods] Class labels are derived from GeneCards and GO database annotations: "categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO)"
- [methods] Input HVGs are the top 2000 selected by Seurat's FindVariableFeatures(): "CPPLS-MLP initially filters the top 2000 HVGs using the FindVariableFeatures() method"
- [readme] MLP classification results are reproducible from the divide folder of the repository: "If you want to reproduce the results of MLP's classification of genes, you can get it in the divide folder"
