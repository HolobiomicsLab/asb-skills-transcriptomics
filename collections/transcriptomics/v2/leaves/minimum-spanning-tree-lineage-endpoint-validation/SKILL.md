---
name: minimum-spanning-tree-lineage-endpoint-validation
description: Use when inferring developmental lineages from single-cell RNA-seq clusters
  and you must verify that unsupervised trajectory endpoints match known mature cell
  types before trusting the lineage structure.
when_to_use_negative:
- The cell types expected as lineage endpoints are not known or cannot be named in
  advance, leaving no acceptance criterion for endpoint validation
- Input data are already normalized or transformed embeddings rather than raw counts,
  undermining the ZINB-WaVE model's assumption of unnormalized count data
- The biological system is linear (unbranched), so minimum spanning tree branching
  and multi-endpoint validation add no information over simple pseudotime ordering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: zinbwave
  role: Fits the zero-inflated negative binomial model on raw counts to produce the
    low-dimensional factors (first 50 components) that define the space on which clustering
    and trajectory inference are run
  repo: https://github.com/drisso/zinbwave
- name: ZINB-WaVE
  role: The underlying signal-extraction method that accounts for zero inflation,
    over-dispersion, and count nature of the data without preliminary normalization
  repo: https://github.com/drisso/zinbwave
- name: Slingshot
  role: Infers the minimum spanning tree lineage structure and branching in unsupervised
    mode from cluster labels and the ZINB-WaVE factor space
  repo: https://github.com/kstreet13/slingshot
- name: RSEC
  role: Clusters cells (via clusterExperiment) on the 50 ZINB-WaVE factors to supply
    the cluster labels required by Slingshot
  repo: https://github.com/epurdom/clusterExperiment
- name: scRNAseq
  role: Bioconductor package used to access the single-cell RNA-seq count data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:4865245f8792b562d1a1b6c772cdae81294a1cf6b5b671d49fd7fed5a129e606
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - RSEC
  - Slingshot
  - ZINB-WaVE
  - scRNAseq
  - zinbwave
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# minimum-spanning-tree-lineage-endpoint-validation

## Summary

Validates single-cell lineage inference by checking that a Slingshot minimum spanning tree, run in unsupervised mode on ZINB-WaVE low-dimensional factors, terminates in the expected mature cell-type endpoints (mOSN, mSUS, MV) on the mouse olfactory epithelium dataset.

## When to use

Use when inferring developmental lineages from single-cell RNA-seq clusters and you must verify that unsupervised trajectory endpoints match known mature cell types before trusting the lineage structure.

## When NOT to use

- The cell types expected as lineage endpoints are not known or cannot be named in advance, leaving no acceptance criterion for endpoint validation
- Input data are already normalized or transformed embeddings rather than raw counts, undermining the ZINB-WaVE model's assumption of unnormalized count data
- The biological system is linear (unbranched), so minimum spanning tree branching and multi-endpoint validation add no information over simple pseudotime ordering

## Inputs

- Raw single-cell RNA-seq count matrix from the Fletcher mouse olfactory epithelium dataset (GEO GSE95601)
- First 50 ZINB-WaVE low-dimensional factors computed on raw counts
- RSEC cluster labels per cell (from clusterExperiment)
- Contrast embedding: first 50 PCA dimensions of the same cells

## Outputs

- Slingshot minimum spanning tree over clusters with inferred lineage structure
- List of terminal (endpoint) clusters of the inferred tree
- Endpoint validation verdict: pass if endpoints are exactly {mOSN, mSUS, MV}, fail otherwise
- Comparative PCA-based trajectory used as a negative control (expected to miss mSUS)

## How to apply

Load the raw Fletcher mouse olfactory epithelium count matrix (GEO GSE95601) into R and fit ZINB-WaVE, retaining the first 50 latent factors so the representation accounts for zero inflation (dropouts), over-dispersion, and the count nature of the data without prior normalization. Cluster cells with RSEC (clusterExperiment) on those 50 factors, then run Slingshot in unsupervised mode on the factors with the RSEC labels to infer the minimum spanning tree. Extract the terminal clusters of the tree and check they correspond exactly to mOSN, mSUS, and MV. As a decision-point contrast, repeat the identical unsupervised Slingshot run on the first 50 PCA dimensions; the PCA-based trajectory is expected to miss the mSUS endpoint, so all three endpoints on ZINB-WaVE factors is the acceptance criterion.

## Related tools

- **zinbwave** (Fits the zero-inflated negative binomial model on raw counts to produce the low-dimensional factors (first 50 components) that define the space on which clustering and trajectory inference are run) — https://github.com/drisso/zinbwave
- **ZINB-WaVE** (The underlying signal-extraction method that accounts for zero inflation, over-dispersion, and count nature of the data without preliminary normalization) — https://github.com/drisso/zinbwave
- **Slingshot** (Infers the minimum spanning tree lineage structure and branching in unsupervised mode from cluster labels and the ZINB-WaVE factor space) — https://github.com/kstreet13/slingshot
- **RSEC** (Clusters cells (via clusterExperiment) on the 50 ZINB-WaVE factors to supply the cluster labels required by Slingshot) — https://github.com/epurdom/clusterExperiment
- **scRNAseq** (Bioconductor package used to access the single-cell RNA-seq count data)

## Examples

```
library(zinbwave); zinb <- zinbFit(counts, K=50); factors <- getFactors(zinb); sce <- slingshot(factors, clusterLabels = rsecLabels)
```

## Evaluation signals

- The minimum spanning tree inferred on the 50 ZINB-WaVE factors terminates in exactly the three mature lineage endpoints mOSN, mSUS, and MV
- The same unsupervised Slingshot run on the first 50 PCA dimensions fails to recover the mSUS endpoint, confirming the contrast expectation
- RSEC cluster labels used as Slingshot input are derived from the same 50-factor ZINB-WaVE space, not from a different embedding
- ZINB-WaVE factors are computed directly on raw counts with no preliminary normalization step
- The environment satisfies zinbwave's stated requirement of R (>=3.4) and Bioconductor (>=3.6)

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant; if unwanted technical variation is unaccounted for, the inferred low-rank matrix W will capture confounding effects
- ZINB-WaVE requires numerically optimizing a non-convex likelihood function, incurring a computational cost relative to PCA
- Unsupervised trajectory inference is sensitive to the clustering labels it is given, so errors in RSEC clustering propagate to the inferred lineages

## Evidence

- [abstract] accounting for zero inflation (dropouts), over-dispersion, and the count nature of the data: "low-dimensional representations of the data that account for zero inflation (dropouts), over-dispersion, and the count nature of the data"
- [readme] lineage inference with slingshot after clustering: [evidence span withheld — anchor status: not_found]
- [methods] zinbwave environment requirements: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [discussion] distances from ZINB-WaVE space for downstream methods: "derive such distances from the low-dimensional space defined by the factors inferred by ZINB-WaVE"
- [readme] GEO accession for the olfactory epithelium dataset: [evidence span withheld — anchor status: not_found]
