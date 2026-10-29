---
name: leiden-resolution-annotation-generation
description: 'Use when preparing scTriangulate input or comparing clustering resolutions:
  count-matrix AnnData needs QC filtering (min_gene>300, min_count>500, pct_counts_mt<20%),
  CPTT log-normalization, 3000 HVGs, and Leiden labels at several resolutions.'
when_to_use_negative:
- Prior annotation columns already exist (e.g., Azimuth label-transfer results); scTriangulate
  can ingest those directly via add_annotation instead of re-clustering
- The data is non-RNA modality (ADT, ATAC), which requires CLR normalization via sctriangulate.preprocessing.Normalization
  rather than log CPTT
- You need a single, fixed-resolution clustering result rather than a panel of conflicting
  annotations for integration
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scanpy
  role: Provides the Leiden clustering, QC, normalization, HVG selection, PCA, neighbors,
    and UMAP executed inside the scanpy_recipe wrapper
  repo: https://github.com/frankligy/scTriangulate
- name: scTriangulate
  role: Supplies scanpy_recipe preprocessing wrapper; version 0.9.0 used for all preprocessing
    and QC in the paper
  repo: https://github.com/frankligy/scTriangulate
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:a3d95330673f9fb3d01a449d278eb1b298e2d965a528cb1b52580027aaa5a82d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# leiden-resolution-annotation-generation

## Summary

Generates a panel of conflicting per-cell Leiden cluster annotations at multiple resolutions (e.g., 1, 2, 3) on QC-filtered, log-normalized single-cell RNA data so that downstream decision-level integrators like scTriangulate can mix-and-match them. Different resolutions surface different informative cell populations, which is precisely the conflict the integration step resolves.

## When to use

Use when preparing scTriangulate input or comparing clustering resolutions: count-matrix AnnData needs QC filtering (min_gene>300, min_count>500, pct_counts_mt<20%), CPTT log-normalization, 3000 HVGs, and Leiden labels at several resolutions.

## When NOT to use

- Prior annotation columns already exist (e.g., Azimuth label-transfer results); scTriangulate can ingest those directly via add_annotation instead of re-clustering
- The data is non-RNA modality (ADT, ATAC), which requires CLR normalization via sctriangulate.preprocessing.Normalization rather than log CPTT
- You need a single, fixed-resolution clustering result rather than a panel of conflicting annotations for integration

## Inputs

- Raw 10x Genomics count matrix (h5 or h5ad), e.g. pbmc_10k_v3
- AnnData with adata.X empty/raw counts, per-cell mitochondrial percentage computable
- Resolutions at which to cluster (paper used 1, 2, 3)

## Outputs

- AnnData with filtered/normalized X (log CPTT), 3000 hypervariable genes selected, PCA/neighbors computed (n_neighbors=15, n_pcs=50)
- adata.obsm['X_umap'] coordinates
- One obs annotation column per resolution (e.g., sctri_rna_leiden_1, sctri_rna_leiden_2, sctri_rna_leiden_3) suitable for ScTriangulate query argument
- QC-filtered cell set (~11,022 cells for pbmc_10k_v3 under the stated thresholds)

## How to apply

Load the raw 10x count matrix (e.g., the pbmc_10k_v3 h5) into scanpy and filter cells using min_gene > 300, min_count > 500, and pct_counts_mt < 20% (this yielded ~11,022 cells in the paper's PBMC example). Log-normalize to CPTT, select the top 3000 hypervariable genes, and run the scTriangulate.preprocessing.scanpy_recipe wrapper (scanpy 1.7.2 with scTriangulate 0.9.0) to run PCA and neighbors (n_neighbors = 15, n_pcs = 50). Run Leiden clustering sequentially at multiple resolutions — in the paper, resolutions 1, 2, and 3 — storing each as a separate annotation column (e.g., sctri_rna_leiden_1, sctri_rna_leiden_2, sctri_rna_leiden_3) in adata.obs, and record adata.obsm['X_umap'] for later visualization. The rationale is that no single resolution identifies the most discrete cell states, so supplying several conflicting annotations gives the Shapley-based decision engine candidates to mix-and-match rather than forcing one resolution a priori.

## Related tools

- **scanpy** (Provides the Leiden clustering, QC, normalization, HVG selection, PCA, neighbors, and UMAP executed inside the scanpy_recipe wrapper) — https://github.com/frankligy/scTriangulate
- **scTriangulate** (Supplies scanpy_recipe preprocessing wrapper; version 0.9.0 used for all preprocessing and QC in the paper) — https://github.com/frankligy/scTriangulate

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3'])  # annotations from scanpy_recipe Leiden at res 1,2,3
```

## Evaluation signals

- Cell count after QC falls near the expected range (~11,022 cells for pbmc_10k_v3 with min_gene>300, min_count>500, pct_counts_mt<20%)
- adata.obs contains one distinct Leiden annotation column per requested resolution, each with no unlabeled cells
- Higher resolutions yield strictly more clusters than lower ones (res 3 > res 2 > res 1), producing genuine label conflicts for downstream Shapley selection
- adata.obsm['X_umap'] exists so downstream lazy_run can auto-generate visualizations
- adata.raw has been deleted, since it interferes with how scanpy calculates marker genes in scTriangulate

## Limitations

- Resolutions used are programmatic defaults; the article notes these settings will likely not be optimal for all study designs
- adata must not carry a .raw attribute, or marker-gene calculation downstream is disturbed
- Runtime grows as more annotation sets are supplied to the downstream integrator

## Evidence

- [methods] Leiden clustering at multiple resolutions: "This step may also include running Leiden clustering sequentially through scTriangulate when no prior annotations exist."
- [methods] scanpy_recipe wrapper: "This is performed using the preprocessing module wrapper function scanpy_recipe."
- [readme] Multiple conflicting annotation columns required: [evidence span withheld — anchor status: external_doc]
- [readme] scanpy_recipe generates UMAP for visualization: [evidence span withheld — anchor status: external_doc]
