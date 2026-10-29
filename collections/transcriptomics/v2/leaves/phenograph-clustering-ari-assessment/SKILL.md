---
name: phenograph-clustering-ari-assessment
description: Use when 2D embeddings of preprocessed CyTOF data (arcsinh X/5, MinMaxScaler
  to [0,1]) must be partitioned into cell populations and quantitatively compared
  to manual gating labels via ARI, e.g. benchmarking dimensionality-reduction quality
  on BMMC/Samusik.
when_to_use_negative:
- The dataset has no manually gated cell-population labels, so ARI cannot be computed
  — use unsupervised metrics or the Mantel-test PCC on centroid distances instead
- The data are raw FCS files that have not yet been arcsinh-transformed and MinMax-scaled;
  clustering unscaled cytometry intensities is out of scope for this workflow
- The goal is to assess preservation of global inter-cluster geometry rather than
  cluster recoverability — the paper uses the Mantel test with Pearson correlation
  for that question
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: phenograph
  role: Graph-based clustering of the 2D embeddings to recover cell populations for
    ARI comparison
- name: ivis
  role: Produces the 2D embeddings (k=15) that PhenoGraph clusters; benchmarking target
    vs t-SNE
  repo: https://github.com/beringresearch/ivis
- name: Python
  role: Implementation environment for preprocessing, embedding, PhenoGraph clustering,
    and ARI computation
  repo: https://github.com/beringresearch/ivis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:b2d4b919b783a5387a67992b4834081fdc1dde5d16ca74d6be4033175cc557c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - phenograph
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# phenograph-clustering-ari-assessment

## Summary

Cluster 2D single-cell embeddings (e.g. ivis or t-SNE embeddings of CyTOF data) with PhenoGraph and score agreement against manually gated cell-population labels using the Adjusted Rand Index (ARI), as done in the Szubert et al. 2019 ivis benchmark on the BMMC (Levine-32-dim) and Samusik CyTOF datasets.

## When to use

Use when 2D embeddings of preprocessed CyTOF data (arcsinh X/5, MinMaxScaler to [0,1]) must be partitioned into cell populations and quantitatively compared to manual gating labels via ARI, e.g. benchmarking dimensionality-reduction quality on BMMC/Samusik.

## When NOT to use

- The dataset has no manually gated cell-population labels, so ARI cannot be computed — use unsupervised metrics or the Mantel-test PCC on centroid distances instead
- The data are raw FCS files that have not yet been arcsinh-transformed and MinMax-scaled; clustering unscaled cytometry intensities is out of scope for this workflow
- The goal is to assess preservation of global inter-cluster geometry rather than cluster recoverability — the paper uses the Mantel test with Pearson correlation for that question

## Inputs

- 2D embedding matrix (ivis or t-SNE output) for each subsampled CyTOF dataset
- Vector of manually gated cell-population labels aligned to the same cells (e.g. k=14 populations for Levine-32-dim BMMC)
- CyTOF expression matrices (e.g. BMMC/Levine-32-dim FCS-derived data, Samusik) preprocessed with arcsinh(X/5) and MinMaxScaler

## Outputs

- PhenoGraph cluster/community assignments for each embedding
- Per-subsample ARI values against manual labels
- Median ARI aggregated across subsamples (reference: BMMC 0.97, Samusik 0.45)
- Summary table such as pcc_ari_benchmark_results.csv

## How to apply

Preprocess the CyTOF matrices with an arcsinh transformation (X/5) followed by MinMaxScaler scaling to the [0,1] range, then compute 2D embeddings (ivis with k=15 or t-SNE). Run PhenoGraph — the graph-based cluster-detection algorithm introduced by Levine et al. (2015) — on each embedding to recover cell communities without using label information. Compare the recovered cluster assignments to the manually gated cell-population labels (14 populations for Levine-32-dim BMMC) using the Adjusted Rand Index. Repeat over many random 10,000-cell subsamples (the paper's benchmark uses 100 subsamples) and aggregate the median ARI; reported reference medians are ARI BMMC ≈ 0.97 and Samusik ≈ 0.45. Persist per-subsample and median ARI values alongside PCC results (e.g. pcc_ari_benchmark_results.csv).

## Related tools

- **phenograph** (Graph-based clustering of the 2D embeddings to recover cell populations for ARI comparison)
- **ivis** (Produces the 2D embeddings (k=15) that PhenoGraph clusters; benchmarking target vs t-SNE) — https://github.com/beringresearch/ivis
- **Python** (Implementation environment for preprocessing, embedding, PhenoGraph clustering, and ARI computation) — https://github.com/beringresearch/ivis

## Examples

```
X = MinMaxScaler().fit_transform(np.arcsinh(X/5)); embeddings = Ivis(embedding_dims=2, k=15).fit_transform(X); communities, Q, _ = phenograph.cluster(embeddings); ari = adjusted_rand_score(labels, communities)
```

## Evaluation signals

- Median ARI across the 100 random 10,000-cell subsamples should approach the reported values: ≈0.97 for ivis on BMMC and ≈0.45 for ivis on Samusik
- ARI lies in [-1, 1]; values near 1 indicate near-perfect recovery of the manually gated populations (e.g. the k=14 BMMC populations)
- PhenoGraph should be run on the embedding only, with manual labels used strictly post hoc for ARI scoring — no label leakage into clustering
- Each subsample yields one ARI per embedding method; check the distribution of per-subsample ARIs before taking the median
- ARI results should be written together with PCC results to a single summary file (pcc_ari_benchmark_results.csv) for cross-metric comparison

## Limitations

- In Levine-32-dim BMMC, cluster labels are available for only 39% (104,184) of 265,627 cells, with the rest unassigned, constraining which cells can be scored
- ARI of ≈0.45 on Samusik shows only partial cluster agreement even for the better embedding method; ARI alone does not capture global structure fidelity (covered by the Mantel PCC)
- t-SNE embeddings can yield spurious clusters with clean boundaries in the presence of random noise, inflating apparent cluster counts and depressing ARI

## Evidence

- [readme] PhenoGraph, a new graph-based algorithm for detecting clusters in high-dimensional mass cytometry data: [evidence span withheld — anchor status: external_doc]
- [readme] Manually gated BMMC cluster labels availability: [evidence span withheld — anchor status: external_doc]
- [methods] Levine dataset provenance for CyTOF benchmarks: "The Levine dataset was obtained from `Data-Driven Phenotypic Dissection of AML Reveals Progenitor-like Cells that Correlate with Prognosis"
- [methods] Preprocessing required before clustering embeddings: "X = MinMaxScaler().fit_transform(X)"
