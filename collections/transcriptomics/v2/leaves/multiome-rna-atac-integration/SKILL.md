---
name: multiome-rna-atac-integration
description: Use when a paired RNA+ATAC (multiome) single-cell dataset needs decision-level
  integration of per-modality Leiden clusterings into stable cell populations, with
  V-Measure/Completeness benchmarking against individual resolutions.
when_to_use_negative:
- When a joint latent-space integration (e.g., Seurat-WNN or TotalVI) is explicitly
  desired — scTriangulate integrates at the decision level and makes no assumption
  about raw value distributions
- When only a single modality or a single clustering resolution is available — there
  are no conflicting annotation sets to mix-and-match
- When the dataset is CITE-Seq/ADT-based needing CLR normalization via concat_rna_and_other
  — this skill's parameters (1000 ATAC peaks, 100,000 peaks HVR selection) are ATAC-specific
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: scTriangulate
  role: Decision-level mix-and-match of RNA and ATAC Leiden clusterings via Shapley
    value; lazy_run with win_fraction_cutoff=0.35 and add_metrics='tfidf5'
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: QC, log CPTT normalization, HVR/HVG selection, neighbor graphs, PCA, Leiden
    clustering at resolutions 1–3 for both modalities
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:c5b705ff11acc51cf97d1ef2be8866fea5d43484f6fbb7a6cc30960a49e5dff4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# multiome-rna-atac-integration

## Summary

Reproduce the 10x Multiome PBMC (RNA+ATAC) decision-level integration with scTriangulate, combining RNA and ATAC Leiden clusterings at resolutions 1–3 into reconciled cell populations (classical/intermediate/inflammatory CD14 monocytes, CD4 T-cell subsets, CASC15-defined ILCs), and evaluate them with Homogeneity, Completeness, and V-Measure.

## When to use

Use when a paired RNA+ATAC (multiome) single-cell dataset needs decision-level integration of per-modality Leiden clusterings into stable cell populations, with V-Measure/Completeness benchmarking against individual resolutions.

## When NOT to use

- When a joint latent-space integration (e.g., Seurat-WNN or TotalVI) is explicitly desired — scTriangulate integrates at the decision level and makes no assumption about raw value distributions
- When only a single modality or a single clustering resolution is available — there are no conflicting annotation sets to mix-and-match
- When the dataset is CITE-Seq/ADT-based needing CLR normalization via concat_rna_and_other — this skill's parameters (1000 ATAC peaks, 100,000 peaks HVR selection) are ATAC-specific

## Inputs

- 10x Genomics Multiome PBMC sparse matrix counts h5 file (RNA + ATAC)
- Processed AnnData objects: RNA (log CPTT, top 3000 HVGs, n_pc=15) and ATAC (top 100,000 peaks, n_pc=100)
- Per-modality Leiden annotation columns at resolutions 1, 2, and 3 in adata.obs
- Silver-standard reference cell-identity annotations for evaluation

## Outputs

- Reconciled scTriangulate cluster assignments (Shapley-based final annotation per nucleus)
- UMAP visualization of integrated clusters (adata.obsm['X_umap'])
- Per-cluster modality contributions (rank-weighted RNA/ATAC marker sums over top-20 markers)
- multiome_pbmc_sctriangulate_metrics.csv with Homogeneity, Completeness, and V-Measure values

## How to apply

Download the Multiome PBMC h5 from 10x Genomics and run QC: filter RNA cells with min_genes<300, min_counts<500, or pct_counts_mt>20%, and require >=1000 ATAC peaks per nucleus, retaining ~10,991 nuclei. Normalize both modalities to logarithmic CPTT, select hypervariable features (RNA top 3000 genes, ATAC top 100,000 peaks), build neighbor graphs (n_neighbor=15), reduce dimensions (RNA n_pc=15, ATAC n_pc=100), and run Leiden at resolutions 1, 2, and 3 per modality. Concatenate the processed RNA and ATAC AnnData objects via scTriangulate.preprocessing.concat_rna_and_other, then invoke scTriangulate lazy_run (v0.9.0) with win_fraction_cutoff=0.35 and add_metrics='tfidf5' so Shapley-based coalitional iteration mixes and matches the six conflicting annotation sets. Overlay reference-assigned identities and compute per-cluster modality contributions as the rank-weighted sum of RNA and ATAC markers among each cluster's top-20 marker list. Finally, compute Homogeneity, Completeness, and V-Measure (harmonic mean) of the scTriangulate solution versus silver-standard references and each individual clustering solution, writing metrics to multiome_pbmc_sctriangulate_metrics.csv.

## Related tools

- **scTriangulate** (Decision-level mix-and-match of RNA and ATAC Leiden clusterings via Shapley value; lazy_run with win_fraction_cutoff=0.35 and add_metrics='tfidf5') — https://github.com/frankligy/scTriangulate
- **scanpy** (QC, log CPTT normalization, HVR/HVG selection, neighbor graphs, PCA, Leiden clustering at resolutions 1–3 for both modalities)

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['rna_leiden_1','rna_leiden_2','rna_leiden_3','atac_leiden_1','atac_leiden_2','atac_leiden_3']); sctri.lazy_run(win_fraction_cutoff=0.35, add_metrics='tfidf5')
```

## Evaluation signals

- multiome_pbmc_sctriangulate_metrics.csv exists with Homogeneity, Completeness, and V-Measure fields populated
- scTriangulate's V-Measure/Completeness exceed those of each individual RNA or ATAC Leiden clustering (reproducing Supplementary Fig. 5d)
- ~10,991 nuclei retained after QC (RNA min_genes<300, min_counts<500, pct_counts_mt>20%; >=1000 ATAC peaks per nucleus)
- Marker analysis recovers classical, intermediate, and inflammatory CD14 monocytes, distinct CD4 T-cell subsets, and CASC15-defined innate lymphoid cells
- Unstable clusters are pruned per win_fraction_cutoff=0.35 and reclassified by nearest-neighbor strategy

## Limitations

- Integration of cluster predictions from dozens of sources (resolutions, modalities, reference annotations) significantly increases runtime; over 15 annotation sets triggers rank-based selection instead of Shapley
- Programmatic default analytical settings may not be optimal for all study designs
- Naming of the resulting populations remains largely non-standardized

## Evidence

- [methods] qc_params: "Multiome PBMC sparse matrix counts h5 data was downloaded from the 10× Genomics website"
- [methods] lazy_run: "We apply the scTriangulate lazy_run function in version 0.9.0 on these four conflicting annotation solutions to derive the final cluster assignment"
- [methods] concat: "we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function"
- [intro] shapley: "leverage the Shapley value from coalitional game theory to determine the relative importance of each set of cluster annotations"
- [abstract] metrics: "When evaluated on diverse multimodal technologies, scTriangulate outperforms alternative approaches to identify high-confidence cell-populations and modality-specific subtypes."
- [readme] usage: "scTriangulate leverages cooperative game theory (Shapley Value) in conjunction with complimentary stability metrics (i.e. reassign score, TFIDF score and SCCAF score)"
- [readme] docker: [evidence span withheld — anchor status: external_doc]
