---
name: nonnegative-tensor-component-analysis
description: Use when you have communication scores assembled into a multi-dimensional
  tensor (e.g., 23 samples x 749 LR pairs x 16 x 16 cell types, or a simulated 3 x
  300 x 12 tensor) and need to deconvolve latent context-driven communication patterns
  across multiple contexts simultaneously.
when_to_use_negative:
- The communication tensor contains many missing values — the implemented tensor decomposition
  algorithm is not optimized for missing values, and the case-study tensors only included
  cell types shared across all samples
- Only a single context/sample is available — regular cell2cell (not Tensor-cell2cell)
  is for inferring cell-cell interactions in one sample or context
- Pathway-level rather than LR-pair-level scores are required — tools such as CellChat
  generate scores at the signaling pathway level instead of LR pairs
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Tensor-cell2cell
  role: Unsupervised TCA-based method that deconvolves intercellular communication
    across multiple contexts into interpretable factors
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Python suite providing the tensor module for assembling the 4D tensor, stats
    module for Bonferroni-corrected t-tests, and clustering module for scipy-based
    hierarchical clustering of sample loadings
  repo: https://github.com/earmingol/cell2cell
- name: tensorly
  role: Backend on which Tensor-cell2cell's tensor decomposition is built
- name: Anaconda
  role: Environment manager used to install cell2cell via pip in a dedicated conda
    environment (python=3.7)
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:f3091f76dca3035da85ea08610e4a71362357da8b807ed5dd39f692947c63f6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - Anaconda
  - Tensor-cell2cell
  - cell2cell
  - tensorly
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# nonnegative-tensor-component-analysis

## Summary

Non-negative tensor component analysis (TCA), as implemented in Tensor-cell2cell (built on tensorly), decomposes a 4D cell-cell communication tensor (contexts x LR pairs x sender cell types x receiver cell types) into interpretable factors capturing context-dependent communication modules.

## When to use

Use when you have communication scores assembled into a multi-dimensional tensor (e.g., 23 samples x 749 LR pairs x 16 x 16 cell types, or a simulated 3 x 300 x 12 tensor) and need to deconvolve latent context-driven communication patterns across multiple contexts simultaneously.

## When NOT to use

- The communication tensor contains many missing values — the implemented tensor decomposition algorithm is not optimized for missing values, and the case-study tensors only included cell types shared across all samples
- Only a single context/sample is available — regular cell2cell (not Tensor-cell2cell) is for inferring cell-cell interactions in one sample or context
- Pathway-level rather than LR-pair-level scores are required — tools such as CellChat generate scores at the signaling pathway level instead of LR pairs

## Inputs

- 4D-communication tensor (contexts x LR pairs x sender cell types x receiver cell types), e.g., 23 x 749 x 16 x 16
- Per-context ligand-receptor communication scores computed from single-nucleus/single-cell RNA-seq (e.g., PRJNA434002 ASD prefrontal cortex dataset, 13 ASD + 10 controls)
- Curated ligand-receptor pair list (e.g., Human-2020-Jin-LR-pairs.csv from LewisLabUCSD/Ligand-Receptor-Pairs)
- Simulated communication scores with assigned temporal patterns (oscillation, pulse, exponential decay, linear decrease)

## Outputs

- Factor loading matrices for context, LR-pair, sender cell-type, and receiver cell-type axes
- Rank-selection curve of normalized reconstruction error across scanned component ranks
- Statistical test results comparing factor loadings between conditions (e.g., ASD vs control t-tests with Bonferroni correction)
- Hierarchical clustering of the sample-by-factor loading matrix
- Saved results object (e.g., simulation_tca_results.pkl)

## How to apply

First assemble communication scores (computed per context with any CCC tool such as cell2cell or CellChat) into a 4D-communication tensor using cell2cell.tensor, restricting to cell types shared across all samples. Then run non-negative tensor component analysis via Tensor-cell2cell (built on tensorly) with init='svd', switching to init='random' if memory errors occur as documented. Select the number of factors by scanning component ranks and picking the rank that minimizes the normalized reconstruction error (e.g., rank 6 for the ASD prefrontal cortex study; rank 4 recovered all four embedded patterns in the simulated 3 cell types x 300 LR pairs x 12 time-point contexts tensor). Finally extract factor loadings for the context, LR-pair, and cell-type axes and couple them with downstream analyses (t-tests with Bonferroni correction via cell2cell.stats, hierarchical clustering via cell2cell.clustering, GSEA) to interpret each factor.

## Related tools

- **Tensor-cell2cell** (Unsupervised TCA-based method that deconvolves intercellular communication across multiple contexts into interpretable factors) — https://github.com/earmingol/cell2cell
- **cell2cell** (Python suite providing the tensor module for assembling the 4D tensor, stats module for Bonferroni-corrected t-tests, and clustering module for scipy-based hierarchical clustering of sample loadings) — https://github.com/earmingol/cell2cell
- **tensorly** (Backend on which Tensor-cell2cell's tensor decomposition is built)
- **Anaconda** (Environment manager used to install cell2cell via pip in a dedicated conda environment (python=3.7))

## Examples

```
conda create -n cell2cell -y python=3.7 jupyter; conda activate cell2cell; pip install cell2cell
```

## Evaluation signals

- In the simulated tensor, the expected rank is 4: the rank that minimizes normalized reconstruction error should match the number of introduced communication patterns, and factor loadings on the LR-pair axis should correspond to the assigned oscillation, pulse, exponential-decay, and linear-decrease patterns
- For real data, factor loadings for known condition-relevant factors (e.g., factors 3 and 4 in the ASD study) must show significant differences between ASD (n=13) and control (n=10) samples after Bonferroni multiple-test correction
- Hierarchical clustering of the sample-by-factor loading matrix should resolve main clusters concordant with condition labels (e.g., 4 clusters aligned with controls vs ASD)
- Rank-4 solution should be robust when Gaussian noise is added to simulated scores
- If init='svd' raises a memory error, rerunning with init='random' should still yield a convergent decomposition

## Limitations

- The implemented tensor decomposition algorithm is not optimized for missing values, restricting analyses to cell types shared across all samples and potentially missing rare cell types
- Further developments are needed to include all possible communication members (LR pairs and cell types missing in some contexts)
- Benchmarking is challenging due to the lack of a ground truth for CCC prediction tools
- Reproducibility relies on analyses, data, and exact code/software versions deposited in a Code Ocean capsule

## Evidence

- [intro] TCA-based strategy that deconvolves intercellular communication across multiple contexts and uncovers modules: "Here, we introduce Tensor-cell2cell, a TCA-based strategy that deconvolves intercellular communication across multiple contexts and uncovers modules, or latent context-dependent patterns, of CCC."
- [results] Rank selection by minimizing normalized reconstruction error: [evidence span withheld — anchor status: not_found]
- [discussion] Only shared cell types included in the tensor: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [discussion] Algorithm not optimized for missing values: "the implemented tensor decomposition algorithm can be further optimized for missing values. Since the implemented algorithm is not optimized for this purpose"
- [readme] cell2cell installation via pip in conda environment: [evidence span withheld — anchor status: not_found]
