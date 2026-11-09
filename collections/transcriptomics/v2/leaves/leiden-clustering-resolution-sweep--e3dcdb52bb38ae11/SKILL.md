---
name: leiden-clustering-resolution-sweep
description: Use when you have QC-filtered multimodal single-cell data (RNA, ATAC,
  ADT/CITE-Seq) and need per-modality Leiden annotations at multiple resolutions as
  players for downstream decision-level integration (scTriangulate) or ensemble clustering.
when_to_use_negative:
- You already have high-confidence reference annotations or label-transfer results
  and do not need unsupervised per-modality resolution sweeps
- You need a joint latent-space multimodal integration (use Seurat-WNN or TotalVI
  instead) rather than decision-level mix-and-match of independent clustering results
- A single fixed resolution is explicitly sufficient for your cell-type discovery
  goal and integration across modalities/resolutions is not required
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Normalization, HVG selection, PCA, and Leiden clustering of each modality
    within the scTriangulate/scanpy 1.7.2 preprocessing framework
  repo: https://github.com/frankligy/scTriangulate
- name: scTriangulate
  role: Provides scanpy_recipe, CLR_normalization, and concat_rna_and_other wrappers
    to normalize, cluster, and merge the modality-specific AnnData objects
  repo: https://github.com/frankligy/scTriangulate
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:6848aea4c8fc9a5b0fc95c72d55be36a0d12c059b7b63683dd2ab79606ac3b1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - TotalVI
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# leiden-clustering-resolution-sweep

## Summary

Generates multiple conflicting Leiden clustering solutions per single-cell modality at several resolutions (e.g., r=1,2,3) after modality-specific normalization and PCA/LSI, producing the per-modality annotation sets that scTriangulate later mix-and-matches at the decision level. The resolution sweep matters because different informative cell populations emerge at different resolutions.

## When to use

Use when you have QC-filtered multimodal single-cell data (RNA, ATAC, ADT/CITE-Seq) and need per-modality Leiden annotations at multiple resolutions as players for downstream decision-level integration (scTriangulate) or ensemble clustering.

## When NOT to use

- You already have high-confidence reference annotations or label-transfer results and do not need unsupervised per-modality resolution sweeps
- You need a joint latent-space multimodal integration (use Seurat-WNN or TotalVI instead) rather than decision-level mix-and-match of independent clustering results
- A single fixed resolution is explicitly sufficient for your cell-type discovery goal and integration across modalities/resolutions is not required

## Inputs

- QC-filtered RNA count matrix (min_gene >= 300, min_counts >= 500, pct_counts_mt <= 20%)
- QC-filtered ATAC peak matrix (min_atac_peaks >= 1000)
- ADT antibody-derived tag count CSV
- AnnData objects per modality with normalized expression (adata.X)

## Outputs

- Per-modality Leiden annotation columns at r=1, 2, 3 (e.g., sctri_rna_leiden_1, sctri_atac_leiden_2) in adata.obs
- Merged tri-modal AnnData (via concat_rna_and_other) with UMAP coordinates in adata.obsm['X_umap']
- Nine conflicting annotation sets ready as scTriangulate query inputs or ClusterEnsembles baselines

## How to apply

Normalize each modality separately before clustering: RNA with log-CPTT on the top 3,000 highly variable genes reduced to 15 PCs; ATAC with log-CPTT on the top 60,000 highly variable peaks reduced to 100 PCs; ADT with center log-ratio (CLR) normalization to 15 PCs (scTriangulate.preprocessing.Normalization.CLR_normalization and scanpy_recipe wrappers). Run Leiden clustering at resolutions 1, 2, and 3 within scanpy/scTriangulate for each modality, then merge the modality AnnData objects with scTriangulate.preprocessing.concat_rna_and_other. Each resolution's annotation becomes a separate obs column (e.g., sctri_rna_leiden_1) passed to ScTriangulate's query argument. The rationale is that no single resolution identifies the most discrete cell states across modalities, so the sweep supplies diverse candidates that Shapley-value-based coalitional iteration later reconciles.

## Related tools

- **scanpy** (Normalization, HVG selection, PCA, and Leiden clustering of each modality within the scTriangulate/scanpy 1.7.2 preprocessing framework) — https://github.com/frankligy/scTriangulate
- **scTriangulate** (Provides scanpy_recipe, CLR_normalization, and concat_rna_and_other wrappers to normalize, cluster, and merge the modality-specific AnnData objects) — https://github.com/frankligy/scTriangulate

## Examples

```
from sctriangulate.preprocessing import scanpy_recipe; sc.tl.leiden(adata, resolution=1); sc.tl.leiden(adata, resolution=2, key_added='leiden_2'); sc.tl.leiden(adata, resolution=3, key_added='leiden_3')
```

## Evaluation signals

- Each modality yields exactly three Leiden solutions (r=1, 2, 3), giving 9 annotation sets for the TEA-Seq tri-modal case (3 modalities x 3 resolutions)
- RNA adata.X is log-CPTT normalized on top 3,000 HVGs; ATAC on top 60,000 variable peaks; ADT is CLR-normalized — verify per-modality normalization before clustering
- Merged AnnData retains cells passing QC (8,213 nuclei for the TEA-Seq dataset with the stated filters) and has no adata.raw attribute, which would interfere with scanpy marker-gene calculation
- The swept annotation sets, when benchmarked against the Azimuth PBMC level-2 reference via Homogeneity/Completeness/V-Measure, should be outperformed by the scTriangulate mix-and-match solution

## Limitations

- No single resolution identifies the most discrete cell states across modalities, so single-resolution results alone are unreliable for final annotation
- Integration of cluster predictions from dozens of sources (resolutions, modalities, reference annotations) significantly increases runtime
- The programmatic defaults (resolution range, HVG/PC counts) may not be optimal for all study designs

## Evidence

- [intro] It is likely that no single-resolution in single-cell analyses will identify the most discrete cell states across different modalities: "It is likely that no single-resolution in single-cell analyses will identify the most discrete cell states across different modalities"
- [methods] All the preprocessing and QC steps were conducted using scTriangulate version 0.9.0 and scanpy version 1.7.2.: "All the preprocessing and QC steps were conducted using scTriangulate version 0.9.0 and scanpy version 1.7.2."
- [methods] we conducted Center log-ratio (CLR) normalization using scTriangulate.preprocessing.Normalization.CLR_normalization function: "we conducted Center log-ratio (CLR) normalization using scTriangulate.preprocessing.Normalization.CLR_normalization function"
- [methods] we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other: "we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other"
- [readme] don't have adata.raw attribute, if have, del adata.raw, this will interfere how scanpy calculate marker genes: [evidence span withheld — anchor status: external_doc]
