---
name: cross-method-benchmark-comparison
description: Use when comparing two or more count-estimation or preprocessing variants
  (e.g. unique vs weighted multi-read allocation, with vs without partial pooling)
  by feeding each through an identical downstream statistical test and reporting matched
  result counts.
when_to_use_negative:
- You only have a single count-estimation method with no alternative variant to compare
  — there is no benchmark contrast to tabulate
- The downstream question is differential bursting kinetics between two samples rather
  than a method-comparison benchmark (use a dedicated differential test instead)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCALE
  role: Downstream bursting-dynamics model fit to each scBASE count matrix to compute
    per-gene burst-correlation logOR and independence test p-values
- name: scBASE
  role: Produces the allelic count matrices under each variant (unique-read, weighted
    allocation, partial pooling) that are benchmarked head-to-head
  repo: github.com/churchill-lab/scBASE
- name: EMASE
  role: EM-based weighted allocation of multi-reads, one of the count-estimation variants
    in the comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:ec26083fa770b7cf8d7f957babcccc6c582785cab89d3cca41a3653b3c0e3ac4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMASE
  - SCALE
  - scBASE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# cross-method-benchmark-comparison

## Summary

Benchmark multiple allelic count-estimation methods (unique-reads, EMASE weighted allocation, with/without scBASE partial pooling) by running the same downstream test on each and tabulating method-by-method result counts, as done in Choi et al. 2019 where the SCALE bursting model was fit to four scBASE count matrices and independence test outcomes were compared.

## When to use

Use when comparing two or more count-estimation or preprocessing variants (e.g. unique vs weighted multi-read allocation, with vs without partial pooling) by feeding each through an identical downstream statistical test and reporting matched result counts.

## When NOT to use

- You only have a single count-estimation method with no alternative variant to compare — there is no benchmark contrast to tabulate
- The downstream question is differential bursting kinetics between two samples rather than a method-comparison benchmark (use a dedicated differential test instead)

## Inputs

- Four scBASE-estimated allelic count matrices for 8,290 genes × 122 blastocyst cells (unique-read, weighted-allocation, unique+partial-pooling, weighted+partial-pooling)
- Allelic read alignments from the CAST×B6 F1 hybrid blastocyst scRNA-Seq data (Deng et al.)
- Bursting model specification (SCALE) for allele-specific bursting rates and the burst-correlation logOR statistic

## Outputs

- Per-method table of significant non-independent genes (logOR > 0) at 5% FDR: 3,381 (unique), 4,815 (weighted), 6,068 (unique+PP), 6,761 (weighted+PP)
- Per-method counts of genes with |logOR| > 2: 2,845/3,763 before pooling and 5,622/6,209 after pooling, out of 8,290 genes
- scale_independence_results.csv summary table

## How to apply

Fit the identical downstream model — here, the SCALE bursting model — to each of the four scBASE-estimated allelic count matrices (unique-read, weighted-allocation, unique+partial-pooling, weighted+partial-pooling) for the same 8,290 analyzed genes from 122 blastocyst cells. For each gene, compute the burst-correlation log odds ratio (logOR) and test deviation from allelic-bursting independence. Adjust p-values with Benjamini-Hochberg correction at a 5% FDR threshold and count significant genes with logOR > 0 per method, plus genes with |logOR| > 2, before and after pooling. The rationale is that unique-reads and weighted allocation handle multi-mapping reads differently, and partial pooling has its biggest impact when read coverage is low and the number of cells is large, so the comparison quantifies how each choice affects downstream inference. Write the per-method summary to a table (e.g. scale_independence_results.csv) and validate against the published values.

## Related tools

- **SCALE** (Downstream bursting-dynamics model fit to each scBASE count matrix to compute per-gene burst-correlation logOR and independence test p-values)
- **scBASE** (Produces the allelic count matrices under each variant (unique-read, weighted allocation, partial pooling) that are benchmarked head-to-head) — github.com/churchill-lab/scBASE
- **EMASE** (EM-based weighted allocation of multi-reads, one of the count-estimation variants in the comparison)

## Examples

```
git clone git://github.com/kbchoi-jax/scbase  # obtain scBASE to generate the four allelic count-matrix variants for the SCALE benchmark
```

## Evaluation signals

- Significant-gene counts at 5% FDR must equal the reported values: 3,381 (unique), 4,815 (weighted), 6,068 (unique+PP), 6,761 (weighted+PP)
- Genes with |logOR| > 2 must equal 2,845 and 3,763 before pooling and 5,622 and 6,209 after pooling, out of 8,290 genes total
- All per-gene tests must use Benjamini-Hochberg adjustment at a 5% FDR threshold
- The majority of genes must show positive logOR (logOR > 0), i.e. statistically dependent and positively correlated bursting, and must reject both perfect independence and perfect dependence
- The same 8,290-gene set must be analyzed across all four count-matrix variants so counts are directly comparable

## Limitations

- Discarding multi-mapping reads (unique-reads method) leads to higher variability in allelic proportion estimates, increased frequency of sampling zeros, and can lead to spurious findings of dynamic and monoallelic expression — motivating the weighted-allocation comparison
- MCMC settings for the bursting model (iterations, burn-in, priors) are deferred to Supplementary Methods and not stated in the main text, so exact reproduction may require supplementary detail
- No data accession IDs (e.g. GEO/SRA) are given for the underlying scRNA-Seq datasets, complicating independent re-acquisition of inputs

## Evidence

- [discussion] SCALE software, which analyzes the dynamics of gene expression, using scBASE estimated counts as input: [evidence span withheld — anchor status: not_found]
- [discussion] Partial pooling has the biggest impact when read coverage is low and the number of cells is large: "Partial pooling has the biggest impact when read coverage is low and the number of cells is large"
- [discussion] the pattern of bursting is statistically dependent and positively correlated (logOR>0) for the majority of genes: [evidence span withheld — anchor status: not_found]
- [discussion] Unique-read counts are obtained directly from counting reads after discarding all genomic and allelic multi-reads: "Unique-read counts are obtained directly from counting reads after discarding all genomic and allelic multi-reads"
- [readme] scBASE, a 'soft' zero-and-one inflated model for cellular allelic proportions: "We propose a new method, **scBASE**, a "soft" zero-and-one inflated model for cellular allelic proportions"
