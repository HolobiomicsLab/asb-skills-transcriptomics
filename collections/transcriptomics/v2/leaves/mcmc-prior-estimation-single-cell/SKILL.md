---
name: mcmc-prior-estimation-single-cell
description: Use when analyzing allele-specific expression (ASE) from scRNA-Seq allele
  counts (e.g., CAST×B6 F1 embryo data) and you need population-informed priors to
  overcome low per-cell coverage before assigning P/B/M expression states per gene
  per cell.
when_to_use_negative:
- Data are bulk RNA-Seq or already-aggregated allelic proportions — the hierarchical
  pooling across cells is designed for single-cell sparsity
- Reads cannot be assigned to a phase-known diploid genome (no strain SNP/indel reconstruction),
  so maternal/paternal counts are unreliable
- The goal is transcriptional burst kinetics inference (burst frequency/size), for
  which txburst-style ML/PL methods on UMI counts are the appropriate choice, not
  ASE state classification
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: scBASE
  role: Implements the MCMC stage of the hierarchical beta-binomial mixture for state-specific
    prior estimation and the downstream EM classification; installed from source ('python
    setup.py install')
  repo: https://github.com/kbchoi-jax/scbase
- name: Stan
  role: Probabilistic programming language used to implement the Bayesian mixture
    model underlying scBASE
- name: EMASE
  role: EM-based weighted allocation of multi-mapping reads to produce the allelic
    read counts that feed prior estimation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:c0caf8a67ef6b1c4bd8be64204c5c9281c196de845fe847672b4f728e4142dac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbchoi-jax/scbase
  tools:
  - EMASE
  - Stan
  - scBASE
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mcmc-prior-estimation-single-cell

## Summary

Estimates state-specific priors (concentration/hyperparameters of the beta-binomial) by running the MCMC stage of scBASE's hierarchical mixture model on allele-specific read counts pooled across all cells, then fixes those priors for downstream EM classification of each cell's genes into paternal monoallelic, bi-allelic, or maternal monoallelic states.

## When to use

Use when analyzing allele-specific expression (ASE) from scRNA-Seq allele counts (e.g., CAST×B6 F1 embryo data) and you need population-informed priors to overcome low per-cell coverage before assigning P/B/M expression states per gene per cell.

## When NOT to use

- Data are bulk RNA-Seq or already-aggregated allelic proportions — the hierarchical pooling across cells is designed for single-cell sparsity
- Reads cannot be assigned to a phase-known diploid genome (no strain SNP/indel reconstruction), so maternal/paternal counts are unreliable
- The goal is transcriptional burst kinetics inference (burst frequency/size), for which txburst-style ML/PL methods on UMI counts are the appropriate choice, not ASE state classification

## Inputs

- Allele-specific (maternal/paternal) read counts per gene per cell, from a phase-known diploid transcriptome alignment (e.g., bowtie -v 3 --all --best --strata against a g2gtools-reconstructed CAST genome)
- Cell population grouping (e.g., 286 Deng et al. pre-implantation embryo cells across nine developmental stages) used for pooling
- Gene expression filter: allele regarded as expressed if estimated abundance > 1 read (or 1 UMI)

## Outputs

- MCMC-estimated state-specific beta-binomial prior parameters shared across the cell population
- Per-cell, per-gene probabilistic classification into P, B, or M allelic expression states (mixture weights π)
- Gene-stage ASE profile class assignments (P, B, M, PB, BM, MP, PBM) under the πg·s > 0.7 majority threshold
- Partially pooled allelic proportion estimates per gene per cell

## How to apply

First count allele-specific reads via EM-based weighted allocation of multi-reads (EMASE) rather than discarding multi-mappers, since the unique-reads approach inflates sampling zeros and false monoallelic calls. Pool all 286 cells (e.g., the nine Deng et al. pre-implantation stages) and run the MCMC stage of the hierarchical beta-binomial mixture — implemented in Stan and shipped in scBASE (install via 'python setup.py install', then 'import scbase') — to estimate state-specific distribution parameters shared across the population. Fix the MCMC-derived priors and, for each stage subpopulation, run the EM classification assigning each gene in each cell to paternal monoallelic (P), bi-allelic (B), or maternal monoallelic (M) states from the maternal read counts xgk drawn from a mixture of three beta-binomial distributions. Optionally apply partial pooling, estimating allelic proportions as a weighted average of class-specific proportions, which helps most when coverage is low and cell numbers are large. Assign genes to the seven ASE profile classes (P, B, M, PB, BM, MP, PBM) per stage using the majority-state threshold πg·s > 0.7, acknowledging these class boundaries are arbitrary descriptive summaries.

## Related tools

- **scBASE** (Implements the MCMC stage of the hierarchical beta-binomial mixture for state-specific prior estimation and the downstream EM classification; installed from source ('python setup.py install')) — https://github.com/kbchoi-jax/scbase
- **Stan** (Probabilistic programming language used to implement the Bayesian mixture model underlying scBASE)
- **EMASE** (EM-based weighted allocation of multi-mapping reads to produce the allelic read counts that feed prior estimation)

## Examples

```
git clone git://github.com/kbchoi-jax/scbase && cd scbase && python setup.py install; import scbase  # run MCMC on pooled 286 cells, then EM classification with fixed priors
```

## Evaluation signals

- MCMC converges with stable state-specific beta-binomial parameters; algorithmic details (iterations, burn-in) follow the Supplementary Methods since they are not in the main text
- After fixing MCMC priors, EM classification assigns each gene in each cell to exactly one of three states (P/B/M) with mixture weights summing to 1
- Per-stage class assignment is only made when the majority-state threshold πg·s > 0.7 is met; otherwise the gene is left unclassified for that stage
- Pooled/weighted-allocation counts show lower estimate variability and fewer sampling zeros than unique-read-only counts (discarding multi-reads increases variability and spurious monoallelic calls)
- Down-sampling check (e.g., 1% of reads in 122 blastocyst cells) shows mean squared error of estimated allelic proportions is reduced relative to unpooled estimates

## Limitations

- ASE class thresholds (πg·s > 0.7) are acknowledged as arbitrary, limiting reproducibility of gene classification
- MCMC settings (number of iterations, burn-in, prior specifics) are deferred to Supplementary Methods and not stated in the main text
- No data accession IDs are given for the datasets (only platform GPL12112 for Deng et al.), complicating exact input reconstruction
- Discarding multi-mapping reads leads to higher variability in allelic proportion estimates, more sampling zeros, and spurious dynamic/monoallelic findings

## Evidence

- [abstract] Hierarchical pooling rationale: "combining information across cells using a hierarchical mixture model reduces sampling variability without sacrificing cell-to-cell heterogeneity"
- [discussion] EM classification of states: [evidence span withheld — anchor status: not_found]
- [discussion] Class threshold: [evidence span withheld — anchor status: not_found]
- [discussion] Partial pooling impact: "Partial pooling has the biggest impact when read coverage is low and the number of cells is large"
- [readme] scBASE software availability: "We propose a new method, **scBASE**, a "soft" zero-and-one inflated model for cellular allelic proportions in which we tackle data sparsity and variability"
