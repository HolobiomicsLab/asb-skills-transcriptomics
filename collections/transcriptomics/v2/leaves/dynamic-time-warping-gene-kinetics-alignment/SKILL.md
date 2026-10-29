---
name: dynamic-time-warping-gene-kinetics-alignment
description: Use when two single-cell pseudotime trajectories (typically already tree-aligned
  with CAPITAL) need marker-gene expression trends compared along pseudotime, e.g.
  matching erythrocyte, megakaryocyte, pre-B-cell, or dendritic-cell lineage kinetics
  between Setty et al. and Velten et al.
when_to_use_negative:
- When the trajectories themselves are branched and no linear path has been selected
  — DTW alone cannot align branching trajectories; first run capital.tl.tree_alignment
  to match leaf clusters
- When the two datasets have different global topological structure, where CAPITAL's
  alignment accuracy was reported to degrade in benchmarking
- When trajectories form complex graphs such as cycles or disconnected graphs, which
  CAPITAL in its present form cannot handle
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CAPITAL
  role: Provides capital.tl.dtw for DTW alignment of gene pseudotime kinetics and
    capital.tl.genes_similarity_score; installed from PyPI via pip install capital
    in a Python 3.8 conda environment
  repo: github.com/ykat0/capital
- name: Python
  role: Implementation language; CAPITAL can also be used in the JupyterLab interactive
    development environment
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:fc561680b03562de12e58eab1781fb3b62753ce467b2f50c2739ae5f577617fa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAPITAL
  - JupyterLab
  - Python
  - conda
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# dynamic-time-warping-gene-kinetics-alignment

## Summary

Align marker-gene pseudotime expression kinetics between two scRNA-seq datasets using dynamic time warping (capital.tl.dtw), so that genes with the same dynamics but shifted pseudotime are matched and gene–gene correlation conservation can be assessed after tree alignment.

## When to use

Use when two single-cell pseudotime trajectories (typically already tree-aligned with CAPITAL) need marker-gene expression trends compared along pseudotime, e.g. matching erythrocyte, megakaryocyte, pre-B-cell, or dendritic-cell lineage kinetics between Setty et al. and Velten et al. bone marrow datasets.

## When NOT to use

- When the trajectories themselves are branched and no linear path has been selected — DTW alone cannot align branching trajectories; first run capital.tl.tree_alignment to match leaf clusters
- When the two datasets have different global topological structure, where CAPITAL's alignment accuracy was reported to degrade in benchmarking
- When trajectories form complex graphs such as cycles or disconnected graphs, which CAPITAL in its present form cannot handle

## Inputs

- Two preprocessed scRNA-seq AnnData objects with inferred pseudotime (capital.tl.dpt) and trajectory trees (capital.tl.trajectory_tree)
- Marker-gene names or gene-expression matrices per lineage from each dataset (e.g. capital.dataset.setty19, capital.dataset.velten17)

## Outputs

- DTW-aligned gene pseudotime kinetics between datasets (capital.tl.dtw output)
- Gene similarity scores (capital.tl.genes_similarity_score)
- Plots of aligned gene expression trends and DTW alignment (capital.pl.gene_expression_trend, capital.pl.dtw)

## How to apply

After inferring pseudotime with capital.tl.dpt and computing each dataset's branched trajectory tree as a minimum spanning tree over clusters with capital.tl.trajectory_tree, extract marker-gene expression trends along pseudotime for corresponding aligned lineages. Run capital.tl.dtw to compute the DTW alignment between gene kinetics from the two datasets; DTW can deal with linear trajectories, so it is applied per matched lineage rather than to the branched trees themselves. Optionally score gene-pair correspondence with capital.tl.genes_similarity_score and render the aligned trends with capital.pl.gene_expression_trend and capital.pl.dtw. The rationale is that DTW alignment of kinetics after CAPITAL's global tree alignment reveals conserved (or divergent) gene-expression dynamics, including gene–gene correlation conservation between species, without manually rescaling pseudotime.

## Related tools

- **CAPITAL** (Provides capital.tl.dtw for DTW alignment of gene pseudotime kinetics and capital.tl.genes_similarity_score; installed from PyPI via pip install capital in a Python 3.8 conda environment) — github.com/ykat0/capital
- **Python** (Implementation language; CAPITAL can also be used in the JupyterLab interactive development environment)

## Examples

```
import capital; capital.tl.dtw(adata_setty, adata_velten)
```

## Evaluation signals

- DTW-aligned marker genes (e.g. across the Setty–Velten bone marrow alignment) show similar expression dynamics along pseudotime, visually consistent with the reported kinetics when rendered with capital.pl.dtw
- The DTW step is run only on lineages matched by the upstream CAPITAL tree alignment, which should reproduce expected leaves for erythrocyte, megakaryocyte, pre-B-cell, and dendritic-cell lineages
- Aligned trends rendered with capital.pl.gene_expression_trend are produced for each gene pair and plotted without errors
- Gene-pair similarity scores from capital.tl.genes_similarity_score are computed for the aligned kinetics

## Limitations

- DTW-based alignment methods can only deal with linear trajectories, so branching trajectories must first be decomposed via CAPITAL's tree alignment
- Alignment accuracy degrades for datasets with different global topological structure
- Denoising and imputing expression counts may generate more predicted clusters than previously reported, including intermediate states, which can affect lineage matching before DTW

## Evidence

- [intro] DTW can only deal with linear trajectories: "It remains, however, an unsettled question how one should select a pair of single lineages to be compared if the pseudotime trajectories include branches in each dataset"
- [methods] dtw tool listed in methods: "tl.tree_alignment
   tl.dpt
   tl.dtw"
- [abstract] revealing gene dynamics: "revealing various gene expression dynamics including gene–gene correlation conservation between different species"
- [results] bone marrow DTW-aligned kinetics: [evidence span withheld — anchor status: not_found]
- [readme] install command: [evidence span withheld — anchor status: external_doc]
