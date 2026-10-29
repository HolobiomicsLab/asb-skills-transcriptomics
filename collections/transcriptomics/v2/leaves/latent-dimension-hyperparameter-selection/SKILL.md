---
name: latent-dimension-hyperparameter-selection
description: Use when benchmarking or deploying paired single-cell multi-omics integration
  methods and a defensible number of latent dimensions must be chosen before comparing
  methods on silhouette score, ARI, and kNN purity.
when_to_use_negative:
- Data are unpaired or single-omics only, where the multi-omics factor models benchmarked
  here (Mowgli, MOFA+, Cobolt, Multigrate) do not apply
- No downstream metric computation is planned and k can be justified by other domain
  knowledge; a full grid search adds cost with no decision signal
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Mowgli
  role: Integration method whose latent dimensionality is tuned via the benchmark
    grid
  repo: https://github.com/cantinilab/mowgli
- name: MOFA+
  role: Baseline trained with a grid over number of factors (15 chosen overall; 5
    for Liu)
- name: Cobolt
  role: Baseline trained with a grid over latent dimensions (30 chosen overall; 5
    for Liu)
- name: Multigrate
  role: Baseline trained with a grid over latent dimensions (50 chosen overall; 15
    for Liu)
- name: Scanpy
  role: kNN graph (k=20) and Leiden clustering across resolutions used to score each
    k setting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:8debd1e6c11f3ddd2e4ba88cd8cefe1cd6450eb9fcff86679ab48b3939b6a56f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/mowgli
  tools:
  - Cobolt
  - MOFA+
  - Mowgli
  - Multigrate
  - Python
  - Scanpy
  - Seurat v4
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# latent-dimension-hyperparameter-selection

## Summary

Benchmarks the number of latent dimensions (k) of each integration method (Mowgli, integrative NMF, MOFA+, Cobolt, Multigrate) across multiple paired multi-omics datasets and selects the overall best-performing k per method for the benchmark.

## When to use

Use when benchmarking or deploying paired single-cell multi-omics integration methods and a defensible number of latent dimensions must be chosen before comparing methods on silhouette score, ARI, and kNN purity.

## When NOT to use

- Data are unpaired or single-omics only, where the multi-omics factor models benchmarked here (Mowgli, MOFA+, Cobolt, Multigrate) do not apply
- No downstream metric computation is planned and k can be justified by other domain knowledge; a full grid search adds cost with no decision signal

## Inputs

- Preprocessed Muon MuData objects for paired multi-omics datasets (Liu scCAT-seq, PBMC 10X, OP Multiome, OP CITE, BM CITE)
- Mowgli, MOFA+, Cobolt, and Multigrate model configurations parameterized by the number of latent dimensions

## Outputs

- Chosen latent-dimension value per method (e.g., k=50 for Mowgli, 15 factors for MOFA+, 30 for Cobolt, 50 for Multigrate; reduced k for Liu-derived data)
- Benchmark metrics (silhouette score, ARI, kNN purity) per k value supporting the selection

## How to apply

For each method (Mowgli, integrative NMF, MOFA+, Cobolt, Multigrate) and each dataset (Liu scCAT-seq, PBMC 10X, OP Multiome, OP CITE, BM CITE), train the model across a grid of latent-dimension values, cluster the resulting embeddings, and score embedding/clustering quality with silhouette score, Adjusted Rand Index across Leiden resolutions, and kNN purity. Choose the overall best-performing number of latent dimensions per method, as done in the paper: 50 for Mowgli (5 for Liu-derived data), 15 factors for MOFA+ (5 for Liu), 30 for Cobolt (5 for Liu), and 50 for Multigrate (15 for Liu). The rationale is that fixed but per-method-optimized k avoids confounding the method comparison with a single method's sensitivity to k. Mowgli does not itself offer a straightforward way to define k, so explicit testing of several k values is required.

## Related tools

- **Mowgli** (Integration method whose latent dimensionality is tuned via the benchmark grid) — https://github.com/cantinilab/mowgli
- **MOFA+** (Baseline trained with a grid over number of factors (15 chosen overall; 5 for Liu))
- **Cobolt** (Baseline trained with a grid over latent dimensions (30 chosen overall; 5 for Liu))
- **Multigrate** (Baseline trained with a grid over latent dimensions (50 chosen overall; 15 for Liu))
- **Scanpy** (kNN graph (k=20) and Leiden clustering across resolutions used to score each k setting)

## Examples

```
import mowgli; model = mowgli.model.MowgliModel(n_latent_dimensions=50, rho_rna=0.01, rho_atac=0.1); model.train(mdata)
```

## Evaluation signals

- Each method's selected k is the setting achieving the best silhouette score / ARI / kNN purity on its datasets (e.g., Mowgli k=50, or k=5 on Liu)
- Chosen-k embeddings yield Leiden clusterings with sensible ARI across resolutions rather than degenerate single-cluster outputs
- The same selection criterion is applied uniformly across all benchmarked methods so k does not bias the comparison
- Silhouette scores are in a valid [-1, 1] range and purity scores fall within [0, 1]

## Limitations

- Mowgli does not offer a straightforward approach to defining the number of latent dimensions; several values of k should be tested
- Optimal transport is expensive to compute, so sweeping k for Mowgli on large datasets (e.g., BM CITE with 29,803 cells) is costly without a GPU
- Smaller k is needed for small datasets (k=5 for Liu-derived data), so k values are not transferable across datasets of very different scale

## Evidence

- [results] For each method, we choose the overall best-performing number of latent dimensions (see Methods): "For each method, we choose the overall best-performing number of latent dimensions (see Methods)"
- [discussion] Mowgli lacks a direct way to set k: [evidence span withheld — anchor status: not_found]
- [methods] Mowgli training optimizes decomposition with L-BFGS: [evidence span withheld — anchor status: not_found]
- [readme] Mowgli installed as Python package in the scverse ecosystem: "Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy."
