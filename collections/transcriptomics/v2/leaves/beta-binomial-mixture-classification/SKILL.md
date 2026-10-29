---
name: beta-binomial-mixture-classification
description: Use when estimating per-cell per-gene allelic expression states (P/B/M)
  from low-coverage allele-specific scRNA-Seq counts (e.g., F1 hybrid CAST×B6 data),
  where U-shaped allelic-proportion distributions and sampling zeros require borrowing
  information across cells.
when_to_use_negative:
- Data is bulk RNA-Seq or lacks allele-resolved counts from a phase-known diploid
  reference — no per-cell allelic proportions to classify
- Genes have high read coverage, where partial pooling adds little benefit (its impact
  is greatest at low coverage, e.g., <10 unique reads per gene, and with many cells)
- A cell-population-context model is not desired because cell-to-cell heterogeneity
  in allelic state must not be smoothed across cells
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scBASE
  role: Implements the beta-binomial mixture classification, the iterative classification/partial-pooling
    loop, and the seven ASE profile classes
  repo: https://github.com/kbchoi-jax/scbase
- name: Stan
  role: Probabilistic programming backend in which the hierarchical beta-binomial
    mixture model is specified and fit (MCMC stage and EM classification)
- name: EMASE
  role: EM-based weighted allocation of genomic and allelic multi-reads, producing
    the weighted counts that serve as one input counting scheme for classification
    and pooling
- name: g2gtools
  role: Reconstruction of the CAST genome by incorporating known SNPs and short indels
    into the reference mouse genome to build the diploid transcriptome index
  repo: http://churchill-lab.github.io/g2gtools/
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:728381dbb68364108edfc84dab8d2d8589b9394d36d1ee882794b9506f3676e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbchoi-jax/scbase
  tools:
  - EMASE
  - Stan
  - g2gtools
  - scBASE
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# beta-binomial-mixture-classification

## Summary

scBASE's probabilistic classification of allelic expression state for each gene in each cell, modeling maternal read counts x_gk as drawn from a mixture of three beta-binomial distributions (paternal monoallelic, bi-allelic, maternal monoallelic) fitted with Stan, coupled to partial pooling of allelic proportions across cells.

## When to use

Use when estimating per-cell per-gene allelic expression states (P/B/M) from low-coverage allele-specific scRNA-Seq counts (e.g., F1 hybrid CAST×B6 data), where U-shaped allelic-proportion distributions and sampling zeros require borrowing information across cells.

## When NOT to use

- Data is bulk RNA-Seq or lacks allele-resolved counts from a phase-known diploid reference — no per-cell allelic proportions to classify
- Genes have high read coverage, where partial pooling adds little benefit (its impact is greatest at low coverage, e.g., <10 unique reads per gene, and with many cells)
- A cell-population-context model is not desired because cell-to-cell heterogeneity in allelic state must not be smoothed across cells

## Inputs

- Allele-specific gene-by-cell read count matrix (maternal/paternal counts) from allele-aware alignment to a phase-known diploid transcriptome
- Unique-read counts (multi-reads discarded) and/or weighted-allocation counts resolved by the EM algorithm in EMASE
- Full-data allelic proportion estimates for validation (e.g., 122 early/mid/late blastocyst cells, 1% down-sampled reads)
- Phase-known diploid genome index (CAST genome reconstructed with g2gtools; aligned with bowtie --all --best --strata -v 3)

## Outputs

- Per-cell per-gene posterior state assignments (P, B, or M) with mixture component probabilities
- Pooled cell-level allelic proportion estimates per gene
- Gene-level ASE profile class (P, B, M, PB, BM, MP, PBM) at π_g·s > 0.7 majority threshold
- MSE comparison table (mse_comparison.csv) of down-sampled vs full-data estimates per counting scheme

## How to apply

