---
name: sparse-subspace-representation-clustering
description: Use when the goal is unsupervised cell-type detection from a cell-by-gene
  scRNA-seq expression matrix with high noise and dropouts, or when benchmarking a
  similarity-enhancement step against the original sparse subspace representation.
when_to_use_negative:
- Input is raw sequencing reads or an already-clustered annotation — this skill expects
  a cells-by-genes expression matrix with numeric expression values
- The task requires the enhanced similarity step itself (adaptive gene selection +
  similarity enhancement); then run full SSRE rather than the SSR-only baseline
- Very large-scale datasets where runtime matters — the article notes SSRE-family
  approaches are relatively time-consuming on large-scale datasets
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MATLAB
  role: 'Implementation language for the SSRE/SSR baseline: sparse subspace representation
    of cell-to-cell similarity plus spectral clustering'
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative implementation language of the SSRE repository used to run the
    SSR baseline and compute NMI/ARI
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:20bc88d4941483b4887b6ed758d429b5f20e2792569a6044a204187a7ddefc01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# sparse-subspace-representation-clustering

## Summary

Clusters scRNA-seq cells by computing a sparse subspace representation (SSR) of cell-to-cell similarity — a linear, self-expressive representation grounded in sparse subspace theory — and feeding it to spectral clustering to reveal cell types. This is the baseline that SSRE's similarity-enhancement strategy was designed to improve upon.

## When to use

Use when the goal is unsupervised cell-type detection from a cell-by-gene scRNA-seq expression matrix with high noise and dropouts, or when benchmarking a similarity-enhancement step against the original sparse subspace representation.

## When NOT to use

- Input is raw sequencing reads or an already-clustered annotation — this skill expects a cells-by-genes expression matrix with numeric expression values
- The task requires the enhanced similarity step itself (adaptive gene selection + similarity enhancement); then run full SSRE rather than the SSR-only baseline
- Very large-scale datasets where runtime matters — the article notes SSRE-family approaches are relatively time-consuming on large-scale datasets

## Inputs

- n-by-p scRNA-seq gene expression matrix (e.g., in_X in a .mat file, cells x genes)
- Ground-truth cell type labels for evaluation (true_labs)
- MATLAB or Python SSRE implementations for the SSR baseline + spectral clustering

## Outputs

- Sparse cell-to-cell similarity matrix from sparse subspace representation
- Per-cell clustering labels (cell type assignments)
- NMI and ARI (and optionally Purity) scores vs. ground-truth labels
- SSR-vs-SSRE comparison table per dataset with win counts

## How to apply

Load a dataset consisting of an n-by-p gene expression matrix (in_X, n cells by p genes) and ground-truth cell type labels (true_labs). Compute the linear representation between cells based on sparse subspace theory to generate a sparse representation of cell-to-cell similarity, then perform spectral clustering on this learned similarity to assign cell type labels. Evaluate clustering quality with NMI and ARI against true_labs (optionally Purity). In the SSRE ablation context, run this SSR baseline side by side with the full SSRE pipeline (adaptive gene selection plus similarity enhancement) on the same ten real scRNA-seq datasets and count wins on NMI and ARI; the paper's reference outcome is SSRE beating SSR on nine of ten datasets in both metrics.

## Related tools

- **MATLAB** (Implementation language for the SSRE/SSR baseline: sparse subspace representation of cell-to-cell similarity plus spectral clustering) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative implementation language of the SSRE repository used to run the SSR baseline and compute NMI/ARI) — https://github.com/CSUBioGroup/SSRE

## Examples

```
[P] = clus_fin_update(rho, lam, lam2, eta, c, in_X, true_labs);
```

## Evaluation signals

- NMI and ARI computed against true_labs are reported for both SSR baseline and SSRE on every dataset
- Win-count check: SSRE should outperform the original SSR on nine of the ten real scRNA-seq datasets in both NMI and ARI; flag any divergent dataset
- The learned similarity matrix should expose a clear block structure after spectral clustering, matching the paper's claim that learned similarities reveal block structure reliably
- Clustering should be reproducible across the MATLAB and Python implementations in the CSUBioGroup/SSRE repository

## Limitations

- SSRE (and hence the SSR-based pipeline) is relatively time-consuming on large-scale datasets; parallel computing is suggested but not implemented in the paper
- Existing similarity-based cell type identification approaches are sensitive to noise, especially for high-dimensional data — the SSR baseline alone computes similarity using only the gene expression of cell pairs
- The three pairwise similarities can be selected individually (one or two instead of all three), changing baseline behavior; all three are used by default in the full pipeline

## Evidence

- [abstract] SSRE repo: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [intro] SSR theory: "SSRE computes the linear representation between cells based on sparse subspace theory, and thus generates a sparse representation of cell-to-cell similarity [19]."
- [discussion] enhancement strategy: "Moreover, a gene selection process and an enhancement strategy were designed based on the characteristics of different similarities to learn more reliable similarities."
- [intro] spectral clustering: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
- [readme] evaluation pattern: [evidence span withheld — anchor status: not_found]
