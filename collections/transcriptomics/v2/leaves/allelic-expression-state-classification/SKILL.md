---
name: allelic-expression-state-classification
description: Use when allelic read counts from single-cell RNA-Seq (e.g., CAST×B6
  F1 embryos) need per-cell, per-gene P/B/M state calls, and random monoallelic bursts
  and sparse coverage make naive read-ratio thresholds unreliable.
when_to_use_negative:
- When reads cannot be assigned allele-of-origin because the organism lacks a phase-known
  diploid reference (no SNP-split genome such as g2gtools-reconstructed CAST)
- When bulk (non-single-cell) RNA-Seq allelic proportions are the target — the beta-binomial
  mixture and partial pooling are designed for extremely low per-cell coverage and
  cell-to-cell heterogeneity
- When allele-specific counts are already high-confidence UMI-level per-allele transcript
  counts requiring burst-kinetics inference (e.g., txburst k_on/k_off/k_syn estimation)
  rather than state classification
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scBASE
  role: Implements the beta-binomial mixture classification and partial pooling of
    allelic expression states; installed from source via 'python setup.py install'
  repo: https://github.com/kbchoi-jax/scbase
- name: Stan
  role: Probabilistic programming backend for the MCMC stage of the hierarchical mixture
    model
- name: EMASE
  role: EM-based weighted allocation of multi-mapping reads to gene- and allele-specific
    counts used as classifier input
- name: bowtie v1.0.0
  role: Aligns reads to the diploid transcriptome index with --all --best --strata
    -v 3 to enumerate all best-scoring alignments including multi-reads
- name: g2gtools
  role: Reconstructs the CAST genome by incorporating known SNPs and short indels
    (Sanger REL-1505) into the reference mouse genome
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:3bfde453d0d4d5820713365aaa3b8139472d7a9e96eabf7278f420f0aaf9f130
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbchoi-jax/scbase
  tools:
  - EMASE
  - Stan
  - bowtie
  - bowtie v1.0.0
  - g2gtools
  - scBASE
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# allelic-expression-state-classification

## Summary

Probabilistic classification of each gene in each single cell into paternal monoallelic (P), bi-allelic (B), or maternal monoallelic (M) expression states using scBASE, a hierarchical mixture of beta-binomial distributions (Stan-backed), with optional partial pooling across cells to reduce sampling variability in low-coverage scRNA-Seq.

## When to use

Use when allelic read counts from single-cell RNA-Seq (e.g., CAST×B6 F1 embryos) need per-cell, per-gene P/B/M state calls, and random monoallelic bursts and sparse coverage make naive read-ratio thresholds unreliable.

## When NOT to use

- When reads cannot be assigned allele-of-origin because the organism lacks a phase-known diploid reference (no SNP-split genome such as g2gtools-reconstructed CAST)
- When bulk (non-single-cell) RNA-Seq allelic proportions are the target — the beta-binomial mixture and partial pooling are designed for extremely low per-cell coverage and cell-to-cell heterogeneity
- When allele-specific counts are already high-confidence UMI-level per-allele transcript counts requiring burst-kinetics inference (e.g., txburst k_on/k_off/k_syn estimation) rather than state classification

## Inputs

- Gene- and allele-specific read count matrix (maternal/paternal counts) per cell, e.g., 286 CAST×B6 F1 pre-implantation embryo cells across nine developmental stages
- Alignment-derived allelic counts from weighted allocation (EMASE) or unique-reads counting after bowtie alignment to a g2gtools-reconstructed diploid CAST/B6 transcriptome

## Outputs

- Per-cell, per-gene posterior P/B/M allelic expression state classifications
- Pooled (partial-pooling) class-specific allelic proportion estimates
- Gene-to-ASE-class assignments (P, B, M, PB, BM, MP, PBM) per stage with mixture weights, e.g., ase_class_assignments_286cells.csv

## How to apply

First resolve genomic and allelic multi-reads into gene- and allele-specific counts, preferably by EM-based weighted allocation in EMASE rather than the unique-reads method, since discarding multi-reads inflates monoallelic calls and retains only 14.9% of available reads. Then model each gene's maternal read count xgk as drawn from one of three beta-binomial mixture components (P, B, M), running the MCMC stage of the hierarchical model on all 286 Deng et al. pre-implantation cells to estimate state-specific priors, followed by per-stage EM classification with the MCMC priors fixed. An allele is regarded as expressed only if its estimated abundance is greater than one read (or one UMI). Alternating classification and partial-pooling updates — re-estimating class-specific allelic proportions as weighted averages across cells in the same state — until convergence yields four possible method variants. Genes per stage can then be assigned to the seven ASE profile classes (P, B, M, PB, BM, MP, PBM) using the mixture-weight threshold πg·s > 0.7 (two-state mixture > 0.9, otherwise PBM).

## Related tools

- **scBASE** (Implements the beta-binomial mixture classification and partial pooling of allelic expression states; installed from source via 'python setup.py install') — https://github.com/kbchoi-jax/scbase
- **Stan** (Probabilistic programming backend for the MCMC stage of the hierarchical mixture model)
- **EMASE** (EM-based weighted allocation of multi-mapping reads to gene- and allele-specific counts used as classifier input)
- **bowtie v1.0.0** (Aligns reads to the diploid transcriptome index with --all --best --strata -v 3 to enumerate all best-scoring alignments including multi-reads)
- **g2gtools** (Reconstructs the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome)

## Examples

```
git clone git://github.com/kbchoi-jax/scbase && cd scbase && python setup.py install && python -c "import scbase"
```

## Evaluation signals

- Exemplar gene calls match the paper: Pacs2 = MP, Mtdh = B, Timm23 and Tulp3 in BM/PB regions, Akr1b3 = PBM
- PBM-class fraction among blastocyst-stage genes falls within ~0.55–0.58 of the reported 56.9%
- Unique-reads vs weighted-allocation contrast reproduces reported values: 14.9% of reads retained, ~66 more monoallelic genes per cell, ~1,908 bi-allelic genes missed, Mtdh 88 vs 7 monoallelic cells
- Stage-wise ASE pattern proportions reproduce Fig 7a dynamics: maternal-dominated zygote/early 2-cell, B peak at late 2-cell/4-cell, PBM dominance from 8-cell onward
- Partial pooling shows the biggest precision gain (lowest mean squared error) when read coverage is low (1% down-sampled) and the number of cells is large

## Limitations

- ASE class boundaries (πg·s > 0.7, two-state mixture > 0.9) are acknowledged as arbitrary, limiting reproducibility of gene classification
- MCMC settings (iterations, burn-in, prior specifics) are deferred to Supplementary Methods and not given in the main text
- Discarding multi-mapping reads (unique-reads method) leads to higher variability in allelic proportion estimates, more sampling zeros, and spurious findings of dynamic and monoallelic expression

## Evidence

- [discussion] Probabilistic classification of allelic expression state via beta-binomial mixture: [evidence span withheld — anchor status: not_found]
- [discussion] Allele regarded as expressed if estimated abundance exceeds one read: "we regarded it as expressed if its estimated abundance is greater than one reads (or one UMI as in Larsson et al.22)"
- [discussion] Majority-state threshold for gene class assignment: [evidence span withheld — anchor status: not_found]
- [intro] Unique-reads counting discards multi-reads: "we exclude multi-mapping reads (multi-reads) and count only reads that map unambiguously to one allele of one gene"
- [readme] scBASE README description of soft zero-and-one inflated model: "We propose a new method, **scBASE**, a “soft” zero-and-one inflated model for cellular allelic proportions in which we tackle data sparsity and variability"
