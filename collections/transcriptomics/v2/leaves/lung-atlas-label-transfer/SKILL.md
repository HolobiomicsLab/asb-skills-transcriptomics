---
name: lung-atlas-label-transfer
description: Use when integrating competing cell-atlas annotation labels from multiple
  lung scRNA/snRNA studies onto one base dataset, and when benchmarking integrated
  annotations against independent references (Azimuth CellRef, HLCA).
when_to_use_negative:
- Data are not lung single-cell datasets or the goal is de novo unsupervised clustering
  without any reference atlas annotations to transfer
- More than 15 annotation sets are provided, where scTriangulate automatically substitutes
  Shapley selection with combined rank-based selection to reduce runtime (desired
  behavior differs)
- The task requires integration at the data level (joint latent space or geometric
  graph, e.g. Seurat-WNN, TotalVI) rather than reconciling conflicting label assignments
  at the decision level
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scTriangulate
  role: Decision-level reconciliation of the four transferred atlas annotations using
    Shapley value and stability metrics (win_fraction_cutoff=0.4, add_metrics='tfidf5')
  repo: https://github.com/frankligy/scTriangulate
- name: cellHarmony
  role: 'Two-step label transfer: MarkerFinder top-60 marker centroids from non-diseased
    controls (Pearson rho > 0.2) followed by base self-reclassification'
- name: scanpy
  role: Preprocessing framework hosting scTriangulate; log CPTT normalization, UMAP,
    and AnnData handling (v1.7.2)
  repo: https://github.com/frankligy/scTriangulate
- name: ICGS2
  role: Marker generation underlying the cellHarmony mapping step
- name: scikit-learn
  role: Computing Homogeneity, Completeness, and V-Measure of integrated annotations
    against HLCA/LungMAP references
- name: ClusterEnsembles
  role: Ensemble-clustering baseline for comparison against the scTriangulate solution
  repo: https://github.com/tsano430/ClusterEnsembles
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:9ca6547c7746f7d77c7e06ba9736617f3b7a6a636c45f92c6c94922a14e93f72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - ICGS2
  - cellHarmony
  - scTriangulate
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lung-atlas-label-transfer

## Summary

Transfer cell-population annotations from four independent lung atlas studies (Melms GSE161383, Adams GSE171524, Travaglini GSE136831, Wang EGAS00001004344) plus HLCA and LungMAP CellRef references onto a Wang 9-donor snRNA-Seq base dataset using cellHarmony's two-step marker-centroid mapping, then reconcile all transferred labels with scTriangulate and benchmark agreement against Azimuth CellRef and HLCA via Homogeneity, Completeness, and V-Measure.

## When to use

Use when integrating competing cell-atlas annotation labels from multiple lung scRNA/snRNA studies onto one base dataset, and when benchmarking integrated annotations against independent references (Azimuth CellRef, HLCA).

## When NOT to use

- Data are not lung single-cell datasets or the goal is de novo unsupervised clustering without any reference atlas annotations to transfer
- More than 15 annotation sets are provided, where scTriangulate automatically substitutes Shapley selection with combined rank-based selection to reduce runtime (desired behavior differs)
- The task requires integration at the data level (joint latent space or geometric graph, e.g. Seurat-WNN, TotalVI) rather than reconciling conflicting label assignments at the decision level

## Inputs

- Lung cell-population assignments and count matrices from GSE161383 (Melms), GSE171524 (Adams), GSE136831 (Travaglini), EGAS00001004344 (Wang snRNA-Seq, 9 donors)
- HLCA reference from Azimuth human-lung-v2 (app.azimuth.hubmapconsortium.org/app/human-lung-v2)
- LungMAP CellRef 1.0 reference
- MNP-Verse mononuclear phagocyte centroids from healthy controls
- Base AnnData (.h5ad) with raw UMI counts and obsm['X_umap']

## Outputs

- AnnData with multiple transferred atlas annotation columns in .obs
- scTriangulate reconciled cluster assignment per cell
- lung_atlas_agreement_metrics.csv with Homogeneity, Completeness, and V-Measure vs HLCA and LungMAP references

## How to apply

