---
name: delve-dynamic-module-identification
description: Use when you have a preprocessed cells x genes single-cell matrix (.h5ad
  or similar) and need to select features that preserve cell-state transitions, e.g.
  modules of genes with dynamic expression along a differentiation or activation trajectory.
when_to_use_negative:
- Input is already a curated feature/regulator list and the goal is de novo feature
  selection from raw variance — DELVE could be improved by incorporating previously
  established regulators but is not designed as a supervised annotator
- The goal is to exclude unwanted dynamic modules from trajectory construction — this
  extension is possible but not implemented in DELVE, which currently leverages all
  dynamic modules equally
- Data has not been quality-filtered (e.g. cells with high mitochondrial reads, sparse
  genes) — noisy, information-poor features should be filtered first as a crucial
  preprocessing step prior to trajectory inference
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DELVE
  role: Unsupervised feature selection method performing the dynamic feature-module
    identification (k-NN neighborhood variance, subsampling, KMeans++ module clustering
    across 10 random initializations)
  repo: https://github.com/jranek/delve
- name: Python
  role: Implementation language; DELVE is available as an open-source Python package
    (pip install delve-fs)
  repo: https://github.com/jranek/delve
- name: UMAP (uniform manifold approximation and projection)
  role: Embedding/graph-neighborhood machinery underlying DELVE's cell-similarity
    graph construction for feature-module ranking
- name: SketchKH
  role: Kernel-herding sketching used to subsample representative cells (num_subsamples=1000
    sketched neighborhoods) before module identification
  repo: https://github.com/CompCy-lab/SketchKH
- name: delve_benchmark
  role: Provides run_delve_fs for feature selection and benchmarking against 11 alternative
    methods (Laplacian score, Hotspot, HVG, etc.)
  repo: https://github.com/jranek/delve_benchmark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:5a822554c2ff69119da372fe6385addbce6bccd053535647fa52aa3b83f75973
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - Python
  - SketchKH
  - UMAP (uniform manifold approximation and projection)
  - delve_benchmark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# delve-dynamic-module-identification

## Summary

Identify dynamically expressed feature modules that constitute core regulatory complexes from preprocessed single-cell data using DELVE (dynamic selection of locally covarying features), separating dynamic modules that track the cellular trajectory from static (noisy) modules. This is the bottom-up feature selection step of DELVE, which mitigates confounding sources of variation before trajectory inference.

## When to use

Use when you have a preprocessed cells x genes single-cell matrix (.h5ad or similar) and need to select features that preserve cell-state transitions, e.g. modules of genes with dynamic expression along a differentiation or activation trajectory.

## When NOT to use

- Input is already a curated feature/regulator list and the goal is de novo feature selection from raw variance — DELVE could be improved by incorporating previously established regulators but is not designed as a supervised annotator
- The goal is to exclude unwanted dynamic modules from trajectory construction — this extension is possible but not implemented in DELVE, which currently leverages all dynamic modules equally
- Data has not been quality-filtered (e.g. cells with high mitochondrial reads, sparse genes) — noisy, information-poor features should be filtered first as a crucial preprocessing step prior to trajectory inference

## Inputs

- Preprocessed single-cell count matrix as cells x features annotated data (.h5ad), e.g. quality-filtered, TPM-normalized, log+1 transformed, highly variable gene-selected scRNA-seq data
- Protein iterative immunofluorescence imaging single-cell data (cells x protein features) is also supported
- Hyperparameters: k=10 nearest neighbors, num_subsamples=1000 sketched neighborhoods, n_clusters=5 modules, n_pcs=50 principal components, 10 random KMeans++ initializations

## Outputs

- Feature-to-module assignments partitioning genes into dynamic modules (e.g. dynamic 0 early activation/proliferation, dynamic 1 effector, dynamic 2 memory) and static modules
- Ranked list of dynamically expressed features recapitulating the cellular trajectory
- Approximate cellular trajectory graph built from the dynamic modules
- Per-module statistics such as average sample variance of pairwise expression change relative to the random-assignment null (e.g. exported to cd8_module_string_permutation_results.csv in the STRING co-regulation workflow)

## How to apply

