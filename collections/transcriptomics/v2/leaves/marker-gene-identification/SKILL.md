---
name: marker-gene-identification
description: Use when you need marker gene sets for Cell Ontology cell types — including
  types with no annotated cells in training data — from OnClass prediction matrices
  and single-cell expression data such as Tabula Muris Senis FACS.
when_to_use_negative:
- You only need cell type label annotation from expression data and do not need per-term
  gene rankings — use OnClass classification (e.g., run_OnClass_example.py) instead.
- The cell types of interest are topologically identical siblings in the Cell Ontology
  graph, which OnClass cannot distinguish, making per-term marker gene identity unreliable.
- Your reference lacks annotated cells and you require experimentally curated marker
  associations rather than correlation-derived markers.
edam_operation: http://edamontology.org/operation_2495
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: OnClass
  role: Generates the pred_Y_seen / pred_Y_all probability vectors whose per-cell-type
    correlations with gene expression yield the marker gene rankings, including for
    unseen Cell Ontology terms.
  repo: https://github.com/wangshenguiuc/OnClass
- name: Python
  role: Language in which the OnClass marker gene identification and prediction scripts
    are implemented and run.
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow 2.0
  role: Backend dependency for training the OnClass model whose prediction matrices
    feed marker gene identification.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:26bda62633d6a581821edcd45ee3dd9e9b8535a85a474667d6191dc52ba3e800
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - TensorFlow
  - tensorflow 2.0
  edam_operation: http://edamontology.org/operation_2495
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# marker-gene-identification

## Summary

Identify marker genes for every Cell Ontology term (seen or unseen) by ranking genes per cell type via Spearman correlation between OnClass predicted cell-type probability vectors and gene expression profiles, then validate them by marker-gene-based prediction AUROC.

## When to use

Use when you need marker gene sets for Cell Ontology cell types — including types with no annotated cells in training data — from OnClass prediction matrices and single-cell expression data such as Tabula Muris Senis FACS.

## When NOT to use

- You only need cell type label annotation from expression data and do not need per-term gene rankings — use OnClass classification (e.g., run_OnClass_example.py) instead.
- The cell types of interest are topologically identical siblings in the Cell Ontology graph, which OnClass cannot distinguish, making per-term marker gene identity unreliable.
- Your reference lacks annotated cells and you require experimentally curated marker associations rather than correlation-derived markers.

## Inputs

- pred_Y_seen.released.npy (OnClass prediction matrix for seen cell types)
- pred_Y_all.released.npy (OnClass prediction matrix covering all Cell Ontology terms, including unseen)
- Annotated single-cell gene expression data (e.g., all FACS cells from Tabula Muris Senis)
- Cell Ontology files from the figshare deposits

## Outputs

- Ranked gene list per cell type with exactly 20 selected marker genes per Cell Ontology term (including unseen terms)
- Marker-gene-based cell type predictions (Figure 5d,e,f, Extended Data Figure 7)
- Per-cell-type AUROC values against curated marker gene sets, stratified for seen and unseen terms in Muris FACS and Muris droplet

## How to apply

First run run_generate_pretrained_model.py to produce the intermediate prediction matrices pred_Y_seen.released.npy and pred_Y_all.released.npy. Then run run_marker_genes_identification.py, which computes the Spearman correlation between each cell type's predicted probability vector in pred_Y_all and each gene's expression profile to rank all genes per cell type, mirroring OnClass's precomputation strategy based on all FACS cells from Tabula Muris Senis, and select the top 20 ranked marker genes per cell type. Next run run_marker_gene_based_prediction.py to build marker-gene-based cell type predictions (Figure 5d,e,f, Extended Data Figure 7). Finally evaluate the markers against curated marker gene sets by computing AUROC per cell type, stratifying for seen versus unseen Cell Ontology terms in Muris FACS and droplet.

## Related tools

- **OnClass** (Generates the pred_Y_seen / pred_Y_all probability vectors whose per-cell-type correlations with gene expression yield the marker gene rankings, including for unseen Cell Ontology terms.) — https://github.com/wangshenguiuc/OnClass
- **Python** (Language in which the OnClass marker gene identification and prediction scripts are implemented and run.) — https://github.com/wangshenguiuc/OnClass
- **tensorflow 2.0** (Backend dependency for training the OnClass model whose prediction matrices feed marker gene identification.)

## Examples

```
python run_generate_pretrained_model.py; python run_marker_genes_identification.py; python run_marker_gene_based_prediction.py
```

## Evaluation signals

- Each cell type receives exactly 20 ranked marker genes.
- Computed AUROC values fall within reported ranges: 0.87–0.97 for seen cell types, 0.87 for unseen types in Muris FACS, and 0.82 for unseen types in Muris droplet.
- Marker rankings derive from Spearman correlation between each cell type's predicted probability vector in pred_Y_all and each gene's expression profile, matching the OnClass precomputation strategy.
- Marker genes are precomputed using all FACS cells from Tabula Muris Senis.
- Marker-gene-based predictions reproduce Figure 5d,e,f and Extended Data Figure 7 outputs.

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph, so markers for such terms are not separable.
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, so marker sets cover only existing ontology terms.
- The Cell Ontology lacks marker gene associations for most of its cell type terms.

## Evidence

- [abstract] OnClass can be used to identify marker genes for all the cell ontology categories, regardless of whether the cell types are present or absent in the training data: "OnClass can be used to identify marker genes for all the cell ontology categories, regardless of whether the cell types are present or absent in the training data"
- [methods] Then run run_marker_genes_identification.py: "Then run `run_marker_genes_identification.py <https://github.com/wangshenguiuc/OnClass/blob/master/script/run_marker_genes_identification.py>`__"
- [methods] Marker genes are precomputed based on all FACS cells from Tabula Muris Senis.: "Marker genes are precomputed based on all FACS cells from Tabula Muris Senis."
- [results] OnClass achieved an AUROC of 0.87 for unseen Cell Ontology terms in Muris FACS and an AUROC of 0.82 for unseen Cell Ontology terms in Muris droplet: [evidence span withheld — anchor status: not_found]
- [discussion] OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph: [evidence span withheld — anchor status: not_found]
