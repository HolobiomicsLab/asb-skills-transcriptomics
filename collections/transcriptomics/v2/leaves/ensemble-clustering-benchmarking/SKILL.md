---
name: ensemble-clustering-benchmarking
description: Use when evaluating whether decision-level integration (scTriangulate)
  of conflicting clustering solutions outperforms consensus ensemble methods (HGPA,
  MCLA, HBGF) against a ground truth or reference annotation.
when_to_use_negative:
- There is no ground truth or trusted reference annotation against which Homogeneity/Completeness/V-Measure
  can be computed
- Only one clustering solution exists (no conflicting annotations to ensemble or mix-and-match)
- The goal is data-level integration into a joint latent space rather than benchmarking
  decision-level reconciliation of existing cluster labels
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: scTriangulate
  role: Decision-level integration method under evaluation; run via lazy_run with
    default settings (win_fraction_cutoff=0.25) on the same annotations fed to ensemble
    methods
  repo: https://github.com/frankligy/scTriangulate
- name: ClusterEnsembles
  role: Provides the HGPA, MCLA, and HBGF consensus ensemble baselines, run with default
    options and CSPA excluded
  repo: https://github.com/tsano430/ClusterEnsembles
- name: Splatter
  role: Generates the simulated ground-truth dataset (splatSimulateGroup, batchCells=3000,
    5 clusters c1-c5, de.prob up to 0.4) used in the simulation benchmark
- name: scanpy
  role: Preprocessing and per-modality Leiden clustering (scanpy_recipe) producing
    the conflicting annotation inputs to benchmark
- name: Seurat-WNN
  role: 'State-of-the-art multimodal baseline (3-way WNN: 50 PCs RNA, 18 PCs ADT,
    2-50 LSI dimensions ATAC) compared in the TEA-Seq benchmark'
- name: TotalVI
  role: Joint RNA+ADT variational autoencoder baseline (top-4000 variable genes, 400
    epochs, scvi-tools 0.14.3) compared in the TEA-Seq benchmark
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:ef2723c14e1f074b9316ac456d7d0f480a47d6d53a24def915fd3f87940010dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - Seurat-WNN
  - Splatter
  - TotalVI
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# ensemble-clustering-benchmarking

## Summary

Benchmark scTriangulate's decision-level mix-and-match integration against conventional consensus ensemble clustering (HGPA, MCLA, HBGF) using Homogeneity, Completeness, and V-Measure against a known ground truth or reference annotation.

## When to use

Use when evaluating whether decision-level integration (scTriangulate) of conflicting clustering solutions outperforms consensus ensemble methods (HGPA, MCLA, HBGF) against a ground truth or reference annotation.

## When NOT to use

- There is no ground truth or trusted reference annotation against which Homogeneity/Completeness/V-Measure can be computed
- Only one clustering solution exists (no conflicting annotations to ensemble or mix-and-match)
- The goal is data-level integration into a joint latent space rather than benchmarking decision-level reconciliation of existing cluster labels

## Inputs

- Four (or more) partially conflicting cluster-annotation columns (e.g., Broad/Median/Fine/Excessive Splatter groupings, or 9 Leiden r=1,2,3 solutions from RNA, ATAC, ADT modalities)
- Ground-truth cluster labels (Splatter simulated c1-c5) or reference annotations (Azimuth human-pbmc level-2)
- AnnData with properly normalized expression (log-CPTT RNA, CLR ADT) and adata.obsm['X_umap']

## Outputs

- simulation_benchmark_metrics.csv with Homogeneity/Completeness/V-Measure rows for scTriangulate, HGPA, MCLA, and HBGF plus recovered cluster counts
- Per-method V-Measure ranking against Seurat-WNN, TotalVI, and MCLA/HGPA/HBGF baselines on the TEA-Seq tri-modal benchmark
- Final scTriangulate per-cell population assignments (win_fraction_cutoff=0.25) for comparison with the ensemble outputs

## How to apply

