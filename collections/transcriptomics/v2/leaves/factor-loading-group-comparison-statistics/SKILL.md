---
name: factor-loading-group-comparison-statistics
description: Use when a tensor decomposition of cell-cell communication scores yields
  context loadings and you must determine which factors differ significantly between
  phenotypic groups or whether samples group by condition.
when_to_use_negative:
- Loadings come from a decomposition with only one context or one sample per group,
  so group-level t-tests are undefined
- You need to compare communication scores per individual ligand-receptor pair rather
  than factor-level latent loadings across contexts
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: cell2cell
  role: Provides cell2cell.stats for t-tests with Bonferroni multiple-test correction
    and cell2cell.clustering with scipy-based hierarchical clustering functions for
    sample loadings
  repo: https://github.com/earmingol/cell2cell
- name: Tensor-cell2cell
  role: Produces the factor and context loadings that are statistically compared and
    clustered downstream
  repo: https://github.com/earmingol/cell2cell
- name: Anaconda
  role: Environment manager used to install and run the cell2cell suite
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:c199e643ca3f553b864e1fc7c21fb686322586e9ce8d93b3af17ce66ee1c5d9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - Anaconda
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# factor-loading-group-comparison-statistics

## Summary

Statistical and clustering comparison of Tensor-cell2cell context loadings to test whether decomposed communication factors distinguish sample groups (e.g., ASD vs controls) and whether samples cluster by condition, using cell2cell.stats t-tests with Bonferroni correction and scipy-based functions in cell2cell.clustering.

## When to use

Use when a tensor decomposition of cell-cell communication scores yields context loadings and you must determine which factors differ significantly between phenotypic groups or whether samples group by condition.

## When NOT to use

- Loadings come from a decomposition with only one context or one sample per group, so group-level t-tests are undefined
- You need to compare communication scores per individual ligand-receptor pair rather than factor-level latent loadings across contexts

## Inputs

- Context (sample) loadings matrix from a Tensor-cell2cell decomposition (samples x factors, e.g., 23 samples x 6 factors)
- Sample condition/group labels (e.g., 13 ASD vs 10 typically developing controls from PRJNA434002)
- cell2cell Tensor object or exported factor loadings

## Outputs

- Per-factor t-test statistics and p-values comparing groups, with Bonferroni-corrected significance calls (e.g., factors 3 and 4 significant)
- Hierarchical clustering output on the sample-by-factor loading matrix (e.g., 4 clusters aligned with condition)
- Saved factor loadings, significance results, and clustering assignments

## How to apply

After running Tensor-cell2cell (e.g., rank-6 decomposition with init='svd', switching to init='random' on memory errors) on a 4D communication tensor such as the 23-sample ASD prefrontal cortex tensor (13 ASD, 10 controls; 749 LR pairs x 16 sender x 16 receiver cell types), extract the context loadings for all factors. Use cell2cell.stats to perform t-tests comparing loadings between groups (ASD n=13 vs control n=10) per factor, applying Bonferroni multiple-test correction to all factor-level tests. Complement this with hierarchical clustering on the sample-by-factor loading matrix using the scipy-based functions in cell2cell.clustering (e.g., expecting 4 clusters aligned with condition labels). In the ASD case study, factors 3 and 4 showed significantly different loadings between ASD and control samples after Bonferroni correction, and clustering resolved four main clusters concordant with condition.

## Related tools

- **cell2cell** (Provides cell2cell.stats for t-tests with Bonferroni multiple-test correction and cell2cell.clustering with scipy-based hierarchical clustering functions for sample loadings) — https://github.com/earmingol/cell2cell
- **Tensor-cell2cell** (Produces the factor and context loadings that are statistically compared and clustered downstream) — https://github.com/earmingol/cell2cell
- **Anaconda** (Environment manager used to install and run the cell2cell suite)

## Examples

```
import cell2cell; ttests = cell2cell.stats.t_test_loadings(context_loadings, group_labels); cell2cell.clustering.hierarchical_clustering(sample_loadings, n_clusters=4)
```

## Evaluation signals

- Significant factors (e.g., factors 3 and 4) remain significant after Bonferroni correction across all factor-level tests
- Hierarchical clustering of the sample-by-factor loading matrix resolves the expected number of clusters (4) concordant with condition labels (controls vs ASD)
- All 23 samples (13 ASD, 10 controls) appear in the loading matrix with complete loadings for all 6 factors
- Group sizes in the t-tests match the source data (n=13 ASD, n=10 controls)

## Limitations

- Only cell types shared across all samples were included in the ASD case-study tensor, potentially missing rare cell types and biasing downstream factor loadings
- The implemented tensor decomposition algorithm is not optimized for missing values, so factors may not capture communication members absent in some contexts
- Significance testing at the factor level does not provide ground-truth benchmarking; benchmarking CCC prediction tools is challenging due to lack of ground truth

## Evidence

- [discussion] In our ASD case-study (Fig. 5), such analyses included GSEA, clustering, visualization and statistical comparison of factors: "In our ASD case-study (Fig. 5), such analyses included GSEA, clustering, visualization and statistical comparison of factors"
- [methods] cell2cell.clustering includes scipy-based clustering functions: "cell2cell.clustering : Includes multiple scipy-based functions for performing clustering methods."
- [discussion] we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [results] Factors 3 and 4 significantly distinguish ASD patients from controls per task finding: [evidence span withheld — anchor status: not_found]
- [methods] the latter is for deconvolving complex patterns of cell-cell communication across multiple samples or contexts simultaneously into interpretable factors: "the latter is for deconvolving complex patterns of cell-cell communication across multiple samples or contexts simultaneously into interpretable factors"
