---
name: cell-type-probability-expression-correlation
description: Use when you need marker genes for cell types absent from training data,
  given OnClass prediction matrices (pred_Y_all.released.npy) and single-cell expression
  data such as Tabula Muris Senis FACS cells.
when_to_use_negative:
- When the cell types of interest have no predicted probability vector available (OnClass
  model has not been run to produce pred_Y_all)
- When inputs are raw count matrices without OnClass cell-type probability scores
  — the method correlates expression against predicted probabilities, not raw annotations
  alone
- When trying to distinguish topologically identical unseen cell types (siblings with
  the same parents in the Cell Ontology graph)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0625
tools:
- name: OnClass
  role: Provides the predicted cell-type probability vectors (pred_Y_all) that are
    correlated with gene expression; implemented in Python
  repo: https://github.com/wangshenguiuc/OnClass
- name: run_marker_genes_identification.py
  role: Computes Spearman correlations between cell-type probability vectors and gene
    expression profiles to rank and select top 20 marker genes per cell type
  repo: https://github.com/wangshenguiuc/OnClass/blob/master/script/run_marker_genes_identification.py
- name: run_marker_gene_based_prediction.py
  role: Builds marker-gene-based cell type predictions from the identified markers
    (Figure 5d,e,f, Extended Data Figure 7)
  repo: https://github.com/wangshenguiuc/OnClass/blob/master/script/run_marker_gene_based_prediction.py
- name: run_generate_pretrained_model.py
  role: Generates the intermediate prediction matrices (pred_Y_seen.released.npy,
    pred_Y_all.released.npy) used as input
  repo: https://github.com/wangshenguiuc/OnClass/blob/master/script/run_generate_pretrained_model.py
- name: tensorflow 2.0
  role: Deep learning backend for OnClass model used to produce prediction matrices
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:1d78a80bf827ea68c365dc268f4a6dd016c35503bce6f8a267c9685d463f5298
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - TensorFlow
  - run_generate_pretrained_model.py
  - run_marker_gene_based_prediction.py
  - run_marker_genes_identification.py
  - tensorflow 2.0
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cell-type-probability-expression-correlation

## Summary

Identifies marker genes for all Cell Ontology terms (including unseen cell types) by computing Spearman correlation between OnClass predicted cell-type probability vectors and per-gene expression profiles, then validates the markers by AUROC against curated marker gene sets.

## When to use

Use when you need marker genes for cell types absent from training data, given OnClass prediction matrices (pred_Y_all.released.npy) and single-cell expression data such as Tabula Muris Senis FACS cells.

## When NOT to use

- When the cell types of interest have no predicted probability vector available (OnClass model has not been run to produce pred_Y_all)
- When inputs are raw count matrices without OnClass cell-type probability scores — the method correlates expression against predicted probabilities, not raw annotations alone
- When trying to distinguish topologically identical unseen cell types (siblings with the same parents in the Cell Ontology graph)

## Inputs

- OnClass intermediate prediction matrices pred_Y_seen.released.npy and pred_Y_all.released.npy from run_generate_pretrained_model.py
- Annotated single-cell gene expression data (e.g., all FACS cells from Tabula Muris Senis)
- Cell Ontology files from the figshare deposits
- Curated marker gene sets for AUROC validation

## Outputs

- Ranked marker gene list per Cell Ontology term (exactly top 20 genes per cell type)
- Marker-gene-based cell type predictions (Figure 5d,e,f, Extended Data Figure 7)
- Per-cell-type AUROC stratified by seen vs unseen Cell Ontology terms

## How to apply

First run run_generate_pretrained_model.py to obtain the intermediate prediction matrices pred_Y_seen.released.npy and pred_Y_all.released.npy, together with annotated single-cell expression data (e.g., all FACS cells from Tabula Muris Senis) and Cell Ontology files from the figshare deposits. Then run run_marker_genes_identification.py, which computes Spearman correlation between each cell type's predicted probability vector in pred_Y_all and each gene's expression profile to rank all genes per cell type, and selects the top 20 marker genes per cell type, mirroring OnClass's precomputation strategy. Optionally run run_marker_gene_based_prediction.py to build marker-gene-based cell type predictions (Figure 5d,e,f, Extended Data Figure 7). Finally, evaluate the identified markers against curated marker gene sets by computing AUROC per cell type, stratifying results for seen cell ontology terms versus unseen terms in Tabula Muris FACS and droplet datasets.

## Related tools

- **OnClass** (Provides the predicted cell-type probability vectors (pred_Y_all) that are correlated with gene expression; implemented in Python) — https://github.com/wangshenguiuc/OnClass
- **run_marker_genes_identification.py** (Computes Spearman correlations between cell-type probability vectors and gene expression profiles to rank and select top 20 marker genes per cell type) — https://github.com/wangshenguiuc/OnClass/blob/master/script/run_marker_genes_identification.py
- **run_marker_gene_based_prediction.py** (Builds marker-gene-based cell type predictions from the identified markers (Figure 5d,e,f, Extended Data Figure 7)) — https://github.com/wangshenguiuc/OnClass/blob/master/script/run_marker_gene_based_prediction.py
- **run_generate_pretrained_model.py** (Generates the intermediate prediction matrices (pred_Y_seen.released.npy, pred_Y_all.released.npy) used as input) — https://github.com/wangshenguiuc/OnClass/blob/master/script/run_generate_pretrained_model.py
- **tensorflow 2.0** (Deep learning backend for OnClass model used to produce prediction matrices)

## Examples

```
python run_marker_genes_identification.py  # after run_generate_pretrained_model.py produced pred_Y_seen.released.npy and pred_Y_all.released.npy
```

## Evaluation signals

- AUROC for seen cell types falls within the reported range 0.87–0.97
- AUROC for unseen Cell Ontology terms is 0.87 in Muris FACS and 0.82 in Muris droplet
- Every cell type receives exactly 20 ranked marker genes
- Marker genes are ranked via Spearman correlation between the cell type's predicted probability vector in pred_Y_all and each gene's expression profile
- Marker-gene-based predictions reproduce Figure 5d,e,f and Extended Data Figure 7

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- The Cell Ontology was not developed specifically for scRNA-seq and therefore likely misses cell types and cell states
- OnClass cannot classify to terms outside the Cell Ontology vocabulary

## Evidence

- [abstract] Marker genes for all Cell Ontology categories including unseen cell types: "OnClass can be used to identify marker genes for all the cell ontology categories, regardless of whether the cell types are present or absent in the training data"
- [methods] Marker gene precomputation based on Tabula Muris Senis FACS cells: "Marker genes are precomputed based on all FACS cells from Tabula Muris Senis."
- [methods] Marker gene identification and prediction scripts: "Then run `run_marker_genes_identification.py <https://github.com/wangshenguiuc/OnClass/blob/master/script/run_marker_genes_identification.py>`__"
- [full_text] AUROC findings for unseen terms in Muris datasets: [evidence span withheld — anchor status: not_found]
- [discussion] Sibling limitation: "Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph."
