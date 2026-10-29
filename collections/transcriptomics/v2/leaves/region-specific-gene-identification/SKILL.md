---
name: region-specific-gene-identification
description: Use when a reconstructed single-cell spatial landscape is available and
  the goal is to find genes determining spatial location or restricted to specific
  regions (e.g., cortical layers L1-L6, hepatic lobule zones) rather than merely mapping
  cells.
when_to_use_negative:
- Before a spatially aware embedding exists — STEM's SDG identification requires the
  trained model's adjacency-embedding-gene pathfinding, so plain differential expression
  on non-spatial SC data is the wrong substitute
- When the goal is deconvolving spot composition into fixed cell-type fractions rather
  than linking SC and ST at single-cell resolution — the article notes deconvolution
  cannot link SC and ST at the single-cell level
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STEM
  role: Trained transfer-learning model whose spatial adjacency-embedding-gene pathfinding
    yields the spatially dominant genes; installed via pip install scSTEM
  repo: https://github.com/WhirlFirst/STEM
- name: scanpy
  role: Preprocessing (sc.pp.normalize_total, sc.pp.log1p, sc.pp.calculate_qc_metrics
    for dropout-rate estimation) feeding the gene-identification step
- name: PyTorch
  role: Backend for the SOmodel, netE embeddings, and softmax mapping matrices used
    in the pathfinding
- name: seaborn
  role: Visualization of region-specific gene expression patterns across reconstructed
    spatial coordinates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:9cad7ceea0f2f8d1a63998025b08142ce2b25b23868b3e110c14e993ce90161f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - PyTorch
  - STEM
  - scSTEM
  - scanpy
  - seaborn
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# region-specific-gene-identification

## Summary

Identify genes whose expression is spatially restricted to specific tissue regions by following STEM's spatial adjacency-embedding-gene pathfinding to extract spatially dominant genes (SDGs), and validate region-specificity against known layer/region marker genes.

## When to use

Use when a reconstructed single-cell spatial landscape is available and the goal is to find genes determining spatial location or restricted to specific regions (e.g., cortical layers L1-L6, hepatic lobule zones) rather than merely mapping cells.

## When NOT to use

- Before a spatially aware embedding exists — STEM's SDG identification requires the trained model's adjacency-embedding-gene pathfinding, so plain differential expression on non-spatial SC data is the wrong substitute
- When the goal is deconvolving spot composition into fixed cell-type fractions rather than linking SC and ST at single-cell resolution — the article notes deconvolution cannot link SC and ST at the single-cell level

## Inputs

- Trained STEM model (netE encoder) with SC and ST count matrices normalized via sc.pp.normalize_total and log1p-transformed
- SC-ST mapping matrix netsc2st and ST-SC mapping matrix netst2sc (softmax over embedding dot products)
- ST spatial coordinates and layer/region annotations (e.g., MERFISH MTG H18 L1-L6 layer boundaries)

## Outputs

- List of spatially dominant genes (SDGs) that determine single-cell spatial location
- Region-specific SC-unique gene list (e.g., CXCL14, CUX2, RORB, NFIA, APOD for MTG layers)
- Cell-type-specific gene expression variation along a spatial axis

## How to apply

After training the STEM SOmodel (e.g., alpha=0.8 MMD weight, lr=0.002, sigma set to half the ST spot adjacent distance, dp computed as 1 minus the ratio of median n_genes_by_counts between SC and ST), obtain SC and ST embeddings via netE and build the softmax mapping matrices netst2sc and netsc2st. Follow the spatial adjacency-embedding-gene pathfinding through the trained model to extract spatially dominant genes (SDGs) — genes that determine the spatial location of individual cells. For a reconstruction such as the human MTG (donor H18 SMART-seq SC + MERFISH ST), compile the region-specific SC-unique gene list and check that known layer markers are recovered: CXCL14, CUX2, RORB, NFIA and APOD. Interpretability is the rationale: SDGs uncover the mechanisms governing tissue spatial organization and, for disease cell types, could point to potential drug targets.

## Related tools

- **STEM** (Trained transfer-learning model whose spatial adjacency-embedding-gene pathfinding yields the spatially dominant genes; installed via pip install scSTEM) — https://github.com/WhirlFirst/STEM
- **scanpy** (Preprocessing (sc.pp.normalize_total, sc.pp.log1p, sc.pp.calculate_qc_metrics for dropout-rate estimation) feeding the gene-identification step)
- **PyTorch** (Backend for the SOmodel, netE embeddings, and softmax mapping matrices used in the pathfinding)
- **seaborn** (Visualization of region-specific gene expression patterns across reconstructed spatial coordinates)

## Examples

```
from STEM.model import SOmodel; from STEM.utils import all_coord, seed_all; testmodel.modeleval(); scemb = testmodel.netE(torch.tensor(scdata.values).float().cuda()); netsc2st = F.softmax(scemb.mm(stemb.t()), dim=1).detach().cpu().numpy()
```

## Evaluation signals

- Known region/layer marker genes are recovered: CXCL14, CUX2, RORB, NFIA and APOD must appear as region-specific SC-unique genes in the MTG reconstruction
- Identified SDGs should show spatially restricted expression when projected onto reconstructed single-cell coordinates (adata.obsm['spatialDA'])
- The upstream model must have converged: the train_wholedata loss curve should decrease over the 400 epochs, and reconstructed cortical-depth distributions of L1-L6 dissection groups must align with true layer boundaries
- SDGs should reflect gene-spatial relations learned from ST data — their spatial patterns should match the ST spots the cells were mapped to via netsc2st

## Limitations

- Using SDGs of cancer cells or disease-related cell types as insights into potential drug targets is only proposed, not demonstrated
- SDG identification depends on embedding quality; applying STEM to SC and ST data from different donors in the same disease state remains untested future work

## Evidence

- [discussion] Upon the model, by following the spatial adjacency-embedding-gene pathfinding, we highlight the spatially dominant genes (SDGs) in the STEM model.: "Upon the model, by following the spatial adjacency-embedding-gene pathfinding, we highlight the spatially dominant genes (SDGs) in the STEM model."
- [intro] Interpretability is critical to uncover the mechanisms governing tissue spatial organization.: "Lastly, interpretability is critical to uncover the mechanisms governing tissue spatial organization. We need to identify the genes that determine the spatial location of individual cells."
- [readme] adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor): "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
- [discussion] Identifying SDGs of cancer cells or disease-related cell types could also be used to provide insights into potential drug targets.: "Furthermore, identifying SDGs of cancer cells or disease-related cell types could also be used to provide insights into potential drug targets."
