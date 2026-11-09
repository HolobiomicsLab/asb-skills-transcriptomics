---
name: kernel-based-cell-similarity-construction
description: Use when you need to encode a known biological signal (cell-cycle labels,
  spatial coordinates, zonation gene scores, or a cell-to-reference mapping) as a
  cell–cell similarity kernel before projecting and removing that signal from scRNA-seq
  expression data with SiFT.
when_to_use_negative:
- No prior knowledge exists about the signal to filter — SiFT strongly relies on prior
  knowledge to define the cell–cell similarity kernel, and an inaccurate or coarse-grained
  mapping may lead to loss of signal.
- The goal is batch-effect integration at the latent-representation level (e.g., bbknn,
  Harmony, scVI) rather than gene-expression-level filtering of a defined biological
  signal.
- Data are already filtered/corrected and downstream work only requires log-transformed
  expression — SiFT output is no longer properly log-transformed.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SiFT (sift-sc)
  role: Provides the sift.kernels kernel constructors (MappingKernel, KnnKernel, RBFKernel,
    LaplacianKernel) used to build the cell–cell similarity kernel
  repo: https://github.com/nitzanlab/sift-sc
- name: Cyclum
  role: Supplies Cyclum bins/phases and circular pseudotime used as attributes for
    cell-cycle kernels in the two-subclone benchmark
  repo: https://github.com/KChen-lab/Cyclum
- name: Python
  role: Implementation language; kernels are instantiated and consumed in Python with
    scanpy-loaded AnnData inputs
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:d4ff5b094c180ddb6dc4e19393d9daf6db7a619df678c5f9e135b7206dea5641
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Cyclum
  - Python
  - SiFT
  - SiFT (sift-sc)
  - pip
  - sift.SiFT
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# kernel-based-cell-similarity-construction

## Summary

Constructs a probabilistic cell–cell similarity kernel capturing similarity with respect to a biological signal (e.g., cell cycle phase, spatial zonation, a reference mapping) as the first step of SiFT signal filtering. The kernel choice (mapping, knn, RBF/Laplacian distance) determines how the signal to be filtered is encoded.

## When to use

Use when you need to encode a known biological signal (cell-cycle labels, spatial coordinates, zonation gene scores, or a cell-to-reference mapping) as a cell–cell similarity kernel before projecting and removing that signal from scRNA-seq expression data with SiFT.

## When NOT to use

- No prior knowledge exists about the signal to filter — SiFT strongly relies on prior knowledge to define the cell–cell similarity kernel, and an inaccurate or coarse-grained mapping may lead to loss of signal.
- The goal is batch-effect integration at the latent-representation level (e.g., bbknn, Harmony, scVI) rather than gene-expression-level filtering of a defined biological signal.
- Data are already filtered/corrected and downstream work only requires log-transformed expression — SiFT output is no longer properly log-transformed.

## Inputs

- Single-cell gene expression count matrix (e.g., 279-cell mESC virtual tumor count matrix, or GSE145197 liver scRNA-seq loaded with scanpy)
- Per-cell attribute annotation encoding the signal to filter (cell-cycle phase labels/scores, Cyclum bins, Seurat cell-cycle labels)
- Optional cell-to-cell or cell-to-location mapping matrix (e.g., novoSpaRc spatial mapping from GSE145197)
- Prior-knowledge gene sets (e.g., zonated genes) for building KNN kernels from expression scores

## Outputs

- Row-stochastic cell–cell similarity kernel object (sift.kernels.MappingKernel, KnnKernel, RBFKernel, or LaplacianKernel)
- Kernel ready to be consumed by sift.SiFT for projection P = K·X and subtraction of the filtered signal

## How to apply

Start from a single-cell gene expression count matrix plus a per-cell attribute encoding the signal to filter (e.g., Cyclum bins, Seurat cell-cycle labels, novoSpaRc spatial cell-to-location mapping). Choose the kernel variant that matches the attribute type: sift.kernels.MappingKernel when a probabilistic cell-to-cell or cell-to-location mapping exists (e.g., ground-truth cell-cycle mapping or novoSpaRc mapping), sift.kernels.KnnKernel for neighborhood similarity built from per-cell scores such as zonation gene expression or cell-cycle gene scores, or sift.kernels.RBFKernel / sift.kernels.LaplacianKernel for distance-based similarity. Build the kernel as row-stochastic (rows sum to 1) so the subsequent projection P = K·X is an average over signal-similar neighbors; on the mESC virtual tumor benchmark, six such kernels (ground-truth mapping, KNN on cell-cycle scores, KNN on Cyclum phases, Cyclum-distance kernel, Cyclum-bin mapping, Seurat-label mapping) were compared. Rationale: an inaccurate or coarse-grained mapping kernel may cause loss of the biological signal of interest, so kernel fidelity to the prior knowledge directly controls filtering quality.

## Related tools

- **SiFT (sift-sc)** (Provides the sift.kernels kernel constructors (MappingKernel, KnnKernel, RBFKernel, LaplacianKernel) used to build the cell–cell similarity kernel) — https://github.com/nitzanlab/sift-sc
- **Cyclum** (Supplies Cyclum bins/phases and circular pseudotime used as attributes for cell-cycle kernels in the two-subclone benchmark) — https://github.com/KChen-lab/Cyclum
- **Python** (Implementation language; kernels are instantiated and consumed in Python with scanpy-loaded AnnData inputs)

## Examples

```
import sift; kernel = sift.kernels.KnnKernel(data=adata, attr='zonation_scores'); sft = sift.SiFT(kernel=kernel, copy=False); sft.filter(target='X')
```

## Evaluation signals

- Kernel is row-stochastic (each cell's similarity row sums to 1) so the projection P = K·X is a proper expectation over signal-similar neighbors.
- Kernel choice matches attribute type: MappingKernel for explicit mappings (e.g., novoSpaRc cell-to-location), KnnKernel for score-based neighborhoods, RBF/Laplacian for distances.
- Downstream SiFT filtering with the constructed kernel preserves total variance of the expression matrix (within numerical tolerance) after pseudocount correction α = min(X~).
- On the cell-cycle benchmark, kernels built from ground-truth or Cyclum-based mappings yield higher bio-conservation scores than coarser label-based kernels.
- For the liver zonation task, a KNN kernel from prior-knowledge zonated genes and a MappingKernel from the novoSpaRc mapping should both yield circadian reconstruction Pearson correlations (0.71 and 0.96 with 10 markers) above the unfiltered baseline of 0.44.

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal.
- No ground truth reference is generally available to evaluate kernel/filtering quality; application-specific integration metrics must be used instead.

## Evidence

- [methods] kernel variants and mapping choice: "sift.kernels.RBFKernel
sift.kernels.LaplacianKernel
sift.kernels.KnnKernel
sift.kernels.MappingKernel"
- [discussion] kernel role in SiFT procedure: "The SiFT procedure starts by defining a cell–cell similarity kernel, capturing similarities with respect to the signal to be filtered."
- [intro] kernel definition from attributes: "These attributes are used to compute a cell–cell similarity kernel with respect to the encoded signal"
- [intro] row-stochastic kernel construction step: [evidence span withheld — anchor status: not_found]
- [abstract] kernel-based method description: "we present SiFT, a kernel-based projection method for filtering biological signals in single-cell data"
- [discussion] kernel reliability caveat: "As we showcase in various examples, an inaccurate or coarse-grained mapping may lead to a loss of signal."