Generate or collect multiple partially conflicting clustering solutions spanning different resolutions and modalities, then run the Python ClusterEnsembles package with default options (CSPA excluded) using HGPA, MCLA, and HBGF, and run scTriangulate 0.9.0 lazy_run with win_fraction_cutoff=0.25 on the same inputs. For the simulation benchmark, build four annotations that only partially overlap the Splatter ground truth (Broad merging c1+c2+c3, Median merging c2+c3, Fine subdividing c5, Excessive subdividing c2 and c3); for the multimodal benchmark, generate 9 Leiden solutions (resolutions 1, 2, 3 for RNA, ATAC, ADT from TEA-Seq) and compare against Seurat-WNN, TotalVI, and Azimuth PBMC level-2 reference annotations. Compute Homogeneity, Completeness, and V-Measure per sklearn conventions (H(C), H(C|K), H(K|C), harmonic mean) for each method, tabulate per-method metrics alongside the number of recovered clusters, and check that scTriangulate's V-Measure matches or exceeds the ensemble methods. The key rationale is that ensemble methods select a 'consensus' whereas scTriangulate picks the most stable solution through coalitional Shapley iteration, so the benchmark must quantify whether mix-and-matching recovers ground truth without introducing artifacts.

## Related tools

- **scTriangulate** (Decision-level integration method under evaluation; run via lazy_run with default settings (win_fraction_cutoff=0.25) on the same annotations fed to ensemble methods) — https://github.com/frankligy/scTriangulate
- **ClusterEnsembles** (Provides the HGPA, MCLA, and HBGF consensus ensemble baselines, run with default options and CSPA excluded) — https://github.com/tsano430/ClusterEnsembles
- **Splatter** (Generates the simulated ground-truth dataset (splatSimulateGroup, batchCells=3000, 5 clusters c1-c5, de.prob up to 0.4) used in the simulation benchmark)
- **scanpy** (Preprocessing and per-modality Leiden clustering (scanpy_recipe) producing the conflicting annotation inputs to benchmark)
- **Seurat-WNN** (State-of-the-art multimodal baseline (3-way WNN: 50 PCs RNA, 18 PCs ADT, 2-50 LSI dimensions ATAC) compared in the TEA-Seq benchmark)
- **TotalVI** (Joint RNA+ADT variational autoencoder baseline (top-4000 variable genes, 400 epochs, scvi-tools 0.14.3) compared in the TEA-Seq benchmark)

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3']); sctri.lazy_run()
```

## Evaluation signals

- simulation_benchmark_metrics.csv contains Homogeneity, Completeness, and V-Measure rows for all four methods (scTriangulate, HGPA, MCLA, HBGF)
- scTriangulate's V-Measure matches or exceeds all ensemble methods against the Splatter c1-c5 ground truth, recovering 5 clusters without artifacts
- On TEA-Seq, scTriangulate attains the highest V-Measure relative to Seurat-WNN, TotalVI, and MCLA/HGPA/HBGF at every evaluated resolution
- The final annotation set recovers rare populations (NK CD56+ bright, B-memory, HSCP) against the Azimuth PBMC L2 reference
- Ensemble baselines fail to mix-and-match broad and excessive inputs into the ground-state-truth clustering, unlike scTriangulate

## Limitations

- Conventional consensus ensemble algorithms (HGPA, MCLA, HBGF) cannot recover the ground-state-truth clustering by mixing-and-matching under- and over-clustered inputs and may include artifacts
- Integration of cluster predictions from dozens of sources significantly increases runtime; Shapley selection is automatically replaced by combined rank-based selection when over 15 annotation sets are provided
- Programmatic default analytical settings (e.g., win_fraction_cutoff=0.25, abs_thresh=10) may not be optimal for all study designs

## Evidence

- [abstract] Rather than ensemble approaches which select the "consensus", scTriangulate picks the most stable solution through coalitional iteration.: "Rather than ensemble approaches which select the "consensus", scTriangulate picks the most stable solution through coalitional iteration."
- [methods] ClusterEnsembles default options: "using the Python package ClusterEnsembles with default options (https://github.com/tsano430/ClusterEnsembles)."
- [methods] lazy_run on four conflicting annotations: "We apply the scTriangulate lazy_run function in version 0.9.0 on these four conflicting annotation solutions to derive the final cluster assignment (default settings)."
- [intro] Shapley value from coalitional game theory: "scTriangulate systematically mixes-and-matches cellular clusters generated by different algorithms spanning different modalities and resolutions to arrive at optimal clustering solutions"
- [readme] README minimum example: "sctri = ScTriangulate(dir='./output',adata=adata,query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3'])"
