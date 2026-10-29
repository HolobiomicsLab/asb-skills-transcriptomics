---
name: louvain-benchmark-comparison
description: Use when you must quantify whether cell-attribute aware community detection
  (EVA/ELVAR) attains stronger significance for a known simulated or real DA shift
  than standard Louvain clustering on the same single-cell graph.
when_to_use_negative:
- The input is already a feature/abundance table without single-cell counts and no
  cell-cell nearest-neighbour graph can be built
- The study displays strong clustering structure — the article notes ELVAR's improvement
  over the Louvain-analog was 'more pronounced in studies displaying weaker clustering
  structure', so the benchmark may show negligible gain on strongly structured data
- You only need a single clustering result without replicates — the comparison requires
  distributions of DA P-values over repeated runs to be meaningful
tools:
- name: ELVAR
  role: 'Test method: cell-attribute aware EVA clustering (Eva_partitions, SelOptAlpha,
    ProcessEVA) plus DoDA negative-binomial DA testing'
  repo: https://github.com/aet21/ELVAR
- name: EVA
  role: Node-attribute aware community detection algorithm (generalization of Louvain)
    applied to the state-manifold graph with the condition attribute
  repo: https://github.com/aet21/ELVAR
- name: Seurat
  role: Builds the Seurat object, variable features (vst), scaling, PCA, and the k=20
    nearest-neighbour cell-cell graph shared by all compared clustering methods
- name: R
  role: Implementation language for the ELVAR pipeline and the Wilcoxon rank-sum benchmarking
    statistics
  repo: https://github.com/aet21/ELVAR
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:408393f6db15a3745e48e9ad17b8d4760fed921d8367d33903f447941b053d67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - EVA
  - MASS
  - Matrix
  - R
  - Seurat
  - dplyr
  - igraph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# louvain-benchmark-comparison

## Summary

Benchmark differential-abundance testing pipelines that differ only in clustering strategy — cell-attribute aware EVA (ELVAR) versus deterministic (sequential) and non-sequential (stochastic) Louvain analogs — by comparing negative-binomial-regression DA-test P-value distributions over repeated runs with one-sided Wilcoxon rank-sum tests.

## When to use

Use when you must quantify whether cell-attribute aware community detection (EVA/ELVAR) attains stronger significance for a known simulated or real DA shift than standard Louvain clustering on the same single-cell graph.

## When NOT to use

- The input is already a feature/abundance table without single-cell counts and no cell-cell nearest-neighbour graph can be built
- The study displays strong clustering structure — the article notes ELVAR's improvement over the Louvain-analog was 'more pronounced in studies displaying weaker clustering structure', so the benchmark may show negligible gain on strongly structured data
- You only need a single clustering result without replicates — the comparison requires distributions of DA P-values over repeated runs to be meaningful

## Inputs

- Simulated or real scRNA/snRNA-Seq count matrix with a known or expected DA shift between two attribute groups (e.g. perturbed cell-state frequency 0.25 in young vs 0.5 in old mice; neutrophil/moDC shifts in severe vs moderate Covid-19)
- Seurat object with cell-cell nearest-neighbour graph (FindNeighbors, dims=1:8, k.param=20) and a vertex attribute for the clustering condition (e.g. 'age', disease severity); graph must be connected
- Per-method DA statistics from repeated runs: Wald/LRT P-values from DoDA negative-binomial regression (varDA = cell-state, varREP = replicate)

## Outputs

- Per-method DA-test P-value distributions over ~100 runs for ELVAR (optimal alpha), LVdet, and LVnonseq
- One-sided Wilcoxon rank-sum P-values comparing ELVAR against LVdet and LVnonseq (e.g. fig2e_wilcoxon_pvalues.csv)
- Sensitivity comparison of ELVAR versus the Louvain benchmark for the target DA shift (e.g. Supplementary Fig. S5 reproduction)

## How to apply

