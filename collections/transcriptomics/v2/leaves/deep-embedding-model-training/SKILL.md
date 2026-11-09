---
name: deep-embedding-model-training
description: Use when clustering scRNA-seq count matrices where some cells (e.g.,
  a 10% labeled hold-out) have known labels or pairwise relationships, and purely
  unsupervised clustering gives biologically uninterpretable clusters.
when_to_use_negative:
- No partial labels or domain knowledge are available, so no pairwise constraints
  can be built and only fully unsupervised clustering is possible
- CellAssign-style marker-gene, predefined-cell-type annotation is sufficient — the
  article notes prior-knowledge methods limited to marker genes ignore dropout events,
  but if marker genes fully define the problem scDCC is overkill
- Data are not scRNA-seq count matrices requiring dropout-robust deep embedding (e.g.,
  already-reduced feature tables or bulk RNA-seq)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scDCC
  role: Core deep constrained clustering model trained with pairwise constraint loss
    terms
  repo: https://github.com/ttgump/scDCC
- name: PyTorch 1.5.1+cu101
  role: Deep learning framework used to implement and train scDCC.py
- name: Scanpy 1.0.4
  role: Preprocessing of the scRNA-seq count matrices before model training
- name: Nvidia Tesla P100
  role: GPU used for training the model on the study datasets
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:b9cae6c2d900cc26bf9ffb5717a2c35c522f72ea4c65aed0fcfb5ee859d8681e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - Nvidia Tesla P100
  - PyTorch 1.5.1+cu101
  - Python
  - Scanpy
  - Scanpy 1.0.4
  - scDCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# deep-embedding-model-training

## Summary

Trains scDCC, a model-based deep embedding clustering model that integrates pairwise must-link/cannot-link constraints from partially labeled cells into a deep autoencoder's loss, to cluster scRNA-seq data from thousands to tens of thousands of cells.

## When to use

Use when clustering scRNA-seq count matrices where some cells (e.g., a 10% labeled hold-out) have known labels or pairwise relationships, and purely unsupervised clustering gives biologically uninterpretable clusters.

## When NOT to use

- No partial labels or domain knowledge are available, so no pairwise constraints can be built and only fully unsupervised clustering is possible
- CellAssign-style marker-gene, predefined-cell-type annotation is sufficient — the article notes prior-knowledge methods limited to marker genes ignore dropout events, but if marker genes fully define the problem scDCC is overkill
- Data are not scRNA-seq count matrices requiring dropout-robust deep embedding (e.g., already-reduced feature tables or bulk RNA-seq)

## Inputs

- scRNA-seq count matrices (e.g., 10X PBMC, mouse bladder, worm neuron, human kidney, Macosko 14,653-cell and Shekhar 27,499-cell mouse retina datasets from the figshare scDCC_data deposit)
- Partial ground-truth cell-type labels for ~10% of cells used to generate pairwise constraints
- Known number of clusters (cell types) per dataset

## Outputs

- Trained scDCC deep embedding model (scDCC.py, PyTorch)
- Cluster assignments for all cells
- Per-constraint-count clustering metrics table (NMI, CA, ARI) aggregated over ten repeats (e.g., retina_clustering_metrics.csv)

## How to apply

Preprocess the count matrix with Scanpy 1.0.4, then hold out ~10% of labeled cells as the constraint source. Sample must-link and cannot-link pairs from this hold-out at increments of --n_pairwise (from 0 up to ~6000, where most NMI/CA/ARI values exceed 0.8), and set --n_clusters to the known cell-type count, --gamma to weight the clustering loss, and ml_weight/cl_weight to weight the must-link and cannot-link penalty terms. Train scDCC.py (PyTorch 1.5.1+cu101, e.g., on an Nvidia Tesla P100) so the soft pairwise constraints are added as additional terms to the loss rather than applied as hard constraints (unlike COP K-means). Evaluate on the remaining ~90% of cells using NMI, CA, and ARI against ground-truth labels, repeating each setting over ten random splits/repeats and aggregating mean metrics per constraint count.

## Related tools

- **scDCC** (Core deep constrained clustering model trained with pairwise constraint loss terms) — https://github.com/ttgump/scDCC
- **PyTorch 1.5.1+cu101** (Deep learning framework used to implement and train scDCC.py)
- **Scanpy 1.0.4** (Preprocessing of the scRNA-seq count matrices before model training)
- **Nvidia Tesla P100** (GPU used for training the model on the study datasets)

## Examples

```
python scDCC_pairwise_CITE_PBMC.py --n_pairwise 6000 --n_clusters <k> --gamma <g>
```

## Evaluation signals

- NMI, CA, and ARI computed between predicted and ground-truth labels improve monotonically as the number of pairwise constraints increases
- With 6000 pairwise constraints, most clustering metrics exceed 0.8 on the four small datasets (10X PBMC, mouse bladder, worm neuron, human kidney)
- Each setting is repeated over ten random splits/repeats and mean scores are reported, so variance across repeats should be reported alongside means
- Large retina datasets (Macosko 14,653 cells; Shekhar 27,499 cells) show consistent improvement over baselines as constraints built from the 10% labeled hold-out increase
- Constraints are sourced only from the 10% labeled hold-out, so evaluation on the remaining 90% must remain independent of the constraint pool

## Limitations

- Requires at least partial labeled data to build pairwise constraints; performance depends on the number of constraints and may be limited by hold-out size (10% labeled)
- Pairwise constraints are soft loss terms, not hard guarantees, so constraint satisfaction is not enforced at inference (unlike hard-constrained clustering such as COP K-means)
- Requires --n_clusters (known cell-type count) to be specified per dataset
- Purely unsupervised clustering (zero constraints) may not produce biologically interpretable clusters in high-dimensionality, dropout-heavy scRNA-seq data

## Evidence

- [abstract] Here we report a principled clustering method named scDCC, that integrates domain knowledge into the clustering step.: "Here we report a principled clustering method named scDCC, that integrates domain knowledge into the clustering step."
- [intro] we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss: "Unlike traditional hard-constrained clustering algorithms (e.g., COP K-means36), we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss"
- [methods] --n_pairwise: number of pairwise constraints want to generate: "--n_pairwise: number of pairwise constraints want to generate"
- [methods] python scDCC_pairwise_CITE_PBMC.py: "python scDCC_pairwise_CITE_PBMC.py"
- [readme] Clustering is a critical step in single cell-based studies.: "Clustering is a critical step in single cell-based studies."
