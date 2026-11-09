---
name: pairwise-constraint-generation
description: 'Use when clustering scRNA-seq data with scDCC and partial labels or
  marker genes are available: hold out ~10% of labeled cells, then sample must-link/cannot-link
  pairs to guide the model''s latent space toward biologically meaningful clusters.'
when_to_use_negative:
- Only unsupervised clustering is needed and no labels or marker genes exist to derive
  must-link/cannot-link pairs — scDCC then reduces to the 0-constraint baseline
- The task requires hard-constrained clustering (e.g., COP-K-means style) where constraints
  must never be violated, rather than soft penalty terms
- 'Constraint labels are expected to be highly unreliable (>10% error): scDCC begins
  to underperform the unsupervised baseline at a 10% constraint error rate (e.g.,
  mouse bladder cells)'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scDCC
  role: Deep constrained clustering model that consumes the generated pairwise constraints
    as soft penalty terms in its loss
  repo: https://github.com/ttgump/scDCC
- name: Scanpy
  role: Preprocessing of scRNA-seq count matrices before constraint generation
- name: ZIFA
  role: Latent representation of marker genes from which k-means pseudo-labels and
    constraints are derived (human liver variant)
- name: Python 3.6.8
  role: Implementation environment for the constraint generation and scDCC wrappers
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:8d53568fb92951fda1b7b1b7417550240db762cd8f7ed79416ee82a926c77038
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - Nvidia Tesla P100
  - Python
  - Python 3.6.8
  - Scanpy
  - ZIFA
  - scDCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pairwise-constraint-generation

## Summary

Generate soft must-link and cannot-link pairwise constraints from a small labeled hold-out set (or marker-gene-derived pseudo-labels) to supply domain knowledge to the scDCC deep constrained clustering model, with the constraint count controlled by --n_pairwise.

## When to use

Use when clustering scRNA-seq data with scDCC and partial labels or marker genes are available: hold out ~10% of labeled cells, then sample must-link/cannot-link pairs to guide the model's latent space toward biologically meaningful clusters.

## When NOT to use

- Only unsupervised clustering is needed and no labels or marker genes exist to derive must-link/cannot-link pairs — scDCC then reduces to the 0-constraint baseline
- The task requires hard-constrained clustering (e.g., COP-K-means style) where constraints must never be violated, rather than soft penalty terms
- Constraint labels are expected to be highly unreliable (>10% error): scDCC begins to underperform the unsupervised baseline at a 10% constraint error rate (e.g., mouse bladder cells)

## Inputs

- Preprocessed scRNA-seq count matrix (Scanpy-normalized) with cell-type labels for a ~10% subset of cells
- 63 liver marker genes + ZIFA latent representation + k-means pseudo-labels (label-free variant, e.g. human liver dataset)
- Target number of pairwise constraints (--n_pairwise), e.g. 0 to 6000
- Loss weights: --gamma (clustering loss), --ml_weight (must-link), --cl_weight (cannot-link)

## Outputs

- Set of must-link and cannot-link cell pairs consumed by scDCC as soft constraints
- Constraint-annotation table (dataset, n_pairwise, error rate) used in robustness experiments

## How to apply

Hold out 10% of cells with known cell-type labels as the constraint source; within this set, cells sharing a label form must-link pairs and cells with different labels form cannot-link pairs, sampled at increments of --n_pairwise from 0 up to about 6000 pairs. Rationale: unlike hard-constrained clustering (e.g., COP-K-means), scDCC converts partial prior knowledge into soft pairwise constraints added as penalty terms to the loss, weighted by --ml_weight (must-link) and --cl_weight (cannot-link) alongside the clustering-loss weight --gamma. When true labels are absent, first compute pseudo-labels (e.g., k-means on ZIFA latent representations of marker genes such as the 63 human liver markers) and generate constraints from those. Iterate the constraint count to probe the performance/annotation-cost trade-off; on four small benchmarks (10X PBMC, mouse bladder, worm neuron, human kidney) most NMI/CA/ARI values exceed 0.8 at 6000 constraints.

## Related tools

- **scDCC** (Deep constrained clustering model that consumes the generated pairwise constraints as soft penalty terms in its loss) — https://github.com/ttgump/scDCC
- **Scanpy** (Preprocessing of scRNA-seq count matrices before constraint generation)
- **ZIFA** (Latent representation of marker genes from which k-means pseudo-labels and constraints are derived (human liver variant))
- **Python 3.6.8** (Implementation environment for the constraint generation and scDCC wrappers)

## Examples

```
python scDCC_pairwise_CITE_PBMC.py --n_pairwise 6000 --gamma 1 --ml_weight 1 --cl_weight 1
```

## Evaluation signals

- Clustering NMI/CA/ARI on the remaining 90% of cells should increase monotonically as --n_pairwise increases (Fig. 2 trend on 10X PBMC, mouse bladder, worm neuron, human kidney)
- At ~6000 pairwise constraints most clustering metrics exceed 0.8 across the four small benchmark datasets
- With 5% of constraints deliberately flipped, performance should still exceed the 0-constraint baseline on all datasets; degradation at 10% error flags excessive label noise
- On the human liver marker-gene variant, scDCC should reach approximately NMI 0.905, CA 0.928, ARI 0.952, exceeding k-means on ZIFA features (NMI 0.833, CA 0.928, ARI 0.924)
- Repeated runs (ten random splits/repeats) should yield stable mean scores across constraint levels

## Limitations

- Constraints must be built from a labeled subset (~10%); purely unlabeled data requires pseudo-labeling first, which caps achievable quality
- scDCC is robust to 5% erroneous (flipped) constraints but begins to underperform at a 10% error rate on some datasets (e.g., mouse bladder cells)
- Constraint-based gains depend on soft penalty weighting (--gamma, --ml_weight, --cl_weight); defaults were tuned for the reported benchmarks (PyTorch 1.5.1+cu101 on an Nvidia Tesla P100)

## Evidence

- [intro] soft pairwise constraints from partial labels: "Unlike traditional hard-constrained clustering algorithms (e.g., COP K-means36), we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss"
- [methods] n_pairwise parameter: "--n_pairwise: number of pairwise constraints want to generate"
- [results] improvement with constraint count: [evidence span withheld — anchor status: not_found]
- [results] robustness to erroneous constraints: [evidence span withheld — anchor status: not_found]
- [readme] repository availability: "scDCC -- Single Cell Deep Constrained Clustering"
