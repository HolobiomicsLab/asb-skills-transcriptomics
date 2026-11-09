---
name: integrated-gradient-gene-attribution
description: Use when a trained STEM model and spatial trajectory (e.g., the 11-segment
  E1z2 spinal-cord pseudo-spatial coordinates) are available and you need to identify
  which genes determine individual cells' spatial locations, beyond expression-based
  DEGs.
when_to_use_negative:
- The STEM model has not been trained or is still in training mode — attributions
  require the converged checkpoint switched to evaluation via modeleval.
- The goal is differential expression along a trajectory rather than identifying genes
  driving spatial localization — expression-based DEGs (218 here) are the comparison
  set, not the attribution product.
- The dataset is not the E1z2 SeqFISH/spatial-trajectory setting and no ground-truth
  pseudo time exists for validating SDG expression trends.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: STEM
  role: Trained transfer-learning model whose spatial adjacency-embedding-gene path
    is the target of the integrated gradient attribution
  repo: https://github.com/WhirlFirst/STEM
- name: PyTorch
  role: Framework used to compute integrated gradient attributions through the STEM
    network
- name: scSTEM
  role: PyPI distribution of STEM that provides the model classes (STEM.model) used
    in the attribution workflow
  repo: https://github.com/WhirlFirst/STEM
- name: Scanpy
  role: Preprocessing (normalize_total, log1p, QC metrics) of the SC/ST count matrices
    fed to the model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:263b85173b2a34600555235cb3a6cc3d95abc4890e75f96831ea040d67f65b86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - PyTorch
  - STEM
  - Scanpy
  - scSTEM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# integrated-gradient-gene-attribution

## Summary

Runs integrated gradient attribution through the trained STEM model's spatial adjacency-embedding-gene path (PyTorch) to rank genes by their contribution to each cell's spatial placement along a trajectory, calling spatially dominant genes (SDGs) at FDR < 0.05 via Benjamini-Hochberg correction.

## When to use

Use when a trained STEM model and spatial trajectory (e.g., the 11-segment E1z2 spinal-cord pseudo-spatial coordinates) are available and you need to identify which genes determine individual cells' spatial locations, beyond expression-based DEGs.

## When NOT to use

- The STEM model has not been trained or is still in training mode — attributions require the converged checkpoint switched to evaluation via modeleval.
- The goal is differential expression along a trajectory rather than identifying genes driving spatial localization — expression-based DEGs (218 here) are the comparison set, not the attribution product.
- The dataset is not the E1z2 SeqFISH/spatial-trajectory setting and no ground-truth pseudo time exists for validating SDG expression trends.

## Inputs

- Trained STEM model checkpoint (SOmodel, n_genes=351) from Figshare DOI 10.6084/m9.figshare.24452812
- Normalized E1z2 simulated ST counts and raw SeqFISH SC count matrices (351 genes, CSV with cells/spots x genes)
- Single-cell spatial coordinates adata.obsm['spatialDA'] produced by all_coord from the netsc2st SC-ST mapping matrix
- Ground-truth pseudo time labels for the E1z2 trajectory (for validation correlation)

## Outputs

- attribution_matrix.npy — M x H matrix of cells x genes with integrated-gradient attribution magnitudes
- sdg_deg_gene_lists.csv — SDG and DEG gene lists (272 SDGs vs 218 DEGs among 351 genes at FDR < 0.05)
- Pearson correlation of SDG expression trends between reconstructed and ground-truth pseudo time (>0.99)

## How to apply

Load the trained STEM checkpoint (Figshare DOI 10.6084/m9.figshare.24452812) with the normalized E1z2 simulated ST and raw SeqFISH SC matrices (351 genes), and switch the model to evaluation mode with modeleval before attributions. Order cells along the 11-segment spinal-cord trajectory defined by the reconstructed pseudo-spatial coordinates obtained via all_coord from the netsc2st SC-ST mapping matrix. Run integrated gradient attribution with PyTorch along the spatial adjacency-embedding-gene path of the STEM network, accumulating an M x H (cells x genes) attribution matrix. Rank genes by attribution magnitude, test significance of each gene's attribution across cells, and apply Benjamini-Hochberg FDR correction at FDR < 0.05 to call SDGs; call expression-based DEGs along the trajectory in parallel as the comparison set. The rationale is interpretability: attributions reveal the genes that determine each cell's spatial location, which DEGs alone miss.

## Related tools

- **STEM** (Trained transfer-learning model whose spatial adjacency-embedding-gene path is the target of the integrated gradient attribution) — https://github.com/WhirlFirst/STEM
- **PyTorch** (Framework used to compute integrated gradient attributions through the STEM network)
- **scSTEM** (PyPI distribution of STEM that provides the model classes (STEM.model) used in the attribution workflow) — https://github.com/WhirlFirst/STEM
- **Scanpy** (Preprocessing (normalize_total, log1p, QC metrics) of the SC/ST count matrices fed to the model)

## Examples

```
testmodel.modeleval(); scembedding = testmodel.netE(torch.tensor(scdata.values,dtype=torch.float32).cuda())
```

## Evaluation signals

- The SDG call must reproduce 272 spatially dominant genes among the 351 genes at FDR < 0.05 (versus 218 expression-based DEGs).
- Pearson correlation between reconstructed and ground-truth pseudo time for SDG expression trends must exceed 0.99.
- The attribution output must be an M x H (cells x genes) matrix saved as attribution_matrix.npy.
- The model must be in evaluation mode (modeleval) before attribution runs; embeddings used for the mapping matrix come from netE on normalized/log-scaled inputs.
- SDG and DEG gene lists must be written to sdg_deg_gene_lists.csv for direct comparison.

## Limitations

- SDG attribution was demonstrated on the E1z2 semi-simulated SeqFISH spinal-cord trajectory; STEM's authors state that applying STEM to SC and ST data from different donors in the same disease state remains untested future work.
- Using SDGs of cancer cells or disease-related cell types as insights into potential drug targets is only proposed, not demonstrated in the article.
- Attribution significance depends on the FDR < 0.05 Benjamini-Hochberg threshold and the quality of the reconstructed pseudo-spatial coordinates from all_coord.

## Evidence

- [intro] interpretability motivation: "Lastly, interpretability is critical to uncover the mechanisms governing tissue spatial organization. We need to identify the genes that determine the spatial location of individual cells."
- [discussion] SDG pathfinding: "Upon the model, by following the spatial adjacency-embedding-gene pathfinding, we highlight the spatially dominant genes (SDGs) in the STEM model."
- [readme] model deposit for reproduction: "The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812)."
- [readme] evaluation-mode embeddings and mapping matrix: "testmodel.modeleval()
scembedding = testmodel.netE(torch.tensor(scdata.values,dtype=torch.float32).cuda())"
- [methods] pseudo-spatial coordinate reconstruction: "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
