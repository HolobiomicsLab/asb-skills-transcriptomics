---
name: slingshot-unsupervised-pseudotime-trajectory-inference
description: Use when single-cell scRNA-seq data contains multiple branching developmental
  lineages with unknown endpoints, and you need to reconstruct trajectories unsupervised
  from a reduced-dimension representation rather than supervised from known progenitor/terminal
  markers.
when_to_use_negative:
- When a supervised trajectory is appropriate (known root cluster and expected endpoints
  can be given to Slingshot directly) — unsupervised mode is only needed when endpoints
  are unknown
- 'When the low-dimensional representation is derived from unwanted technical variation
  that is not accounted for: ''there is no guarantee that the low-dimensional signal
  extracted by ZINB-WaVE is biologically relevant'', so trajectory inference on such
  factors can be misleading'
- When a PCA-based representation is relied on without verification — in the olfactory
  epithelium contrast, unsupervised PCA-based inference misses the mSUS endpoint
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Slingshot
  role: Unsupervised lineage trajectory and branching inference via minimum spanning
    tree on the reduced-dimension representation
  repo: https://github.com/kstreet13/slingshot
- name: ZINB-WaVE / zinbwave
  role: Produces the first 50 low-dimensional factors used as the Slingshot input
    space, accounting for zero inflation, over-dispersion, and count data without
    prior normalization
  repo: https://github.com/drisso/zinbwave
- name: RSEC (clusterExperiment)
  role: Clusters cells on the ZINB-WaVE factors to define the cluster labels consumed
    by Slingshot
  repo: http://bioconductor.org/packages/release/bioc/html/clusterExperiment.html
- name: scRNAseq
  role: Provides access to the single-cell RNA-seq dataset used for the analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:388c72ea0dab8735c7785a6b649b729372f66e875ed016e0d7e907abae521738
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kstreet13/slingshot
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - RSEC (clusterExperiment)
  - Slingshot
  - ZINB-WaVE / zinbwave
  - scRNAseq
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# slingshot-unsupervised-pseudotime-trajectory-inference

## Summary

Run Slingshot in unsupervised mode on a low-dimensional representation (here, the first 50 ZINB-WaVE factors) with cluster labels from RSEC to infer minimum spanning tree lineage structure and identify terminal (endpoint) clusters, without supplying a known root or trajectory beforehand.

## When to use

Use when single-cell scRNA-seq data contains multiple branching developmental lineages with unknown endpoints, and you need to reconstruct trajectories unsupervised from a reduced-dimension representation rather than supervised from known progenitor/terminal markers.

## When NOT to use

- When a supervised trajectory is appropriate (known root cluster and expected endpoints can be given to Slingshot directly) — unsupervised mode is only needed when endpoints are unknown
- When the low-dimensional representation is derived from unwanted technical variation that is not accounted for: 'there is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant', so trajectory inference on such factors can be misleading
- When a PCA-based representation is relied on without verification — in the olfactory epithelium contrast, unsupervised PCA-based inference misses the mSUS endpoint

## Inputs

- Raw scRNA-seq count matrix (e.g., Fletcher mouse olfactory epithelium, GEO GSE95601)
- ZINB-WaVE low-dimensional factors (first 50 components) computed on raw counts
- Cell cluster labels from RSEC (clusterExperiment) computed on the ZINB-WaVE factors
- Contrast input: first 50 PCA dimensions of the same cells

## Outputs

- Minimum spanning tree lineage structure inferred by Slingshot in unsupervised mode
- Terminal (endpoint) clusters of the inferred MST, expected to be mOSN, mSUS, and MV
- Pseudotime/lineage assignments per cell along each inferred lineage

## How to apply

Load the raw single-cell count matrix (e.g., the Fletcher mouse olfactory epithelium data, GEO GSE95601) and compute a ZINB-WaVE low-dimensional representation retaining the first 50 components; ZINB-WaVE models zero inflation (dropouts), over-dispersion, and the count nature of the data on raw counts without prior normalization, with its cell-level intercept acting as a global-scaling normalization factor. Cluster the cells with RSEC (clusterExperiment) on those 50 ZINB-WaVE factors to obtain cluster labels for Slingshot. Run Slingshot in unsupervised mode on the 50 factors with the RSEC labels to infer a minimum spanning tree lineage structure, then extract the terminal (endpoint) clusters of the MST. As a decision point, compare against the same unsupervised Slingshot run on the first 50 PCA dimensions: on the olfactory epithelium data, the PCA-based trajectory misses the mSUS endpoint whereas the ZINB-WaVE-based run recovers all three mature endpoints (mOSN, mSUS, MV), suggesting the ZINB-WaVE low-dimensional signal better matches the true developmental process.

## Related tools

- **Slingshot** (Unsupervised lineage trajectory and branching inference via minimum spanning tree on the reduced-dimension representation) — https://github.com/kstreet13/slingshot
- **ZINB-WaVE / zinbwave** (Produces the first 50 low-dimensional factors used as the Slingshot input space, accounting for zero inflation, over-dispersion, and count data without prior normalization) — https://github.com/drisso/zinbwave
- **RSEC (clusterExperiment)** (Clusters cells on the ZINB-WaVE factors to define the cluster labels consumed by Slingshot) — http://bioconductor.org/packages/release/bioc/html/clusterExperiment.html
- **scRNAseq** (Provides access to the single-cell RNA-seq dataset used for the analysis)

## Examples

```
BiocManager::install("zinbwave")
```

## Evaluation signals

- The minimum spanning tree inferred on the 50 ZINB-WaVE factors must terminate in exactly the three mature lineage endpoints mOSN, mSUS, and MV
- The PCA-based contrast run (unsupervised Slingshot on first 50 PCA dimensions) is expected to miss the mSUS endpoint — its presence in the ZINB-WaVE run and absence in the PCA run is the discriminating signal
- Cluster labels used by Slingshot must come from RSEC run on the same 50 ZINB-WaVE factors, not from a different feature space
- ZINB-WaVE factors must be computed on raw counts (no prior normalization), since the model's cell-level intercept serves as a global-scaling normalization factor

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant; if unwanted technical variation affects the data and is not accounted for, the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects
- ZINB-WaVE's better performance with respect to PCA comes at a computational cost because a non-convex likelihood must be numerically optimized

## Evidence

- [abstract] the model and its estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA) and zero-inflated: [evidence span withheld — anchor status: not_found]
- [intro] includes a sample-level intercept, which serves as a global-scaling normalization factor: "includes a sample-level intercept, which serves as a global-scaling normalization factor"
- [methods] Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6).: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [readme] After filtering and normalization, we clustered the data using clusterExperiment, performed developmental ordering and inferred lineage trajectories and branching with slingshot.: [evidence span withheld — anchor status: not_found]
- [readme] The data are available on GEO in [GSE95601]: [evidence span withheld — anchor status: not_found]