Run the identical preprocessing (Seurat FindVariableFeatures vst, ScaleData, RunPCA, FindNeighbors with dims=1:8 and k.param=20) once, then cluster the same cell-cell nearest-neighbour state-manifold graph with three methods: Eva_partitions (resolution=1, threshold=1e-7, Vattr.name set to the condition attribute, alpha optimized over 0.1-0.9 via SelOptAlpha requiring purity and modularity within the 75th centile of maximum with a cluster count deviating from alpha=0), the deterministic Louvain baseline (LVdet), and the non-sequential stochastic Louvain baseline (LVnonseq). For each method, select attribute-enriched communities with ProcessEVA (binomial enrichment tests, Bonferroni correction) and run negative-binomial regression DA-testing with DoDA (varDA = cell-state/condition label, varREP = replicate variable such as mouse or patient), repeating over ~100 runs to capture stochasticity. Compare the per-method Wald/LRT P-value distributions with one-sided Wilcoxon rank-sum tests; the benchmark passes if ELVAR's P-values are significantly smaller than both Louvain baselines, mirroring Fig. 2e of the ELVAR paper.

## Related tools

- **ELVAR** (Test method: cell-attribute aware EVA clustering (Eva_partitions, SelOptAlpha, ProcessEVA) plus DoDA negative-binomial DA testing) — https://github.com/aet21/ELVAR
- **EVA** (Node-attribute aware community detection algorithm (generalization of Louvain) applied to the state-manifold graph with the condition attribute) — https://github.com/aet21/ELVAR
- **Seurat** (Builds the Seurat object, variable features (vst), scaling, PCA, and the k=20 nearest-neighbour cell-cell graph shared by all compared clustering methods)
- **R** (Implementation language for the ELVAR pipeline and the Wilcoxon rank-sum benchmarking statistics) — https://github.com/aet21/ELVAR

## Examples

```
eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha=0.8, Vattr.name="age"); da.o <- DoDA(seu.o, sigcl.o, varDA="naive", varREP="mouse");
```

## Evaluation signals

- One-sided Wilcoxon rank-sum test on the DoDA P-value distributions shows ELVAR has significantly smaller P-values than both LVdet and LVnonseq (as in Fig. 2e)
- The optimal alpha from SelOptAlpha satisfies: purity within the 75th centile of maximum, modularity within the 75th centile of maximum, and a cluster count deviating from standard Louvain (alpha=0)
- The identical Seurat graph (FindNeighbors dims=1:8, k.param=20) is used for all three clustering methods, so differences reflect clustering strategy only
- Eva_partitions called with resolution=1, threshold=1e-7, Vattr.name matching the condition attribute; graph verified connected before clustering
- For the Covid-19 validation (task_004), the ELVAR run must reproduce significant Wald/LRT increase in neutrophils and decrease in moDCs in severe cases with higher sensitivity than the Louvain benchmark

## Limitations

- ELVAR incurs increased computational complexity and runtimes compared to Milo or the Louvain-analog (optimizing alpha may require up to 900 runs: 100 runs × 9 alpha values, though parallelizable over nodes)
- ELVAR is more sensitive to false positives in the clustering attribute (sample/phenotype label), which can affect the quality of inferred communities
- Improvement over the Louvain-analog is more pronounced in studies with weaker clustering structure and less noticeable with stronger clustering
- No benchmark against miloDE was performed; the authors flag this as an open question, and batch correction prior to ELVAR rarely improved sensitivity

## Evidence

- [abstract] Using simulated and real single-cell and single-nucleus RNA-Seq datasets, we benchmark ELVAR against an analogous algorithm that uses Louvain for clustering: "Using simulated and real single-cell and single-nucleus RNA-Seq datasets, we benchmark ELVAR against an analogous algorithm that uses Louvain for clustering"
- [methods] eva_partitions call with resolution and threshold parameters: "eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");"
- [methods] SelOptAlpha over the alpha grid: "selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1));"
- [discussion] ELVAR higher sensitivity than competing methods: "ELVAR displayed higher sensitivity than Milo and DA-seq to detect biologically important and plausible DA-shifts"
- [readme] README describes ELVAR package purpose: "ELVAR is an R-package for differential abundance (DA) testing of cell-types in single-cell RNA-Seq data."
