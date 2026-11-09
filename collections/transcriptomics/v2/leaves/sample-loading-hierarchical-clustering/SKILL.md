---
name: sample-loading-hierarchical-clustering
description: Use when you have a completed Tensor-cell2cell decomposition (e.g., rank-6
  on a 4D communication tensor) and want to check whether sample loadings across factors
  resolve clusters concordant with phenotype labels such as condition or severity.
when_to_use_negative:
- The decomposition has not yet been run or factor loadings have not been extracted
  — clustering raw communication scores per context instead of the Tensor-cell2cell
  loadings loses the shared factor structure.
- You need statistical evidence of condition differences — use per-factor t-tests
  with Bonferroni correction (cell2cell.stats) instead, since clustering alone does
  not test significance.
- Samples have missing cell types or LR pairs, since only cell types shared across
  all samples can be included in the tensor being clustered.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2258
tools:
- name: cell2cell
  role: Provides cell2cell.clustering, a module of scipy-based functions for performing
    hierarchical clustering on Tensor-cell2cell factor loadings, plus cell2cell.tensor
    for building the tensor and cell2cell.stats for downstream factor significance
    tests
  repo: https://github.com/earmingol/cell2cell
- name: Tensor-cell2cell
  role: Runs the TCA-based tensor decomposition (e.g., rank 6, init='svd') whose sample/context
    loadings form the clustering input
  repo: https://github.com/earmingol/cell2cell
- name: Anaconda
  role: Environment manager used to install cell2cell via pip in a dedicated conda
    environment
  repo: https://github.com/earmingol/cell2cell
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:9de3678097a1fd9e547f15e5c6d7ca2dc87ccd6a13145421f76342a50c1f5f23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - Anaconda
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# sample-loading-hierarchical-clustering

## Summary

Hierarchical clustering on the sample-by-factor loading matrix produced by Tensor-cell2cell tensor decomposition, used to test whether samples group by condition (e.g., ASD patients vs controls) based on shared communication factors. Implemented with the scipy-based functions in the cell2cell.clustering module.

## When to use

Use when you have a completed Tensor-cell2cell decomposition (e.g., rank-6 on a 4D communication tensor) and want to check whether sample loadings across factors resolve clusters concordant with phenotype labels such as condition or severity.

## When NOT to use

- The decomposition has not yet been run or factor loadings have not been extracted — clustering raw communication scores per context instead of the Tensor-cell2cell loadings loses the shared factor structure.
- You need statistical evidence of condition differences — use per-factor t-tests with Bonferroni correction (cell2cell.stats) instead, since clustering alone does not test significance.
- Samples have missing cell types or LR pairs, since only cell types shared across all samples can be included in the tensor being clustered.

## Inputs

- Sample-by-factor context loading matrix from Tensor-cell2cell decomposition (e.g., 23 samples x 6 factors)
- Condition/phenotype labels per sample (e.g., 13 ASD vs 10 control samples from PRJNA434002)
- Fitted Tensor-cell2cell tensor object (cell2cell.tensor)

## Outputs

- Hierarchical clustering dendrogram of samples based on factor loadings
- Cluster assignments of samples (e.g., 4 main clusters in the ASD case study)
- Assessment of cluster-concordance with condition labels

## How to apply

After running Tensor-cell2cell (e.g., rank 6 with init='svd', falling back to init='random' on memory errors), extract the context (sample) loadings for all factors into a samples-by-factors matrix — e.g., 23 samples (13 ASD, 10 controls) x 6 factors in the ASD prefrontal cortex case study. Apply hierarchical clustering using the scipy-based functions in cell2cell.clustering on this loading matrix to group samples by similarity of their communication-factor profiles. Assess whether the number of emergent clusters matches the expected condition structure — in the ASD case, four main clusters aligned with condition (controls vs ASD). Interpret cluster concordance together with per-factor significance tests (t-tests with Bonferroni correction via cell2cell.stats) rather than as standalone proof.

## Related tools

- **cell2cell** (Provides cell2cell.clustering, a module of scipy-based functions for performing hierarchical clustering on Tensor-cell2cell factor loadings, plus cell2cell.tensor for building the tensor and cell2cell.stats for downstream factor significance tests) — https://github.com/earmingol/cell2cell
- **Tensor-cell2cell** (Runs the TCA-based tensor decomposition (e.g., rank 6, init='svd') whose sample/context loadings form the clustering input) — https://github.com/earmingol/cell2cell
- **Anaconda** (Environment manager used to install cell2cell via pip in a dedicated conda environment) — https://github.com/earmingol/cell2cell

## Examples

```
conda create -n cell2cell -y python=3.7 jupyter && conda activate cell2cell && pip install cell2cell
```

## Evaluation signals

- Hierarchical clustering of the 23-sample x 6-factor loading matrix resolves 4 main clusters concordant with condition labels (controls vs ASD).
- Clusters group 13 ASD samples and 10 controls consistently with the factors (3 and 4) found significant after Bonferroni-corrected t-tests.
- The clustering uses loadings from a completed decomposition (rank 6, init='svd' or 'random' fallback), not raw per-context LR scores.
- Output dendrogram and cluster assignments are saved alongside factor loadings and significance results for reproducibility.
- Software versions match those in the Code Ocean capsule (https://doi.org/10.24433/CO.0051950.v2) for reproducible runs.

## Limitations

- The underlying tensor contains only cell types shared across all samples, so rare cell types are excluded and the loading structure being clustered may miss context-specific populations.
- The implemented tensor decomposition algorithm is not optimized for missing values in the communication tensor, so incomplete tensors degrade the loadings being clustered.
- Clustering result is descriptive; it must be paired with statistical comparison of factors (Bonferroni-corrected t-tests) to claim condition discrimination.

## Evidence

- [discussion] In our ASD case-study (Fig. 5), such analyses included GSEA, clustering, visualization and statistical comparison of factors: "In our ASD case-study (Fig. 5), such analyses included GSEA, clustering, visualization and statistical comparison of factors"
- [methods] cell2cell.clustering : Includes multiple scipy-based functions for performing clustering methods.: "cell2cell.clustering : Includes multiple scipy-based functions for performing clustering methods."
- [discussion] we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [readme] Step 3: Install cell2cell pip install cell2cell: [evidence span withheld — anchor status: not_found]