First obtain allele-specific counts via the unique-reads method (discard all genomic and allelic multi-reads) or via EM-based weighted allocation of multi-reads in EMASE, treating an allele as expressed if its estimated abundance is greater than one read (or one UMI). In scBASE (Stan-backed), model the maternal read count x_gk of each gene in each cell as drawn from one of three beta-binomial mixture components representing the P, B, and M states, initializing the mixture weights and component Beta densities; run the MCMC stage on pooled cells to estimate state-specific priors, then fix these priors and run the EM classification per stage/subpopulation. Assign each gene-cell observation to its most probable state from posterior component probabilities, then re-estimate class-specific allelic proportions by partial pooling (the weighted average across cells assigned to the same state) and update the mixture weights and Beta densities with the pooled values, alternating until convergence. To summarize a gene across cells, assign it to a majority-state ASE profile class (P, B, M, PB, BM, MP, PBM) when π_g·s > 0.7 of cells share a state; note the class boundaries are explicitly acknowledged as arbitrary. Partial pooling has the biggest impact when read coverage is low and the number of cells is large.

## Related tools

- **scBASE** (Implements the beta-binomial mixture classification, the iterative classification/partial-pooling loop, and the seven ASE profile classes) — https://github.com/kbchoi-jax/scbase
- **Stan** (Probabilistic programming backend in which the hierarchical beta-binomial mixture model is specified and fit (MCMC stage and EM classification))
- **EMASE** (EM-based weighted allocation of genomic and allelic multi-reads, producing the weighted counts that serve as one input counting scheme for classification and pooling)
- **g2gtools** (Reconstruction of the CAST genome by incorporating known SNPs and short indels into the reference mouse genome to build the diploid transcriptome index) — http://churchill-lab.github.io/g2gtools/

## Examples

```
git clone git://github.com/kbchoi-jax/scbase; cd scbase; python setup.py install; python -c 'import scbase'
```

## Evaluation signals

- Convergence of the alternating classification/pooling loop: mixture weights and Beta densities stabilize and state assignments stop changing
- Down-sampling MSE check on 1%-sampled blastocyst reads: pooling improves the majority of genes (unique reads 4,392 vs 1,367 of 5,759, avg MSE diff ~0.018; weighted allocation 5,078 vs 1,673 of 6,751, avg diff ~0.016)
- Largest MSE gains concentrate among low-coverage genes (<10 unique reads per gene), consistent with pooling helping most where coverage is low
- Classification reproduces known stage dynamics on Deng et al. 286-cell time course: maternal-monoallelic dominance in zygote/early 2-cell, B peak at late 2-cell/4-cell, PBM dominance from 8-cell onward
- Genes are only called expressed when estimated abundance exceeds one read (or one UMI), keeping the state assignment restricted to detectable alleles

## Limitations

- MCMC settings (number of iterations, burn-in, prior specifics) are deferred to Supplementary Methods, so exact reproduction requires the supplement
- The ASE class boundaries (π_g·s > 0.7 majority threshold) are arbitrary and acknowledged as only a simple descriptive summary
- Discarding multi-mapping reads (unique-reads scheme) leads to higher variability in allelic proportion estimates, more sampling zeros, and can produce spurious findings of dynamic and monoallelic expression
- No down-sampling seed/randomization procedure is specified for the 1% read sampling, and no GEO/SRA accession IDs are provided for the datasets

## Evidence

- [discussion] computing a probabilistic classification based on a mixture model in which the maternal read counts xgk are drawn from one of three beta-binomial distributions: [evidence span withheld — anchor status: not_found]
- [intro] Lastly, we can apply partial pooling to improve the individual cell-level estimation by combining information across cells that are in the same allelic expression state.: "Lastly, we can apply partial pooling to improve the individual cell-level estimation by combining information across cells that are in the same allelic expression state."
- [discussion] If a majority of cells (πg⋅s>0.7) are in a particular ASE state... then we will assign the gene to the class: [evidence span withheld — anchor status: not_found]
- [readme] We propose a new method, **scBASE**, a “soft” zero-and-one inflated model for cellular allelic proportions in which we tackle data sparsity and variability by harnessing information derived from cell's population context.: "We propose a new method, **scBASE**, a “soft” zero-and-one inflated model for cellular allelic proportions in which we tackle data sparsity and variability by harnessing information derived from"
