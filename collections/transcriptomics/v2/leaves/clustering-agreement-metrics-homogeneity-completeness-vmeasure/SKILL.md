---
name: clustering-agreement-metrics-homogeneity-completeness-vmeasure
description: Use when two or more single-cell clustering solutions (or one solution
  plus a reference atlas annotation like Azimuth L2) must be compared quantitatively
  to determine which recovers known cell populations.
when_to_use_negative:
- Reference annotations are unavailable — the metrics require a ground-truth labeling,
  so purely unsupervised cluster-stability assessment (e.g., reassign, TFIDF, SCCAF
  scores) is the wrong tool
- Comparing clusterings of different cell sets — Homogeneity/Completeness/V-Measure
  are only defined on the same N cells with paired labels
- Population naming/label semantics is the goal — the paper notes naming remains largely
  non-standardized and these metrics treat labels as exchangeable integers
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn metrics
  role: Compute Homogeneity, Completeness, and V-Measure between candidate cluster
    labels and the Azimuth L2 reference
- name: scTriangulate
  role: Produces the final per-cell population assignments (win_fraction_cutoff=0.25,
    version 0.9.0) being benchmarked
  repo: https://github.com/frankligy/scTriangulate
- name: ClusterEnsembles
  role: Generates HGPA, MCLA, and HBGF ensemble baseline clusterings (CSPA excluded)
    to score against the reference
  repo: https://github.com/tsano430/ClusterEnsembles
- name: Seurat-WNN
  role: 'Multimodal integration baseline: 3-way weighted nearest neighbor clustering
    (50 PCs RNA, 18 PCs ADT, 2-50 LSI ATAC) across varying resolutions'
- name: TotalVI
  role: RNA+ADT variational-autoencoder baseline (scvi-tools 0.14.3) with Leiden clustering
    on the joint embedding
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:ecee581cebfc11420235b5514213ffe0fe27ceb1194e91ff67a6f8fa0abdf44a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - Seurat-WNN
  - TotalVI
  - scTriangulate
  - scanpy
  - scikit-learn metrics
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# clustering-agreement-metrics-homogeneity-completeness-vmeasure

## Summary

Benchmark clustering solutions (e.g., scTriangulate, Seurat-WNN, TotalVI, MCLA/HGPA/HBGF ensembles) against a reference annotation such as the Azimuth PBMC level-2 labels by computing Homogeneity, Completeness, and V-Measure scores to quantify agreement.

## When to use

Use when two or more single-cell clustering solutions (or one solution plus a reference atlas annotation like Azimuth L2) must be compared quantitatively to determine which best recovers known cell populations.

## When NOT to use

- Reference annotations are unavailable — the metrics require a ground-truth labeling, so purely unsupervised cluster-stability assessment (e.g., reassign, TFIDF, SCCAF scores) is the wrong tool
- Comparing clusterings of different cell sets — Homogeneity/Completeness/V-Measure are only defined on the same N cells with paired labels
- Population naming/label semantics is the goal — the paper notes naming remains largely non-standardized and these metrics treat labels as exchangeable integers

## Inputs

- Reference cell annotations (e.g., Azimuth PBMC level-2 labels mapped to post-QC TEA-Seq nuclei)
- Clustering solution assignments per cell (Leiden r=1,2,3 per modality, Seurat-WNN, TotalVI, HGPA/MCLA/HBGF ensemble labels, scTriangulate final populations)
- QC-filtered TEA-Seq AnnData with RNA/ADT/ATAC counts (8,213 nuclei after min_gene>=300, min_counts>=500, pct_counts_mt<=20%, min_atac_peaks>=1000)

## Outputs

- Table of Homogeneity, Completeness, and V-Measure scores for every clustering solution against the Azimuth L2 reference
- Ranking of methods (scTriangulate vs. Seurat-WNN, TotalVI, ensemble baselines) with rare-population recovery annotations

## How to apply

