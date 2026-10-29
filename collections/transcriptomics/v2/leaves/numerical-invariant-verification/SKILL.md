---
name: numerical-invariant-verification
description: Use when a SiFT-filtered single-cell expression matrix (X~ = X − K·X
  with pseudocount α = min(X~)) must be algebraically validated before saving to .h5ad
  or feeding downstream analysis.
when_to_use_negative:
- Filtering used a non-row-stochastic kernel or a non-SiFT method, so variance preservation
  after K·X projection does not hold by construction and the invariants would give
  misleading pass/fail signals
- The goal is biological (not algebraic) correctness of the filtering — the article
  notes SiFT 'suffers from the general challenge of any computational method which
  lacks a ground truth reference', so invariant checks cannot substitute for biological
  validation
- The matrix has been re-normalized or log-transformed after SiFT's α shift, since
  SiFT's corrected expression 'is no longer properly log-transformed' and comparisons
  to the original counts are then invalid
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SiFT (sift-sc)
  role: Produces the kernel projection (P = K·X), the filtered matrix X~ = X − K·X,
    and the pseudocount-corrected output whose invariants this skill verifies
  repo: https://github.com/nitzanlab/sift-sc
- name: pip
  role: Installs the sift-sc package ('pip install sift-sc') so the filtering output
    to be validated can be generated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:b384414aa7a67d3046146c7806ab15bb9e33f67417b6b95fd7cdfdbf8bcbaacc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - SiFT
  - SiFT (sift-sc)
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# numerical-invariant-verification

## Summary

Post-hoc numerical validation of the SiFT filtering output: after computing the filtered matrix X~ = X − K·X with the pseudocount correction α = min(X~), verify that total variance is preserved and pairwise gene differences are unchanged, before persisting results as filtered_expression.h5ad and a sift_invariants_report.

## When to use

Use when a SiFT-filtered single-cell expression matrix (X~ = X − K·X with pseudocount α = min(X~)) must be algebraically validated before saving to .h5ad or feeding downstream analysis.

## When NOT to use

- Filtering used a non-row-stochastic kernel or a non-SiFT method, so variance preservation after K·X projection does not hold by construction and the invariants would give misleading pass/fail signals
- The goal is biological (not algebraic) correctness of the filtering — the article notes SiFT 'suffers from the general challenge of any computational method which lacks a ground truth reference', so invariant checks cannot substitute for biological validation
- The matrix has been re-normalized or log-transformed after SiFT's α shift, since SiFT's corrected expression 'is no longer properly log-transformed' and comparisons to the original counts are then invalid

## Inputs

- Original single-cell gene expression count matrix X (cells x genes) used for variance comparison
- SiFT-filtered matrix X~ = X − K·X computed via sift.SiFT / sift.sifter with a row-stochastic kernel
- Pseudocount value α = min(X~) from the global correction step

## Outputs

- filtered_expression.h5ad containing the validated filtered expression matrix
- sift_invariants_report recording pass/fail of the variance-preservation and pairwise-difference invariants

## How to apply

After SiFT projects the expression count matrix onto the cell–cell similarity kernel (P = K·X, built with sift.kernels.MappingKernel, KnnKernel, RBFKernel, or LaplacianKernel) and subtracts the projection to yield X~, check that total variance of X~ matches the original matrix X within a small numerical tolerance — this holds by construction because K is row-stochastic. Then apply the global pseudocount correction α = min(X~) to shift filtered values back to a valid expression scale, and confirm that pairwise gene differences are unchanged after the α shift, since a global shift cannot alter gene-to-gene relationships. Write the validated filtered matrix to filtered_expression.h5ad and record the invariant check results in sift_invariants_report. Note that because SiFT performs correction at the gene expression level, the corrected data 'is no longer properly log-transformed', so flag this when handing off to log-expecting downstream tasks.

## Related tools

- **SiFT (sift-sc)** (Produces the kernel projection (P = K·X), the filtered matrix X~ = X − K·X, and the pseudocount-corrected output whose invariants this skill verifies) — https://github.com/nitzanlab/sift-sc
- **pip** (Installs the sift-sc package ('pip install sift-sc') so the filtering output to be validated can be generated)

## Examples

```
import numpy as np; Xt = X - K.dot(X); alpha = Xt.min(); Xt = Xt - alpha; assert np.isclose(Xt.var(), X.var())
```

## Evaluation signals

- Total variance of the filtered matrix X~ equals total variance of the original matrix X within a small numerical tolerance
- Pairwise gene differences are unchanged after the global α = min(X~) shift (shift-invariance check)
- The cell–cell similarity kernel K is row-stochastic (each row sums to 1), making K·X a valid expected expression over neighboring cells
- After pseudocount correction, filtered values sit on a valid expression scale (no values below the α offset)
- filtered_expression.h5ad and sift_invariants_report are written and the report records the invariant checks

## Limitations

- Invariants confirm algebraic consistency only; the article acknowledges 'SiFT suffers from the general challenge of any computational method which lacks a ground truth reference that can be used to evaluate the results', so passing invariants do not certify biological signal preservation
- SiFT modifies gene expression so that data 'is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks', limiting downstream comparability even when invariants pass
- SiFT 'strongly relies on the prior knowledge used to define the cell–cell similarity kernel' and 'an inaccurate or coarse-grained mapping may lead to a loss of signal' — invariants will still pass even when the kernel encodes a poor signal definition

## Evidence

- [other] invariant workflow check: [evidence span withheld — anchor status: not_found]
- [intro] projection step: "we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication"
- [discussion] log-transform caveat: "it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks"
- [discussion] no ground truth caveat: "SiFT suffers from the general challenge of any computational method which lacks a ground truth reference that can be used to evaluate the results."
- [discussion] removal of projection: "This kernel is then used to obtain a projection of gene expression onto the signal, which is then removed from the original expression."