Download lung cell-population assignments and count data from GSE161383, GSE171524, GSE136831, and EGAS00001004344, selecting the Wang et al. 9-donor snRNA-Seq dataset as the base. Map each query dataset onto the base using cellHarmony's two-step strategy: MarkerFinder top-60 marker centroids computed from non-diseased controls with Pearson correlation rho > 0.2, followed by self-reclassification of the base dataset; apply the same mapping to the HLCA (Azimuth human-lung-v2) and LungMAP CellRef 1.0 references. Attach all transferred annotations and UMAP coordinates to the base AnnData with scTriangulate.preprocessing.add_annotation and add_umap, normalize raw UMI counts to log CPTT, then run scTriangulate (v0.9.0, scanpy 1.7.2) with win_fraction_cutoff = 0.4 and add_metrics = 'tfidf5' to reconcile the conflicting labels at the decision level. Compute Homogeneity, Completeness, and V-Measure (sklearn conventions) of the scTriangulate solution against HLCA and LungMAP reference assignments, including subdivided dendritic-cell and macrophage populations, and compare against an ensemble-clustering baseline built with the ClusterEnsembles package.

## Related tools

- **scTriangulate** (Decision-level reconciliation of the four transferred atlas annotations using Shapley value and stability metrics (win_fraction_cutoff=0.4, add_metrics='tfidf5')) — https://github.com/frankligy/scTriangulate
- **cellHarmony** (Two-step label transfer: MarkerFinder top-60 marker centroids from non-diseased controls (Pearson rho > 0.2) followed by base self-reclassification)
- **scanpy** (Preprocessing framework hosting scTriangulate; log CPTT normalization, UMAP, and AnnData handling (v1.7.2)) — https://github.com/frankligy/scTriangulate
- **ICGS2** (Marker generation underlying the cellHarmony mapping step)
- **scikit-learn** (Computing Homogeneity, Completeness, and V-Measure of integrated annotations against HLCA/LungMAP references)
- **ClusterEnsembles** (Ensemble-clustering baseline for comparison against the scTriangulate solution) — https://github.com/tsano430/ClusterEnsembles

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['melms','adams','travaglini','wang','hlca','lungmap'], add_metrics='tfidf5'); sctri.lazy_run(win_fraction_cutoff=0.4)
```

## Evaluation signals

- V-Measure values in lung_atlas_agreement_metrics.csv must equal 2*h*c/(h+c) from the tabulated Homogeneity and Completeness
- All three agreement metrics (Homogeneity, Completeness, V-Measure) must lie in [0,1]
- scTriangulate's agreement scores against Azimuth CellRef and HLCA should exceed the ensemble-clustering baseline (ClusterEnsembles)
- Each cell in the final scTriangulate output carries exactly one reconciled annotation (binary assignment matrix X of shape I*N)
- Cells with winning fraction below win_fraction_cutoff = 0.4 are flagged as unstable rather than confidently assigned

## Limitations

- Programmatic default analytical settings (e.g. win_fraction_cutoff) may not be optimal for all study designs
- Integration of cluster predictions from dozens of sources significantly increases runtime; Shapley selection is replaced by rank-based selection when over 15 annotation sets are provided
- Population naming remains largely non-standardized, complicating cross-atlas label comparison

## Evidence

- [methods] obtained from four independent studies (GSE161383, GSE171524, GSE136831, EGAS00001004344): "obtained from four independent studies (GSE161383, GSE171524, GSE136831, EGAS00001004344)"
- [methods] add_annotation and add_umap: "The obtained annotations and umap coordinates were added to the existing AnnData by scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions"
- [methods] winning fraction pruning: "By default, wfc < 0.25 will be labeled as unstable clusters."
- [discussion] reference-classification use case: "the integration of diverse cellular annotations from independent studies (i.e., reference-classification), multimodal measurements, distinct clustering algorithms"
- [methods] software versions: "All the preprocessing and QC steps were conducted using scTriangulate version 0.9.0 and scanpy version 1.7.2."
- [intro] Shapley decision-level integration: "We consider each set of cluster annotations as an individual player and leverage the Shapley value from coalitional game theory to determine the relative importance of each set of cluster annotations"
- [readme] README minimum example: "sctri = ScTriangulate(dir='./output',adata=adata,query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3'])"