Treat one annotation set as the ground-truth reference (e.g., Azimuth human-pbmc level-2 labels derived from post-QC raw UMI counts) and the other as the candidate clustering. For each candidate — here the 9 TEA-Seq Leiden solutions from RNA (log-CPTT, top-3000 HVGs, 15 PCs), ATAC (top-60000 peaks, 100 PCs), and ADT (CLR normalization, 15 PCs) at resolutions 1–3, plus Seurat 3-way WNN, TotalVI (top-4000 genes, 400 epochs), and ClusterEnsembles HGPA/MCLA/HBGF — compute Homogeneity, Completeness, and their harmonic mean (V-Measure) against the reference. The key decision point is that V-Measure is the headline metric: a solution is superior if it attains the highest V-Measure across resolutions and methods. Also tabulate recovery of rare populations (NK CD56+ bright, B-memory, HSCP) as a biological sanity check alongside the numeric scores.

## Related tools

- **scikit-learn metrics** (Compute Homogeneity, Completeness, and V-Measure between candidate cluster labels and the Azimuth L2 reference)
- **scTriangulate** (Produces the final per-cell population assignments (win_fraction_cutoff=0.25, version 0.9.0) being benchmarked) — https://github.com/frankligy/scTriangulate
- **ClusterEnsembles** (Generates HGPA, MCLA, and HBGF ensemble baseline clusterings (CSPA excluded) to score against the reference) — https://github.com/tsano430/ClusterEnsembles
- **Seurat-WNN** (Multimodal integration baseline: 3-way weighted nearest neighbor clustering (50 PCs RNA, 18 PCs ADT, 2-50 LSI ATAC) across varying resolutions)
- **TotalVI** (RNA+ADT variational-autoencoder baseline (scvi-tools 0.14.3) with Leiden clustering on the joint embedding)

## Examples

```
from sklearn.metrics import homogeneity_completeness_v_measure; h, c, v = homogeneity_completeness_v_measure(labels_pred=sctri.obs['sctri_annotation'], labels_true=adata.obs['azimuth_L2'])
```

## Evaluation signals

- V-Measure of scTriangulate exceeds that of Seurat-WNN, TotalVI, and all MCLA/HGPA/HBGF ensemble solutions across evaluated resolutions
- Each score table entry has three components (Homogeneity, Completeness, V-Measure) computed over the same 8,213 QC-filtered nuclei with paired labels
- Completeness and Homogeneity are consistent with the reported V-Measure as its harmonic mean (2*H*C/(H+C))
- Final scTriangulate annotation set recovers rare populations (NK CD56+ bright, B-memory, HSCP) consistent with the Azimuth L2 reference
- Scores are stable when the reference is re-derived from post-QC raw UMI counts via the Azimuth human-pbmc app

## Limitations

- Metrics depend on reference annotation quality; the Azimuth L2 labels themselves are a prediction, not truth
- No single resolution identifies the most discrete cell states across modalities, so scores must be compared across multiple resolutions rather than one
- Population naming remains largely non-standardized, complicating label-matching before metric computation
- Integration of dozens of clustering sources significantly increases runtime (Shapley selection is replaced by rank-based selection above 15 annotation sets)

## Evidence

- [methods] Compute Homogeneity, Completeness, and V-Measure of every clustering solution against the Azimuth L2 reference, and tabulate results alongside the final scTriangulate populations: [evidence span withheld — anchor status: not_found]
- [methods] Derive Azimuth L2 annotations from post-QC raw UMI counts: [evidence span withheld — anchor status: not_found]
- [results] scTriangulate showed improved performance over all evaluated clustering solutions and resolutions: [evidence span withheld — anchor status: not_found]
- [intro] No single resolution identifies most discrete cell states: "It is likely that no single-resolution in single-cell analyses will identify the most discrete cell states across different modalities"
- [readme] README: scTriangulate reconciles conflicting clustering solutions: "scTriangulate is a Python package to mix-and-match conflicting clustering results in single cell analysis, and generate reconciled clustering solutions."
