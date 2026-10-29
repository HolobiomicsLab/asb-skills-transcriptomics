---
name: pairwise-constraint-guided-deep-clustering
description: Use when clustering scRNA-seq data where purely unsupervised methods
  give biologically uninterpretable clusters and some prior knowledge (e.g., known
  same-type or different-type cell pairs) can be encoded as pairwise constraints.
when_to_use_negative:
- When prior knowledge is limited to marker genes and predefined cell types — methods
  like CellAssign or SCINA are designed for that setting, while scDCC targets pairwise
  constraints
- When no prior information is available and hard-constrained clustering is required
  — scDCC deliberately converts priors into soft constraints rather than hard constraints
  like COP K-means
- When the input is not scRNA-seq data with dropout events and high dimensionality,
  the motivating regime for scDCC's ZINB-style deep embedding
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDCC
  role: 'Primary method: model-based deep embedding clustering with soft pairwise
    constraints integrated via a novel loss function'
  repo: https://github.com/ttgump/scDCC
- name: PyTorch
  role: Deep learning framework used to implement and train scDCC (version 1.5.1+cu101)
- name: Scanpy
  role: scRNA-seq preprocessing library used in the scDCC pipeline (version 1.0.4)
- name: CIDR
  role: Competing clustering baseline on the 2100-cell datasets
  repo: https://github.com/VCCRI/CIDR
- name: DCA
  role: Denoising baseline combined with k-means for comparison
  repo: https://github.com/theislab/dca
- name: DEC
  role: Deep embedding clustering baseline (Keras implementation)
  repo: https://github.com/XifengGuo/DEC-keras
- name: MPSSC
  role: Spectral clustering baseline on the same 2100-cell samples
  repo: https://github.com/ishspsy/project
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:7645aabd676d65f41ebd3501ebbcbaa3edb36fe7fa404ed2916c89e70d670764
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - CIDR
  - DCA
  - DEC
  - MPSSC
  - Nvidia Tesla P100
  - PyTorch
  - Python
  - Scanpy
  - scDCC
  - scVI
  - scvis
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# pairwise-constraint-guided-deep-clustering

## Summary

Integrates partial prior knowledge into deep clustering of scRNA-seq data by converting it into soft must-link/cannot-link pairwise constraints added as extra terms to the loss function, implemented as scDCC (Single Cell Deep Constrained Clustering).

## When to use

Use when clustering scRNA-seq data where purely unsupervised methods give biologically uninterpretable clusters and some prior knowledge (e.g., known same-type or different-type cell pairs) can be encoded as pairwise constraints.

## When NOT to use

- When prior knowledge is limited to marker genes and predefined cell types — methods like CellAssign or SCINA are designed for that setting, while scDCC targets pairwise constraints
- When no prior information is available and hard-constrained clustering is required — scDCC deliberately converts priors into soft constraints rather than hard constraints like COP K-means
- When the input is not scRNA-seq data with dropout events and high dimensionality, the motivating regime for scDCC's ZINB-style deep embedding

## Inputs

- scRNA-seq count dataset with ground-truth or reference cell labels (e.g., 2100-cell samples from the figshare scDCC_data deposit, https://figshare.com/articles/dataset/scDCC_data/21563517)
- Prior knowledge convertible into pairwise must-link / cannot-link constraints
- Number of pairwise constraints to generate (--n_pairwise)

## Outputs

- Cluster assignments for each cell
- NMI, CA, and ARI values computed against ground-truth cell labels
- Per-method comparison table (e.g., benchmark_comparison_table.csv)

## How to apply

Download the scRNA-seq data (e.g., the four small 2100-cell datasets from the figshare scDCC_data deposit) and run scDCC via scDCC.py in Python 3.6.8 with PyTorch 1.5.1+cu101 and Scanpy 1.0.4 on a GPU such as an Nvidia Tesla P100. Generate soft pairwise constraints rather than hard-constrained assignments (unlike COP K-means), setting --n_pairwise for the number of constraints to generate. Weight the clustering, must-link, and cannot-link loss terms with --gamma, --ml_weight, and --cl_weight; with 0 constraints scDCC reduces to scDeepCluster. Run competing baselines (CIDR, DCA+k-means, DEC, MPSSC, PCA+k-means, SCVI+k-means, SCVIS+k-means, SIMLR, SC3, Seurat) on the same data and compute NMI, CA, and ARI against ground-truth labels to assemble a comparison table (e.g., benchmark_comparison_table.csv).

## Related tools

- **scDCC** (Primary method: model-based deep embedding clustering with soft pairwise constraints integrated via a novel loss function) — https://github.com/ttgump/scDCC
- **PyTorch** (Deep learning framework used to implement and train scDCC (version 1.5.1+cu101))
- **Scanpy** (scRNA-seq preprocessing library used in the scDCC pipeline (version 1.0.4))
- **CIDR** (Competing clustering baseline on the 2100-cell datasets) — https://github.com/VCCRI/CIDR
- **DCA** (Denoising baseline combined with k-means for comparison) — https://github.com/theislab/dca
- **DEC** (Deep embedding clustering baseline (Keras implementation)) — https://github.com/XifengGuo/DEC-keras
- **MPSSC** (Spectral clustering baseline on the same 2100-cell samples) — https://github.com/ishspsy/project

## Examples

```
python scDCC.py --n_pairwise 600 --gamma 0.1 --ml_weight 1 --cl_weight 1
```

## Evaluation signals

- scDCC with 0 constraints should behave like scDeepCluster and outperform most competing scRNA-seq clustering methods on the 2100-cell samples
- With pairwise constraints, scDCC should attain higher NMI, CA, and ARI than all competing methods on the four small datasets
- Every method's metrics must be computed against the same ground-truth cell labels on identical down-sampled datasets
- The per-method metric values should assemble cleanly into a single comparison table (benchmark_comparison_table.csv)
- Increasing the number of generated pairwise constraints (--n_pairwise) should not degrade performance relative to the unconstrained run

## Limitations

- Purely unsupervised clustering methods may not produce biologically interpretable clusters on scRNA-seq data, so the unconstrained (0-constraint) mode inherits that weakness
- Marker-gene-based prior-knowledge methods (CellAssign, SCINA) are a different paradigm and cannot be swapped in directly for pairwise constraints
- Hard-constrained algorithms like COP K-means cannot handle inconsistent priors the way soft constraints can
- Discussion section content is absent from the provided document, limiting detail on caveats

## Evidence

- [intro] Unlike traditional hard-constrained clustering algorithms (e.g., COP K-means36), we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss: "Unlike traditional hard-constrained clustering algorithms (e.g., COP K-means36), we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss"
- [abstract] scDCC improves clustering: "Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance"
- [methods] constraint generation flags: "--n_pairwise: number of pairwise constraints want to generate"
- [intro] motivating gap: "Most, if not all, existing algorithms for scRNA-seq, however, only support clustering in an unsupervised fashion, and are incapable of integrating prior information."
- [readme] repository README method description: "When confronted by the high dimensionality and pervasive dropout events of scRNA-Seq data, purely unsupervised clustering methods may not produce biologically interpretable clusters"
