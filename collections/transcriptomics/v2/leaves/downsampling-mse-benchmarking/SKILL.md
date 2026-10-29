---
name: downsampling-mse-benchmarking
description: Use when evaluating whether a pooling or counting strategy improves single-cell
  allelic proportion estimates from sparsely sequenced scRNA-Seq data, using down-sampled
  reads and MSE against full-data estimates as the accuracy metric.
when_to_use_negative:
- The data are UMI-based counts without allelic multi-read ambiguity (e.g. the Larsson
  et al. 224-cell UMI data), where multi-read allocation and pooling gains of this
  kind do not apply as designed.
- The goal is inference of transcriptional burst kinetics parameters (k_on, k_off,
  k_syn) rather than allelic proportion estimation accuracy — use txburst instead.
- Full-depth data are unavailable, so no reference allelic proportions exist against
  which down-sampled MSE can be computed.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scBASE
  role: Fits the mixture of three beta-binomial distributions (via Stan) to classify
    genes into paternal monoallelic, bi-allelic, or maternal monoallelic states and
    produces pooled and unpooled allelic proportion estimates.
  repo: github.com/churchill-lab/scBASE
- name: EMASE
  role: EM-based weighted allocation of multi-reads to produce the weighted-allocation
    count matrix used as one of the two counting schemes in the MSE comparison.
- name: Stan
  role: Probabilistic programming backend used by scBASE to fit the hierarchical beta-binomial
    mixture model during classification and estimation.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:a08f77bd65f855771b16f28ca44b5df23fcd838165c90f1d76d4f22117e74952
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMASE
  - Stan
  - scBASE
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# downsampling-mse-benchmarking

## Summary

Benchmark estimation precision of allelic proportion methods by randomly down-sampling reads (1% in 122 blastocyst cells), re-estimating proportions under alternative read-counting schemes, and comparing per-gene mean squared error (MSE) against full-data estimates.

## When to use

Use when evaluating whether a pooling or counting strategy improves single-cell allelic proportion estimates from sparsely sequenced scRNA-Seq data, using down-sampled reads and MSE against full-data estimates as the accuracy metric.

## When NOT to use

- The data are UMI-based counts without allelic multi-read ambiguity (e.g. the Larsson et al. 224-cell UMI data), where multi-read allocation and pooling gains of this kind do not apply as designed.
- The goal is inference of transcriptional burst kinetics parameters (k_on, k_off, k_syn) rather than allelic proportion estimation accuracy — use txburst instead.
- Full-depth data are unavailable, so no reference allelic proportions exist against which down-sampled MSE can be computed.

## Inputs

- 1% down-sampled allele-specific read counts for 122 early/mid/late blastocyst cells (unique-read and EMASE weighted-allocation variants)
- Full-data allelic proportion estimates per gene per cell (reference truth for MSE)
- Allelic expression state classifications (P/B/M) from the scBASE beta-binomial mixture model

## Outputs

- mse_comparison.csv with per-gene MSE for each of the four scBASE methods
- Counts of genes improved by partial pooling (unique reads: 4,392 of 5,759; weighted allocation: 5,078 of 6,751)
- Average MSE differences by counting scheme and expression-level strata (largest gains at <10 unique reads per gene)

## How to apply

Load 1% down-sampled allele-specific read counts from the 122 early/mid/late blastocyst cells alongside the full-data allelic proportion estimates. Derive two counting schemes from the same alignments: unique-read counts (discard all genomic and allelic multi-reads) and weighted-allocation counts (resolve multi-reads via the EM algorithm in EMASE). Classify each gene in each cell into paternal monoallelic, bi-allelic, or maternal monoallelic states using the scBASE mixture of three beta-binomial distributions fitted via Stan, then estimate allelic proportions under four scBASE methods (with/without partial pooling, i.e. the weighted average of class-specific proportions across cells in the same state). Compute per-gene MSE of each down-sampled estimate against the full-data estimate for each method and count genes whose MSE improves under pooling. The rationale is that partial pooling has the biggest impact when read coverage is low and the number of cells is large, so the greatest gains are expected for genes with fewer than ~10 unique reads.

## Related tools

- **scBASE** (Fits the mixture of three beta-binomial distributions (via Stan) to classify genes into paternal monoallelic, bi-allelic, or maternal monoallelic states and produces pooled and unpooled allelic proportion estimates.) — github.com/churchill-lab/scBASE
- **EMASE** (EM-based weighted allocation of multi-reads to produce the weighted-allocation count matrix used as one of the two counting schemes in the MSE comparison.)
- **Stan** (Probabilistic programming backend used by scBASE to fit the hierarchical beta-binomial mixture model during classification and estimation.)

## Examples

```
git clone git://github.com/kbchoi-jax/scbase
```

## Evaluation signals

- Number of genes improved by partial pooling under unique reads falls in the reported range: 4,392 of 5,759 genes.
- Number of genes improved under weighted allocation falls in the reported range: 5,078 of 6,751 genes.
- Average MSE differences are approximately 0.018 (unique reads) and 0.016 (weighted allocation).
- Genes with the greatest MSE improvement concentrate in the low-expression stratum (<10 unique reads per gene), reproducing the Fig 3 pattern.
- mse_comparison.csv is written with per-gene MSE values for all four scBASE methods (with/without pooling × two counting schemes).

## Limitations

- The down-sampling seed/randomization procedure for sampling 1% of reads is not specified in the article, so exact per-gene MSE values may not be bit-for-bit reproducible.
- Discarding multi-mapping reads (unique-reads scheme) leads to higher variability in allelic proportion estimates, increased sampling zeros, and spurious findings of monoallelic expression — the comparison itself quantifies this cost.
- MCMC settings (iterations, burn-in, priors) are deferred to Supplementary Methods, which may be needed for exact replication.
- No data accession IDs (GEO/SRA) are given for the blastocyst scRNA-Seq dataset beyond Illumina HiSeq 2000 platform GPL12112.

## Evidence

- [discussion] Down-sampling to evaluate estimation precision: "We randomly sampled 1% of reads in each of 122 cells at the early, mid, and late blastocyst stages"
- [discussion] Unique-read counting scheme definition: "Unique-read counts are obtained directly from counting reads after discarding all genomic and allelic multi-reads"
- [discussion] Weighted allocation via EMASE: [evidence span withheld — anchor status: not_found]
- [discussion] Partial pooling rationale and expected regime of benefit: "Partial pooling has the biggest impact when read coverage is low and the number of cells is large"
- [discussion] Pooling method definition: "estimate the allelic proportions as a weighted average"
