---
name: cell-cycle-gene-phase-annotation
description: Use when single cells have been reordered along a differentiation trajectory
  (pseudo-temporal position) and you must cluster gene expression dynamics into temporal
  waves and check whether cell-cycle gene sets (mitosis vs DNA synthesis) fall into
  distinct, out-of-phase clusters.
when_to_use_negative:
- Cells have not been reordered along a pseudo-temporal trajectory — GAM fitting requires
  pseudo-temporal position along a path (e.g. from Mpath cell ordering)
- Genes are not differentially regulated along the target path — use only the path-specific
  DEG lists (Supplementary Data 2/3), not the full transcriptome
- The developmental system starts from multiple independent progenitor cell types
  or lacks FACS-sorted terminal subsets — Mpath explicitly does not yet support these
  cases
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Mpath
  role: R package used to fit GAMs of expression as a smooth function of pseudo-temporal
    position and cluster fitted gene expression dynamics into temporal wave clusters
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: Runtime environment for Mpath's R implementation of GAM fitting and clustering
- name: Ingenuity Pathway Analysis (IPA)
  role: Pathway annotation of wave clusters to identify mitosis and DNA-synthesis
    gene sets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:a375f6ce9e0cfe25d889697c8d924152e9916d66c2f58f1d61a0d9c12accc750
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Ingenuity Pathway Analysis (IPA)
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cell-cycle-gene-phase-annotation

## Summary

Annotates temporal wave clusters of differentially regulated genes along pseudo-temporally ordered dendritic cell (DC) differentiation paths, specifically identifying mitosis and DNA-synthesis gene sets and verifying they occupy nearly out-of-phase waves. Grounded in Mpath's GAM-based clustering on the cDC2 and cDC1 paths of mouse DC progenitor development.

## When to use

Use when single cells have been reordered along a differentiation trajectory (pseudo-temporal position) and you must cluster gene expression dynamics into temporal waves and check whether cell-cycle gene sets (mitosis vs DNA synthesis) fall into distinct, out-of-phase clusters.

## When NOT to use

- Cells have not been reordered along a pseudo-temporal trajectory — GAM fitting requires pseudo-temporal position along a path (e.g. from Mpath cell ordering)
- Genes are not differentially regulated along the target path — use only the path-specific DEG lists (Supplementary Data 2/3), not the full transcriptome
- The developmental system starts from multiple independent progenitor cell types or lacks FACS-sorted terminal subsets — Mpath explicitly does not yet support these cases

## Inputs

- Reordered (pseudo-temporally ordered) single cells on the cDC2 path
- Reordered (pseudo-temporally ordered) single cells on the cDC1 path
- Path-specific differentially expressed gene lists: Supplementary Data 2 (early CDP to late cDC2-committed preDC) and Supplementary Data 3 (early CDP to late cDC1-committed preDC)
- Gene expression matrix (single-cell RNA-sequencing) for the ordered cells

## Outputs

- temporal_wave_clusters.csv with gene-to-wave cluster assignments and per-path cluster summaries
- 7 distinct temporal wave clusters on the cDC2 path and 6 on the cDC1 path
- Pathway annotations (IPA terms) for each wave cluster, including mitosis and DNA-synthesis gene sets

## How to apply

Load the reordered cells on each developmental path (cDC2 and cDC1) together with the path-specific differentially expressed gene lists (Supplementary Data 2 for early CDP to late cDC2-committed preDC; Supplementary Data 3 for early CDP to late cDC1-committed preDC). For each differentially regulated gene, fit a generalized additive model (GAM) of expression as a smooth function of pseudo-temporal position along the reordered path using Mpath's R implementation, then cluster the fitted expression dynamics into temporal wave clusters. Verify the clustering recovers exactly 7 wave clusters on the cDC2 path and 6 on the cDC1 path. Annotate each wave cluster with pathway terms via Ingenuity Pathway Analysis (IPA) to identify mitosis and DNA-synthesis gene sets, and confirm they are assigned to temporally out-of-phase (nearly opposite) waves, reflecting the temporal coupling between cell cycle and cDC differentiation. Write gene-to-wave cluster assignments and per-path cluster summaries to temporal_wave_clusters.csv.

## Related tools

- **Mpath** (R package used to fit GAMs of expression as a smooth function of pseudo-temporal position and cluster fitted gene expression dynamics into temporal wave clusters) — https://github.com/JinmiaoChenLab/Mpath
- **R** (Runtime environment for Mpath's R implementation of GAM fitting and clustering)
- **Ingenuity Pathway Analysis (IPA)** (Pathway annotation of wave clusters to identify mitosis and DNA-synthesis gene sets)

## Evaluation signals

- GAM-based clustering on the reordered cDC2 path recovers exactly 7 distinct temporal wave clusters
- GAM-based clustering on the reordered cDC1 path recovers exactly 6 distinct temporal wave clusters
- Mitosis genes and DNA-synthesis genes are assigned to two nearly out-of-phase waves (temporal coupling between cell cycle and cDC differentiation)
- The final wave on both paths is enriched for MHC class II antigen processing and presentation
- temporal_wave_clusters.csv contains one wave-cluster assignment per input differentially regulated gene for both paths

## Limitations

- Mpath requires signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction, which the downstream ordering and wave analysis inherits
- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension)
- Utility demonstrated only on mouse bone marrow-derived DC progenitors and primary human myoblasts; wave-cluster counts (7 and 6) are specific to the cDC2/cDC1 paths

## Evidence

- [abstract] Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation: "Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation."
- [discussion] Cell ordering discovered phase-dependent waves: "its ordering of single cells discovered phase-dependent waves of gene regulation on the scale of DC development and subset differentiation continuum."
- [discussion] Switch from proliferation to differentiation: "it placed individual cells in order along the developmental trajectories, revealing a switch from proliferation to differentiation during DC maturation."
- [supplementary] DEG identification method: "The DEGs were identified using ANOVA followed by post-hoc Tukey's test. P values were adjusted for multi-test correction using Benjamini-Hochberg."
- [supplementary] IPA pathway annotation: "as defined by Ingenuity Pathway Analysis (IPA)"
