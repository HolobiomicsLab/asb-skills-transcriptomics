---
name: odds-ratio-association-analysis
description: Use when single-cell allelic count matrices (e.g., scBASE unique-read,
  weighted-allocation, or partial-pooled estimates) are available and the goal is
  to test whether maternal/paternal bursting is coordinated rather than independent.
when_to_use_negative:
- Input is bulk or non-allelic expression data with no maternal/paternal read separation
  — the logOR of allelic burst co-occurrence cannot be defined
- Allelic counts were obtained by discarding multi-mapping reads without the weighted-allocation
  or partial-pooling alternative, when the goal is to quantify pooling effects on
  the independence test
- The data are UMI counts from a single experiment where the question is differential
  bursting between two samples (txburstTEST.py territory), not within-cell allelic
  burst association
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SCALE
  role: Analyzes dynamics of gene expression; fits the bursting model to scBASE estimated
    counts and yields the burst-correlation log odds ratio and independence tests
- name: scBASE
  role: Produces the allele-specific count matrices (unique-read, weighted allocation,
    partial pooling) that serve as input to SCALE
  repo: https://github.com/churchill-lab/scBASE
- name: EMASE
  role: EM-based weighted allocation of multi-mapping reads used to generate one of
    the count matrices
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:3eb8e40b4d793fcec9620511e06b4e92890fb2cdfbadaecf1870f9897e7cefbd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/churchill-lab/scBASE
  tools:
  - EMASE
  - SCALE
  - scBASE
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# odds-ratio-association-analysis

## Summary

Tests whether maternal and paternal allelic bursting events occur independently by fitting the SCALE bursting model to scBASE-estimated allele-specific counts and computing a per-gene burst-correlation log odds ratio (logOR), with Benjamini-Hochberg FDR correction.

## When to use

Use when single-cell allelic count matrices (e.g., scBASE unique-read, weighted-allocation, or partial-pooled estimates) are available and the goal is to test whether maternal/paternal bursting is coordinated rather than independent.

## When NOT to use

- Input is bulk or non-allelic expression data with no maternal/paternal read separation — the logOR of allelic burst co-occurrence cannot be defined
- Allelic counts were obtained by discarding multi-mapping reads without the weighted-allocation or partial-pooling alternative, when the goal is to quantify pooling effects on the independence test
- The data are UMI counts from a single experiment where the question is differential bursting between two samples (txburstTEST.py territory), not within-cell allelic burst association

## Inputs

- scBASE-estimated allelic count matrices (unique-read method) for allele-specific read counts
- scBASE-estimated allelic count matrices (EM-based weighted allocation of multi-reads)
- scBASE counts with partial pooling applied across cells in the same allelic expression state
- 8,290 analyzed genes from 122 blastocyst cells (early, mid, late)

## Outputs

- Per-gene burst-correlation log odds ratio (logOR) values
- Per-gene independence-test p-values adjusted at 5% FDR (Benjamini-Hochberg)
- Summary table (scale_independence_results.csv) of significant-gene counts and |logOR|>2 counts per scBASE method

## How to apply

Load the four scBASE-estimated allelic count matrices (unique-read, weighted-allocation, and each with partial pooling) for the analyzed genes (8,290 genes in the 122-cell blastocyst analysis). Fit the SCALE bursting model to each matrix to estimate allele-specific bursting rates and compute the per-gene burst-correlation log odds ratio (logOR). Test per-gene deviation from independence and adjust p-values at a 5% FDR threshold using Benjamini-Hochberg correction. Tabulate, per scBASE method, the number of genes with significant non-independence (logOR > 0) and the number with |logOR| > 2, comparing before versus after partial pooling to quantify the pooling effect.

## Related tools

- **SCALE** (Analyzes dynamics of gene expression; fits the bursting model to scBASE estimated counts and yields the burst-correlation log odds ratio and independence tests)
- **scBASE** (Produces the allele-specific count matrices (unique-read, weighted allocation, partial pooling) that serve as input to SCALE) — https://github.com/churchill-lab/scBASE
- **EMASE** (EM-based weighted allocation of multi-mapping reads used to generate one of the count matrices)

## Examples

```
pd.read_pickle('scale_independence_results.csv')  # or: SCALE fit per gene; see github.com/kbchoi-jax/scbase: git clone git://github.com/kbchoi-jax/scbase
```

## Evaluation signals

- Reproduced significant-gene counts at 5% FDR must match 3,381 (unique reads), 4,815 (weighted allocation), 6,068 (unique+partial pooling), and 6,761 (weighted+partial pooling) out of 8,290 genes
- Reproduced |logOR|>2 counts must match 2,845/3,763 before pooling and 5,622/6,209 after pooling (Fig 5, Supplementary Fig 2)
- Majority of significant genes must show logOR > 0, i.e., positive burst correlation, rejecting both perfect independence and perfect dependence
- Summary CSV (scale_independence_results.csv) must contain per-method counts for all four scBASE count matrices
- Partial pooling must increase the number of significant genes relative to the un-pooled counterpart for both unique-read and weighted-allocation inputs

## Limitations

- Discarding multi-mapping reads (unique-reads method) leads to higher variability in allelic proportion estimates, more sampling zeros, and can produce spurious findings, biasing the independence test
- MCMC settings for SCALE (iterations, burn-in, priors) are deferred to Supplementary Methods and not given in the main text
- The txburst repository notes a data duplication issue in SS3_c57_UMIs_mESC.csv; unrelated to SCALE but relevant if mixing burst-inference tooling

## Evidence

- [discussion] Allelic bursting is statistically dependent and positively correlated: [evidence span withheld — anchor status: not_found]
- [discussion] SCALE uses scBASE estimated counts as input: [evidence span withheld — anchor status: not_found]
- [discussion] Partial pooling impact depends on coverage and cell number: "Partial pooling has the biggest impact when read coverage is low and the number of cells is large"
- [readme] scBASE provides soft zero-and-one inflated model for allelic proportions: "We propose a new method, **scBASE**, a "soft" zero-and-one inflated model for cellular allelic proportions in which we tackle data sparsity and variability"
- [intro] Application to independence of allelic bursting: [evidence span withheld — anchor status: not_found]
