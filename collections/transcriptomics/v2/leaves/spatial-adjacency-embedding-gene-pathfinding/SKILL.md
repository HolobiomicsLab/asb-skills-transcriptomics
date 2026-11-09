---
name: spatial-adjacency-embedding-gene-pathfinding
description: Use when a trained STEM model (e.g. on the E1z2 SeqFISH spinal-cord trajectory
  with 351 genes) and reconstructed pseudo-spatial coordinates are available, and
  the goal is interpretable attribution of which genes drive each cell's spatial placement
  rather than expression-only differential testing.
when_to_use_negative:
- You only need differential expression along a trajectory — expression-based DEGs
  (218 called on E1z2) cover that and do not require attribution through the model.
- No trained STEM model checkpoint or SC-ST mapping (netsc2st / spatialDA coordinates)
  exists for the dataset — the pathfinding requires the trained spatially aware embedding
  network.
- The analysis targets absolute spatial coordinate prediction or SC-ST mapping itself
  rather than post-hoc gene interpretation of an already-trained model.
tools:
- name: STEM
  role: Trained spatially aware embedding model whose spatial adjacency-embedding-gene
    path is traversed for attribution; evaluated via modeleval before attribution
  repo: https://github.com/WhirlFirst/STEM
- name: PyTorch
  role: Backpropagation framework used to run integrated gradient attribution through
    the STEM network and accumulate the M x H attribution matrix
- name: Scanpy
  role: Used for normalize_total/log1p preprocessing of SC and ST counts and for calling
    expression-based DEGs as the comparison set
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:46d5d440ecc42d57747844067492971db1de72c6db27b2717c1872a02eeab2b8
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# spatial-adjacency-embedding-gene-pathfinding

## Summary

Interprets the trained STEM deep transfer-learning model by following its spatial adjacency-embedding-gene path to identify spatially dominant genes (SDGs) — the genes that most contribute to determining each cell's spatial location along a spatial trajectory.

## When to use

Use when a trained STEM model (e.g. on the E1z2 SeqFISH spinal-cord trajectory with 351 genes) and reconstructed pseudo-spatial coordinates are available, and the goal is interpretable attribution of which genes drive each cell's spatial placement rather than expression-only differential testing.

## When NOT to use

- You only need differential expression along a trajectory — expression-based DEGs (218 called on E1z2) cover that and do not require attribution through the model.
- No trained STEM model checkpoint or SC-ST mapping (netsc2st / spatialDA coordinates) exists for the dataset — the pathfinding requires the trained spatially aware embedding network.
- The analysis targets absolute spatial coordinate prediction or SC-ST mapping itself rather than post-hoc gene interpretation of an already-trained model.

## Inputs

- Trained STEM model checkpoint from the Figshare deposit (DOI 10.6084/m9.figshare.24452812)
- Normalized, log1p-scaled simulated ST and raw SeqFISH SC count matrices for E1z2 (351 genes)
- Single-cell pseudo-spatial coordinates adata.obsm['spatialDA'] produced by all_coord from the netsc2st SC-ST mapping matrix
- Reconstructed pseudo time and ground-truth pseudo time along the spinal-cord spatial trajectory (for validation)

## Outputs

- M x H cells-by-genes attribution matrix (attribution_matrix.npy)
- SDG and DEG gene lists (sdg_deg_gene_lists.csv); 272 SDGs among 351 genes at FDR < 0.05 vs 218 expression-based DEGs
- Pearson correlation of SDG expression trends between reconstructed and ground-truth pseudo time

## How to apply

Load the trained STEM checkpoint from the Figshare deposit (DOI 10.6084/m9.figshare.24452812) with the normalized E1z2 simulated ST and raw SeqFISH SC count matrices (351 genes), and place cells on the 11-segment spinal-cord trajectory via the SC-ST mapping coordinates in adata.obsm['spatialDA'] computed with all_coord. Switch the model to evaluation mode (modeleval) and pass each cell's gene expression through netE, then run integrated gradient attribution with PyTorch along the spatial adjacency-embedding-gene path, accumulating an M x H cells-by-genes attribution matrix. Rank genes by attribution magnitude, test the significance of each gene's attribution across cells, and apply Benjamini-Hochberg correction calling genes significant at FDR < 0.05 as spatially dominant genes (SDGs), in parallel with expression-based DEGs along the trajectory as a comparison set. The rationale is that STEM's spatially aware embeddings tie gene contributions to predicted spatial adjacency, so attributions recover genes governing spatial organization even when their expression-fold changes are modest.

## Related tools

- **STEM** (Trained spatially aware embedding model whose spatial adjacency-embedding-gene path is traversed for attribution; evaluated via modeleval before attribution) — https://github.com/WhirlFirst/STEM
- **PyTorch** (Backpropagation framework used to run integrated gradient attribution through the STEM network and accumulate the M x H attribution matrix)
- **Scanpy** (Used for normalize_total/log1p preprocessing of SC and ST counts and for calling expression-based DEGs as the comparison set)

## Examples

```
testmodel.modeleval(); scembedding = testmodel.netE(torch.tensor(scdata.values,dtype=torch.float32).cuda()); # then integrated gradients along the adjacency-embedding-gene path
```

## Evaluation signals

- Reproduce 272 SDGs among the 351 E1z2 genes at Benjamini-Hochberg FDR < 0.05
- SDG set should be smaller/more selective than the 218 expression-based DEGs called along the same trajectory
- Pearson correlation of SDG expression trends between reconstructed pseudo time and ground-truth pseudo time must exceed 0.99
- Attribution matrix must have shape M x H (cells x genes) matching the 351-gene input
- sdg_deg_gene_lists.csv must contain both the SDG and the comparison DEG gene lists

## Limitations

- SDGs of cancer cells or disease-related cell types as potential drug targets were only proposed in the article, not demonstrated.
- Applying STEM to SC and ST data from different donors in the same disease state remains untested future work, limiting generalization of pathfinding results across donors.
- Attribution significance and the >0.99 trend correlation were validated on the simulated E1z2 SeqFISH spinal-cord trajectory; other tissues may require re-validation.

## Evidence

- [discussion] Pathfinding is the article's named interpretability mechanism: "Upon the model, by following the spatial adjacency-embedding-gene pathfinding, we highlight the spatially dominant genes (SDGs) in the STEM model."
- [intro] Motivation: interpretability for spatial location determinants: "Lastly, interpretability is critical to uncover the mechanisms governing tissue spatial organization. We need to identify the genes that determine the spatial location of individual cells."
- [readme] Pseudo-spatial coordinates feed the pathfinding inputs: "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
- [readme] Model switched to evaluation mode before embedding/attribution: "testmodel.modeleval()
scembedding = testmodel.netE(torch.tensor(scdata.values,dtype=torch.float32).cuda())"
