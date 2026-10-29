---
name: mixture-weight-threshold-assignment
description: Use when per-cell P/B/M posterior classifications from scBASE must be
  summarized into per-gene, per-stage ASE profile classes using mixture-weight thresholds,
  e.g. to quantify the fraction of blastocyst genes in the PBM (dynamic bursting)
  region.
when_to_use_negative:
- Input data lacks per-cell P/B/M state posteriors (e.g. only bulk or population-level
  allelic proportions are available)
- Allele-specific read counts were obtained by discarding multi-mapping reads, which
  the article shows increases sampling zeros and can produce spurious monoallelic/dynamic
  calls that bias class assignment
- The goal is inference of transcriptional burst kinetics parameters (e.g. via txburst)
  rather than descriptive classification of expression states
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3572
tools:
- name: scBASE
  role: Produces the per-cell P/B/M posterior classifications from which the mixture
    weights πg·s are computed for class assignment
  repo: https://github.com/kbchoi-jax/scbase
- name: EMASE
  role: Upstream EM-based weighted allocation of multi-mapping reads to alleles, feeding
    the counts used for P/B/M classification
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:10368f8076e9c1913bd41a81374c64485420a4f39c41e642a1dca9de8acece01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbchoi-jax/scbase
  tools:
  - EMASE
  - scBASE
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# mixture-weight-threshold-assignment

## Summary

Assigns each gene in each developmental stage to one of seven ASE profile classes (P, B, M, PB, BM, MP, PBM) by thresholding the scBASE mixture weights πg·s, i.e. the fraction of cells classified into each allelic expression state by the beta-binomial mixture model.

## When to use

Use when per-cell P/B/M posterior classifications from scBASE must be summarized into per-gene, per-stage ASE profile classes using mixture-weight thresholds, e.g. to quantify the fraction of blastocyst genes in the PBM (dynamic bursting) region.

## When NOT to use

- Input data lacks per-cell P/B/M state posteriors (e.g. only bulk or population-level allelic proportions are available)
- Allele-specific read counts were obtained by discarding multi-mapping reads, which the article shows increases sampling zeros and can produce spurious monoallelic/dynamic calls that bias class assignment
- The goal is inference of transcriptional burst kinetics parameters (e.g. via txburst) rather than descriptive classification of expression states

## Inputs

- Per-cell allelic expression state posterior classifications (P/B/M) from scBASE's beta-binomial mixture model for the 286 Deng et al. CAST×B6 F1 pre-implantation embryo cells
- Per-gene, per-stage mixture weights πg·s (fraction of cells in each ASE state)

## Outputs

- Gene-to-ASE-class assignment table (gene, stage, mixture weights, assigned class P/B/M/PB/BM/MP/PBM), written to ase_class_assignments_286cells.csv
- Fraction of blastocyst-stage genes assigned to the PBM class (expected ≈ 56.9%)

## How to apply

Start from the per-cell allelic expression state posteriors (paternal monoallelic, bi-allelic, maternal monoallelic) computed by scBASE's beta-binomial mixture model for the 286 pre-implantation embryo cells of the Deng et al. CAST×B6 F1 hybrid. For each gene g and stage s, compute the mixture weight πg·s, the fraction of cells assigned to each ASE state. Assign class P, B, or M if πg·s > 0.7 for a single state; assign PB, BM, or MP if a two-state mixture exceeds 0.9; otherwise assign PBM. The article notes these class boundaries are arbitrary but serve as a simple descriptive summary. Applied to blastocyst-stage genes, this yields the reported 56.9% of genes in the PBM region, reflecting mixed mono- and bi-allelic expression consistent with dynamic bursting.

## Related tools

- **scBASE** (Produces the per-cell P/B/M posterior classifications from which the mixture weights πg·s are computed for class assignment) — https://github.com/kbchoi-jax/scbase
- **EMASE** (Upstream EM-based weighted allocation of multi-mapping reads to alleles, feeding the counts used for P/B/M classification)

## Examples

```
$ git clone git://github.com/kbchoi-jax/scbase
```

## Evaluation signals

- The PBM-class fraction among blastocyst genes falls within ~0.55–0.58 of the reported 0.569
- Exemplar gene calls match the paper: Pacs2 = MP, Mtdh = B, Timm23 and Tulp3 in BM/PB regions, Akr1b3 = PBM (expert review)
- Every gene-stage row satisfies the threshold logic: single-state πg·s > 0.7 → P/B/M; two-state mixture > 0.9 → PB/BM/MP; else PBM
- Output CSV contains stage, mixture weights, and assigned class for all classified genes
- Bursting-pattern logOR > 0 for the majority of genes, consistent with the article's reported positive correlation of allelic bursting

## Limitations

- The class boundaries are arbitrary; the article states 'the aim of this classification is to provide a simple descriptive summary', so class calls are descriptive rather than statistically calibrated
- Classification inherits biases from upstream read counting: discarding multi-mapping reads leads to higher variability in allelic proportion estimates and spurious findings of dynamic and monoallelic expression
- Low per-cell sequencing depth limits discernment of expression dynamics, motivating pooling across cells within the mixture model

## Evidence

- [discussion] Class assignment uses mixture-weight thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] Thresholds acknowledged as arbitrary descriptive summary: "The class boundaries are arbitrary but the aim of this classification is to provide a simple descriptive summary"
- [intro] Per-cell state classification is the prerequisite step: "Next, we compute a probabilistic classification of each gene in each cell into paternal monoallelic, bi-allelic, or maternal monoallelic expression states."
- [readme] scBASE mixture model pools information across cells: "We propose a new method, **scBASE**, a “soft” zero-and-one inflated model for cellular allelic proportions in which we tackle data sparsity and variability"
- [discussion] Multi-read handling upstream of classification: "multi-reads are resolved by computing a weighted allocation based on the estimated probability of each alignment"
