---
name: spectral-clustering-execution
description: Use when an scRNA-seq expression matrix needs cell-type clustering via
  spectral clustering on a learned or enhanced similarity matrix, and clustering quality
  must be scored with NMI and ARI against provided cell-type labels.
when_to_use_negative:
- When only a raw high-dimensional expression matrix is available and no similarity-learning
  step (sparse subspace representation or enhancement) has been performed — spectral
  clustering here presumes a learned affinity input
- For very large-scale scRNA-seq datasets where runtime matters — SSRE is described
  as relatively time-consuming on large-scale data
- When the goal is multi-omics integration or incorporation of gene-function/regulatory
  information — these are only proposed as future work, not implemented
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SSRE
  role: Produces the sparse subspace representation (SSR) and enhanced cell-to-cell
    similarity matrices that are combined with spectral clustering; MATLAB and Python
    implementations used to run clustering and score NMI/ARI
  repo: https://github.com/CSUBioGroup/SSRE
- name: MPSSC
  role: Reference MATLAB pipeline showing spectral-clustering-on-learned-similarity
    execution with NMI/Purity/ARI scoring helpers (calc2_nmis, RandIndex, purity)
    on scRNA-seq .mat datasets
  repo: https://github.com/ishspsy/project
- name: SC3
  role: Alternative unsupervised scRNA-seq cell clustering tool for cross-method comparison
    context
  repo: https://github.com/hemberg-lab/SC3
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:4f7a99bb0ba52ff9e0e679dfdcdea6f6ca8d385d715e9c0d4f980820234a51d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - MPSSC
  - Python
  - SC3
  - SSRE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# spectral-clustering-execution

## Summary

Run spectral clustering on a learned cell-to-cell similarity matrix (SSR sparse subspace representation or SSRE-enhanced similarity) to identify cell types from scRNA-seq data, and quantify clustering accuracy with NMI/ARI against ground-truth labels.

## When to use

Use when an scRNA-seq expression matrix needs cell-type clustering via spectral clustering on a learned or enhanced similarity matrix, and clustering quality must be scored with NMI and ARI against provided cell-type labels.

## When NOT to use

- When only a raw high-dimensional expression matrix is available and no similarity-learning step (sparse subspace representation or enhancement) has been performed — spectral clustering here presumes a learned affinity input
- For very large-scale scRNA-seq datasets where runtime matters — SSRE is described as relatively time-consuming on large-scale data
- When the goal is multi-omics integration or incorporation of gene-function/regulatory information — these are only proposed as future work, not implemented

## Inputs

- scRNA-seq gene expression matrix (n cells × p genes, e.g. a .mat file with in_X)
- ground-truth cell-type label vector (true_labs) for the same cells
- learned cell-to-cell similarity matrix (sparse subspace representation from SSR, or SSRE-enhanced similarity)

## Outputs

- per-cell cluster assignments (clus_labs)
- clustering performance metrics: NMI and ARI (plus Purity where supported) computed against true labels
- SSR-vs-SSRE comparison table per dataset with win counts

## How to apply

Load the scRNA-seq expression matrix (n cells × p genes) together with ground-truth cell-type labels, then compute a cell-to-cell similarity matrix — either the sparse subspace representation (SSR) baseline from sparse subspace theory, or the SSRE-enhanced similarity produced by the two-step adaptive gene selection plus similarity enhancement procedure. Feed the similarity matrix (as the graph affinity) into spectral clustering to reveal the block structure of the data and obtain cluster labels for each cell. Evaluate the resulting labels with NMI and ARI against the true cell-type labels, and tabulate results per dataset. For the ablation decision point, compare SSR vs. full SSRE per dataset and count wins: the paper's reference result is SSRE outperforming SSR on nine of ten real scRNA-seq datasets in both NMI and ARI; flag any dataset where the outcome diverges. MATLAB and Python implementations are provided in the CSUBioGroup/SSRE repository.

## Related tools

- **SSRE** (Produces the sparse subspace representation (SSR) and enhanced cell-to-cell similarity matrices that are combined with spectral clustering; MATLAB and Python implementations used to run clustering and score NMI/ARI) — https://github.com/CSUBioGroup/SSRE
- **MPSSC** (Reference MATLAB pipeline showing spectral-clustering-on-learned-similarity execution with NMI/Purity/ARI scoring helpers (calc2_nmis, RandIndex, purity) on scRNA-seq .mat datasets) — https://github.com/ishspsy/project
- **SC3** (Alternative unsupervised scRNA-seq cell clustering tool for cross-method comparison context) — https://github.com/hemberg-lab/SC3

## Examples

```
[P] = clus_fin_update(rho, lam, lam2, eta, c, in_X, true_labs); [NMI, ~,clus_labs,~]=calc2_nmis(max(true_labs), double(P),true_labs); ARI=RandIndex(clus_labs,true_labs);
```

## Evaluation signals

- NMI and ARI computed against provided ground-truth cell-type labels are the primary correctness metrics
- Ablation check: SSRE should beat SSR on 9 of 10 real scRNA-seq datasets in both NMI and ARI; flag any dataset where the outcome diverges
- Spectral clustering on the learned similarity should reliably reveal the block structure of the scRNA-seq data
- Metric outputs should match the schema of the provided scoring helpers, e.g. [NMI, Purity, ARI] triples and cluster labels aligned 1:1 with input cells
- SSRE should show superior performance in most cases versus state-of-the-art single-cell clustering methods on the ten real and five simulated datasets

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is proposed but not implemented
- Traditional clustering struggles on scRNA-seq data due to high noise rate and high dropouts, so results on noisy high-dimensional inputs without the gene-selection/enhancement step may degrade
- Users may substitute one or two of the three pairwise similarities instead of the default all-three configuration, which changes the learned similarity fed to spectral clustering

## Evidence

- [intro] spectral clustering on learned similarity reveals block structure: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
- [intro] SSR sparse representation generation: "SSRE computes the linear representation between cells based on sparse subspace theory, and thus generates a sparse representation of cell-to-cell similarity"
- [abstract] MATLAB/Python implementations available: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [discussion] scalability limitation: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework"
- [readme] MPSSC MATLAB execution pattern with NMI/ARI scoring: [evidence span withheld — anchor status: not_found]