First preprocess the count matrix (e.g. remove cells with >20% mitochondrial reads and genes observed in <3 cells or with <400 counts, TPM-normalize, log+1 transform, select highly variable genes — for the LCMV CD8+ T cell dataset (GSE131847) this yielded 29893 cells x 2000 genes). Then run DELVE feature-module identification on the preprocessed matrix with k=10 nearest neighbors, num_subsamples=1000 sketched neighborhoods (via SketchKH kernel-herding sketching), n_clusters=5 modules after PCA with n_pcs=50, repeating KMeans++ clustering across 10 random initializations. Define dynamic modules as those whose average sample variance of pairwise expression change exceeds the random-assignment null, taking the intersection across the 10 runs; modules below the null are static. In the LCMV CD8+ T cell case this yielded three dynamic modules (dynamic 0: early activation/proliferation; dynamic 1: effector; dynamic 2: memory) plus one static module; the dynamic modules can then be validated by permutation tests of STRING co-regulation, with significant p-values (0.001) for dynamic modules and no evidence (p=1.0) for the static module.

## Related tools

- **DELVE** (Unsupervised feature selection method performing the dynamic feature-module identification (k-NN neighborhood variance, subsampling, KMeans++ module clustering across 10 random initializations)) — https://github.com/jranek/delve
- **Python** (Implementation language; DELVE is available as an open-source Python package (pip install delve-fs)) — https://github.com/jranek/delve
- **UMAP (uniform manifold approximation and projection)** (Embedding/graph-neighborhood machinery underlying DELVE's cell-similarity graph construction for feature-module ranking)
- **SketchKH** (Kernel-herding sketching used to subsample representative cells (num_subsamples=1000 sketched neighborhoods) before module identification) — https://github.com/CompCy-lab/SketchKH
- **delve_benchmark** (Provides run_delve_fs for feature selection and benchmarking against 11 alternative methods (Laplacian score, Hotspot, HVG, etc.)) — https://github.com/jranek/delve_benchmark

## Examples

```
from delve_benchmark.tl import run_delve_fs; features = run_delve_fs(adata, k=10, num_subsamples=1000, n_clusters=5, n_pcs=50)
```

## Evaluation signals

- Dynamic modules identified from real LCMV CD8+ T cell data show significant STRING co-regulation (permutation p-value = 0.001, R=1000 size-matched random gene sets), while the static module shows no evidence (p-value = 1.0), matching Fig. 5b
- The identified modules are stable across the 10 random KMeans++ initializations (intersection across runs defines the dynamic set)
- On simulated scRNA-seq (Splatter/SymSim), similarity-based feature selection including DELVE should achieve the highest recovery of differentially expressed genes, highest cell type classification accuracy, and most accurate estimation of individual cell progression
- Output feature-to-module assignment table has the expected schema (features x module labels, one row per selected gene) and the preprocessed matrix matches expected dimensions (29893 cells x 2000 genes for the GSE131847 CD8+ T cell dataset)

## Limitations

- Feature selection can greatly bias interpretation of the underlying cellular trajectory; careful consideration should be made when performing feature selection for trajectory analysis
- DELVE currently leverages all dynamic modules equally for constructing the approximate cellular trajectory; extending the framework to exclude unwanted dynamic modules is possible but not implemented
- DELVE could be improved by incorporating a set of previously established regulators
- Similarity-based feature selection methods like DELVE rely on the cell-similarity graph from the full dataset and can fail to identify relevant features when the number of noisy features outweighs the number of informative ones

## Evidence

- [readme] DELVE is an unsupervised feature selection method for identifying a representative subset of dynamically-expressed molecular features that recapitulate cellular trajectories from single-cell data: "DELVE is an unsupervised feature selection method for identifying a representative subset of dynamically-expressed molecular features that recapitulate cellular trajectories from single-cell data"
- [intro] DELVE uses a bottom-up approach to mitigate the effect of unwanted sources of variation confounding feature selection: "DELVE uses a bottom-up approach to mitigate the effect of unwanted sources of variation confounding feature selection and trajectory inference, and instead models cell states from dynamic feature"
- [abstract] DELVE is available as an open-source python package: "DELVE is available as an open-source python package"
- [intro] appropriate filtering of noisy, information-poor, or irrelevant features can serve as a crucial preprocessing step: "appropriate filtering of noisy, information-poor, or irrelevant features can serve as a crucial and necessary step for cell type identification, inference of dynamic phenotypes"
- [discussion] performing feature selection with DELVE prior to performing trajectory inference improved the identification and resolution of gene programs: "performing feature selection with DELVE prior to performing trajectory inference improved the identification and resolution of gene programs"
