---
name: spatial-trajectory-pseudotime-trend-correlation
description: Use when single-cell spatial coordinates have been reconstructed from
  an SC-ST mapping (e.g., via all_coord on netsc2st) and you must verify the ordering
  is biologically faithful against ground-truth pseudo time before trusting downstream
  SDG/DEG calls.
when_to_use_negative:
- SC data has no reconstructed spatial coordinates or SC-ST mapping (e.g., no adata.obsm['spatialDA']
  from all_coord), so no pseudo time exists to compare against ground truth.
- Evaluating methods that do not claim to preserve single-cell topology — the trend-correlation
  check is specific to validating pseudo-spatial orderings like STEM's.
- Gene lists called from a different tissue or dataset without a defined spatial trajectory
  (the E1z2 benchmark has 11 spinal-cord segments; other data need their own trajectory
  definition).
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STEM
  role: Provides the trained model whose integrated gradient attributions call the
    SDGs whose trajectory trends are correlated, and the netE embeddings/mapping matrices
    that produce the reconstructed pseudo-spatial coordinates
  repo: https://github.com/WhirlFirst/STEM
- name: Scanpy
  role: Hosts the AnnData object carrying adata.obsm['spatialDA'] reconstructed coordinates
    and provides QC metrics for normalized SC/ST count matrices
- name: PyTorch
  role: Runs the STEM model in evaluation mode (modeleval) and computes netE embeddings
    used to build netsc2st for spatial coordinate reconstruction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:5e03cf7af73c7932a2bfe55fe6dadcfff431d88842cfc5d5aa53bfc9e0e3caef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - PyTorch
  - STEM
  - Scanpy
  - scSTEM
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# spatial-trajectory-pseudotime-trend-correlation

## Summary

Validates single-cell spatial reconstruction by ordering cells along a spatial trajectory (e.g., the 11-segment E1z2 spinal-cord trajectory from SeqFISH data) and computing the Pearson correlation of gene expression trends between reconstructed pseudo time and ground-truth pseudo time. A correlation >0.99 for SDG expression trends demonstrates that STEM's spatially dominant genes preserve trajectory structure.

## When to use

Use when single-cell spatial coordinates have been reconstructed from an SC-ST mapping (e.g., via all_coord on netsc2st) and you must verify the ordering is biologically faithful against ground-truth pseudo time before trusting downstream SDG/DEG calls.

## When NOT to use

- SC data has no reconstructed spatial coordinates or SC-ST mapping (e.g., no adata.obsm['spatialDA'] from all_coord), so no pseudo time exists to compare against ground truth.
- Evaluating methods that do not claim to preserve single-cell topology — the trend-correlation check is specific to validating pseudo-spatial orderings like STEM's.
- Gene lists called from a different tissue or dataset without a defined spatial trajectory (the E1z2 benchmark has 11 spinal-cord segments; other data need their own trajectory definition).

## Inputs

- AnnData object with adata.obsm['spatialDA'] reconstructed coordinates (351-gene E1z2 SeqFISH SC data)
- SC-ST mapping matrix netsc2st (softmax of scembedding.mm(stembedding.t()))
- ST spot spatial coordinates (simu_st_metadata.csv / spcoor)
- SDG gene list (272 genes at FDR < 0.05) and/or DEG gene list (218 genes) as CSV
- Ground-truth pseudo time values along the spatial trajectory

## Outputs

- Pearson correlation coefficient between reconstructed and ground-truth pseudo-time expression trends for SDGs (expected >0.99)
- Cell ordering along the 11-segment spatial trajectory
- Per-gene trend vectors across ordered positions for SDG and DEG sets

## How to apply

Order cells along the spatial trajectory defined by the reconstructed pseudo-spatial coordinates stored in adata.obsm['spatialDA'], which all_coord derives from the SC-ST mapping matrix netsc2st and the ST spot coordinates (spcoor). Sort cells along the trajectory segments (11 segments for the E1z2 spinal-cord simulation) and extract expression values for the target gene set — here the 272 spatial dominant genes (SDGs) called at FDR < 0.05 by integrated gradient attribution, or the 218 expression-based DEGs as a comparison. Build per-gene trend vectors across ordered positions for both the reconstructed pseudo time and the ground-truth pseudo time, then compute the Pearson correlation between the paired trend vectors with scipy.stats.pearsonr. Rationale: STEM claims to preserve the original topology of all single cells, so expression trends along the reconstructed trajectory should match ground truth almost exactly (Pearson >0.99 on E1z2), providing a quantitative reproduction check rather than a visual comparison.

## Related tools

- **STEM** (Provides the trained model whose integrated gradient attributions call the SDGs whose trajectory trends are correlated, and the netE embeddings/mapping matrices that produce the reconstructed pseudo-spatial coordinates) — https://github.com/WhirlFirst/STEM
- **Scanpy** (Hosts the AnnData object carrying adata.obsm['spatialDA'] reconstructed coordinates and provides QC metrics for normalized SC/ST count matrices)
- **PyTorch** (Runs the STEM model in evaluation mode (modeleval) and computes netE embeddings used to build netsc2st for spatial coordinate reconstruction)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(sdg_trend_reconstructed, sdg_trend_groundtruth); assert r > 0.99
```

## Evaluation signals

- Pearson correlation between reconstructed and ground-truth pseudo-time SDG expression trends exceeds 0.99 on the E1z2 trajectory.
- The gene set being validated contains 272 SDGs among 351 genes at Benjamini-Hochberg FDR < 0.05 (versus 218 expression-based DEGs).
- Cells order coherently along the 11-segment spinal-cord spatial trajectory defined by spatialDA coordinates.
- Reconstruction used the SC-ST mapping matrix netsc2st with row-wise softmax normalization (each cell's probabilities over spots sum to 1).
- Input matrices were normalized with sc.pp.normalize_total and sc.pp.log1p before embedding, matching the published preprocessing.

## Limitations

- STEM applying SC and ST data from different donors in the same disease state remains untested, so trajectory correlation results may not transfer across donors.
- The >0.99 threshold was established on the simulated 351-gene E1z2 SeqFISH benchmark; performance on real, larger-gene data is not guaranteed by this validation alone.
- The skill validates trajectory ordering, not the correctness of individual SDG calls — attribution significance and trend correlation should be checked together.

## Evidence

- [results] SDG trend correlation on E1z2: [evidence span withheld — anchor status: not_found]
- [discussion] SDG pathfinding motivation: "Upon the model, by following the spatial adjacency-embedding-gene pathfinding, we highlight the spatially dominant genes (SDGs) in the STEM model."
- [readme] Reconstructed coordinate computation: "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
- [readme] Topology preservation claim: "Compared with other methods, STEM is the only method that preserves the original topology structure of all single cells."
