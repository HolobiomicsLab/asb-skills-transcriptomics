---
name: single-cell-rsec-clustering
description: Use when single-cell count data must be partitioned into cell-type/identity
  clusters that will be supplied as labels to a trajectory-inference method such as
  Slingshot, especially after projecting cells into a ZINB-WaVE low-dimensional space.
when_to_use_negative:
- The goal is a purely supervised cell-type assignment from known markers rather than
  data-driven clustering of unlabeled cells.
- Cells have not been filtered for poor-quality samples, contaminants, and doublets
  — RSEC clusters would capture technical artifacts (the oeHBC-diff pipeline explicitly
  filters and renormalizes before clustering).
- The analysis needs a continuous pseudotime ordering without discrete cluster anchors,
  in which case direct trajectory inference on the low-dimensional factors may suffice.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: clusterExperiment
  role: Provides RSEC, the consensus resampling clustering used to define cluster
    labels for Slingshot on the olfactory epithelium data
  repo: https://github.com/epurdom/clusterExperiment
- name: RSEC
  role: Consensus clustering algorithm run on the 50 ZINB-WaVE factors to produce
    cluster labels for unsupervised Slingshot lineage reconstruction
- name: zinbwave
  role: Generates the 50-factor low-dimensional representation of raw counts in which
    RSEC clustering is performed
  repo: https://github.com/drisso/zinbwave
- name: Slingshot
  role: Consumes the RSEC cluster labels to infer minimum-spanning-tree lineages over
    the ZINB-WaVE factors
  repo: https://github.com/kstreet13/slingshot
- name: SCONE
  role: Normalization of the filtered olfactory epithelium data prior to clustering
  repo: https://github.com/YosefLab/scone
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:b5e8f8ce698446e453f6c03503389f6a7f88049b9b66f7e94e6f17d368ee8d18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/epurdom/clusterExperiment
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - RSEC
  - SCONE
  - Slingshot
  - clusterExperiment
  - zinbwave
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-rsec-clustering

## Summary

Cluster single-cell RNA-seq cells with RSEC (from the clusterExperiment framework) to obtain discrete cluster labels that serve as anchors for Slingshot lineage inference, as done on the mouse olfactory epithelium (GSE95601) data using ZINB-WaVE factors.

## When to use

Use when single-cell count data must be partitioned into cell-type/identity clusters that will be supplied as labels to a trajectory-inference method such as Slingshot, especially after projecting cells into a ZINB-WaVE low-dimensional space.

## When NOT to use

- The goal is a purely supervised cell-type assignment from known markers rather than data-driven clustering of unlabeled cells.
- Cells have not been filtered for poor-quality samples, contaminants, and doublets — RSEC clusters would capture technical artifacts (the oeHBC-diff pipeline explicitly filters and renormalizes before clustering).
- The analysis needs a continuous pseudotime ordering without discrete cluster anchors, in which case direct trajectory inference on the low-dimensional factors may suffice.

## Inputs

- single-cell RNA-seq raw count matrix (e.g., mouse olfactory epithelium, GEO GSE95601 ExpressionSet/SummarizedExperiment)
- low-dimensional cell representation such as the first 50 ZINB-WaVE factors
- filtered, quality-controlled, and (re)normalized cell-by-gene data from upstream pipeline steps

## Outputs

- per-cell cluster labels from RSEC/clusterExperiment
- cluster-level annotation aligned to known cell types (HBC, immature/mature OSN, SUS, MV) for downstream Slingshot lineage inference

## How to apply

Load the single-cell count matrix (e.g., the Fletcher mouse olfactory epithelium data from GEO GSE95601) and, if following the ZINB-WaVE route, compute the first 50 ZINB-WaVE factors from raw counts so that zero inflation, over-dispersion, and the count nature of the data are modeled without prior normalization. Run RSEC — the clusterExperiment-based resampling consensus clustering — on the 50-dimensional ZINB-WaVE factors to assign each cell to a cluster; in the companion olfactory stem-cell analysis the authors used clusterExperiment for clustering and Slingshot (via oeHBCdiff_clust.R / oeHBCdiff_slingshot.Rmd) for developmental ordering and lineage reconstruction. The rationale is that cluster labels must capture the underlying differentiation structure, and clustering in the ZINB-WaVE space (rather than PCA) preserves the signal needed to recover all mature lineages. Decision point: verify that the resulting cluster set contains clusters corresponding to the known cell types (HBCs through to mOSN, mSUS, and MV endpoints) before feeding labels to Slingshot; if mature-cell clusters are missing, adjust resolution/cluster-merging parameters and rerun.

## Related tools

- **clusterExperiment** (Provides RSEC, the consensus resampling clustering used to define cluster labels for Slingshot on the olfactory epithelium data) — https://github.com/epurdom/clusterExperiment
- **RSEC** (Consensus clustering algorithm run on the 50 ZINB-WaVE factors to produce cluster labels for unsupervised Slingshot lineage reconstruction)
- **zinbwave** (Generates the 50-factor low-dimensional representation of raw counts in which RSEC clustering is performed) — https://github.com/drisso/zinbwave
- **Slingshot** (Consumes the RSEC cluster labels to infer minimum-spanning-tree lineages over the ZINB-WaVE factors) — https://github.com/kstreet13/slingshot
- **SCONE** (Normalization of the filtered olfactory epithelium data prior to clustering) — https://github.com/YosefLab/scone

## Examples

```
./oeHBCdiff_2_renorm_clust_devO_DE.sh  # runs oeHBCdiff_clust.R (clusterExperiment/RSEC clustering) then oeHBCdiff_slingshot.Rmd (lineage inference)
```

## Evaluation signals

- The MST inferred by Slingshot on the RSEC clusters from the 50 ZINB-WaVE factors must terminate in exactly the three mature lineage endpoints: mOSN, mSUS, and MV.
- A contrast run on the first 50 PCA dimensions should fail to recover the mSUS endpoint, confirming the ZINB-WaVE-based clustering is the discriminating factor.
- Cluster labels must be non-degenerate (multiple clusters present, no single mega-cluster) so Slingshot can build a branching MST.
- Clusters must map onto biologically expected populations — HBCs at the root and mature mOSN/mSUS/MV populations toward the tips — consistent with the cell-type assignments of the oeHBC-diff analysis.

## Limitations

- Clustering quality depends on the input low-dimensional representation; PCA-based clustering missed the mSUS lineage endpoint, so the choice of feature space directly controls lineage recovery.
- Biological contaminants and doublets must be removed before clustering; co-expression of differentiated cell markers was used to identify known doublets in the source analysis.
- RSEC involves resampling-based consensus clustering and can be computationally intensive on large cell numbers.

## Evidence

- [other] Cluster the cells with RSEC on the 50 ZINB-WaVE factors to define cluster labels for Slingshot.: [evidence span withheld — anchor status: not_found]
- [readme] README describes clustering and slingshot in the olfactory epithelium project: [evidence span withheld — anchor status: not_found]
- [readme] README dependency on clusterExperiment version: [evidence span withheld — anchor status: not_found]
- [abstract] Article: ZINB-WaVE factors are computed from raw counts without prior normalization: "low-dimensional representations of the data that account for zero inflation (dropouts), over-dispersion, and the count nature of the data"
- [readme] Data are available on GEO in GSE95601: [evidence span withheld — anchor status: not_found]
