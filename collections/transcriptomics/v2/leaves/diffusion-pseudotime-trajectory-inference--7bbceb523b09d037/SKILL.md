---
name: diffusion-pseudotime-trajectory-inference
description: Use when a preprocessed .h5ad single-cell object (e.
when_to_use_negative:
- The data has no defensible root population or ordered biological process (e.g.,
  discrete unrelated samples) — DPT requires a biologically meaningful starting state
- The feature space still contains many noisy or irrelevant features and feature selection
  has not yet been performed — the article shows similarity-based feature selection
  (DELVE, Laplacian score, Hotspot) should be applied first, since noisy features
  can outweigh informative ones on the cell-similarity graph
- A branched trajectory topology with explicit branch assignments is required rather
  than a continuous ordering — use Slingshot, which the delve_benchmark pipeline provides
  for trajectory inference
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Scanpy
  role: Provides the diffusion pseudotime implementation (sc.tl.dpt) and diffusion
    map embedding (20 components, k=10 kNN graph) used to order cells along differentiation
    and cell-cycle trajectories
- name: DELVE
  role: Upstream unsupervised feature selection whose selected dynamic modules define
    the feature space on which the pseudotime trajectory is rebuilt and evaluated
  repo: https://github.com/jranek/delve
- name: delve_benchmark
  role: Supplies the fs class whose evaluate_select() method ties feature selection
    to trajectory-preservation evaluation and the alternative Slingshot trajectory
    inference in R
  repo: https://github.com/jranek/delve_benchmark
- name: statsmodels
  role: Fits negative binomial generalized additive models regressing each selected
    gene along estimated pseudotime to call trajectory-associated genes at BH q <
    0.05
- name: scikit-learn
  role: Builds the k=10 nearest-neighbor affinity graph underlying the diffusion map
    embedding
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:9f8c38ae000b0454dcd98d5081eef02d200a47a376c5c3074a7bac5c313c6ff3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - EnrichR
  - Hotspot
  - Python
  - ScVelo
  - Scanpy
  - Slingshot
  - UMAP (uniform manifold approximation and projection)
  - anndata
  - delve_benchmark
  - gseapy
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.782608695652174
n_claims: 23
---

# diffusion-pseudotime-trajectory-inference

## Summary

Orders cells along a differentiation or cell-cycle trajectory using Scanpy diffusion pseudotime (DPT) on feature-selected single-cell data (20 diffusion components, k=10 kNN graph), and uses the resulting pseudotime to compare how well competing feature sets (DELVE, HVG, RNA velocity) preserve the underlying biology.

## When to use

Use when a preprocessed .h5ad single-cell object (e.g., from Zenodo record 10534873) has undergone feature selection and you need to infer cell ordering along a lineage — such as hESC-to-definitive-endoderm differentiation or proliferative vs. arrested cell-cycle progression — and score feature sets by trajectory fidelity.

## When NOT to use

- The data has no defensible root population or ordered biological process (e.g., discrete unrelated samples) — DPT requires a biologically meaningful starting state
- The feature space still contains many noisy or irrelevant features and feature selection has not yet been performed — the article shows similarity-based feature selection (DELVE, Laplacian score, Hotspot) should be applied first, since noisy features can outweigh informative ones on the cell-similarity graph
- A branched trajectory topology with explicit branch assignments is required rather than a continuous ordering — use Slingshot, which the delve_benchmark pipeline provides for trajectory inference

## Inputs

- Preprocessed single-cell .h5ad object (e.g., 5397 cells x 2000 genes CPM-normalized, log+1, HVG-filtered hESC-to-DE differentiation data, or RPE/PDAC 4i protein imaging matrices) from Zenodo record 10534873
- A ranked feature subset (top p=500 genes from DELVE, HVG, or ScVelo RNA velocity; top p=30 imaging features) on which to rebuild the cell-cell graph
- Root-cell annotation: day-0 time point, youngest cell-cycle age, or pRB-high/pRB-low lineage gating
- Ground truth trajectory annotations (day/age in hours, cell cycle phase) for correlation-based validation

## Outputs

- Per-cell diffusion pseudotime values along the inferred trajectory
- Kendall rank correlation of estimated pseudotime vs. ground truth age (e.g., proliferation 0.656 / arrest 0.405 on RPE 4i)
- Genes significantly varying along pseudotime (negative binomial GAM, BH q < 0.05) and their pseudotime traces (e.g., SOX2, POU5F1, EOMES, GSC, SOX17, FOXA2)
- Comparison table of trajectory-quality metrics (pseudotime correlation, lineage-specific TF counts) across feature selection methods

## How to apply

Build a k=10 nearest-neighbor affinity graph on the selected feature matrix, embed cells with 20 diffusion map components via Scanpy, and root the diffusion pseudotime at biologically defined start cells (day-0 hESC cells for definitive-endoderm differentiation, youngest-age cells for RPE 4i, or cells with lowest pRB for arrest lineages). For the 4i cell-cycle analyses, separate proliferative (high pRB) from arrested (low pRB) lineages and compute pseudotime per lineage. Evaluate the trajectory by computing the Kendall rank correlation of estimated pseudotime against ground truth age/phase annotations, and in the transcriptomic case regress each of the p=500 selected genes along pseudotime with a negative binomial GAM (statsmodels), calling genes significant at Benjamini-Hochberg q < 0.05. Feature selection must precede trajectory inference because noisy features (dropout, irrelevant imaging measurements, cell-cycle confounders) can distort or mask the intended trajectory; the article reports that performing feature selection with DELVE prior to trajectory inference improved identification and resolution of gene programs.

## Related tools

- **Scanpy** (Provides the diffusion pseudotime implementation (sc.tl.dpt) and diffusion map embedding (20 components, k=10 kNN graph) used to order cells along differentiation and cell-cycle trajectories)
- **DELVE** (Upstream unsupervised feature selection whose selected dynamic modules define the feature space on which the pseudotime trajectory is rebuilt and evaluated) — https://github.com/jranek/delve
- **delve_benchmark** (Supplies the fs class whose evaluate_select() method ties feature selection to trajectory-preservation evaluation and the alternative Slingshot trajectory inference in R) — https://github.com/jranek/delve_benchmark
- **statsmodels** (Fits negative binomial generalized additive models regressing each selected gene along estimated pseudotime to call trajectory-associated genes at BH q < 0.05)
- **scikit-learn** (Builds the k=10 nearest-neighbor affinity graph underlying the diffusion map embedding)

## Examples

```
sc.pp.neighbors(adata, n_neighbors=10); sc.tl.diffmap(adata, n_comps=20); adata.uns['iroot'] = np.flatnonzero(adata.obs['day'] == 0)[0]; sc.tl.dpt(adata)
```

## Evaluation signals

- Kendall rank correlation between estimated pseudotime and ground truth age should reproduce the reported ranges (RPE 4i proliferation 0.656, arrest 0.405)
- The diffusion embedding must use the paper's parameters: 20 diffusion map components and a k=10 kNN graph, with the root set to day-0 (or youngest-age) cells
- Pseudotime traces of known regulators should show expected ordering: SOX2 and POU5F1 decline while EOMES, GSC, SOX17, and FOXA2 rise along hESC-to-definitive-endoderm differentiation (Fig. 6h)
- Genes regressed along pseudotime should be called significant only at Benjamini-Hochberg q < 0.05 in the negative binomial GAM
- The DELVE-selected feature set should yield higher trajectory fidelity (e.g., lineage-specific TF counts ~30% above HVG and RNA velocity in Fig. 6g) than alternative feature selection methods

## Limitations

- Feature selection can greatly bias the interpretation of the underlying cellular trajectory, so careful consideration should be made when performing feature selection for trajectory analysis
- All dynamic modules were leveraged equally for constructing the approximate cellular trajectory; excluding unwanted dynamic modules (e.g., confounding cell-cycle programs) is possible but was not implemented
- Diffusion pseudotime provides a continuous ordering but not explicit branch structure; branched topologies require Slingshot in the benchmarking pipeline

## Evidence

- [discussion] performing feature selection with DELVE prior to performing trajectory inference improved the identification and resolution of gene programs: "performing feature selection with DELVE prior to performing trajectory inference improved the identification and resolution of gene programs"
- [methods] Infer the DE differentiation trajectory using diffusion pseudotime (Scanpy dpt, 20 diffusion components, k=10) rooted in day-0 cells for each feature set: [evidence span withheld — anchor status: not_found]
- [methods] Infer trajectories with the diffusion pseudotime algorithm in Scanpy (20 diffusion map components, k=10 kNN graph, root cells with youngest age): [evidence span withheld — anchor status: not_found]
- [readme] Performs feature selection according to a feature selection strategy, and then evaluates method performance according to the evaluation criteria or task of interest: [evidence span withheld — anchor status: external_doc]
