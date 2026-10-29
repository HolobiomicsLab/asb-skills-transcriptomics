---
name: gene-ase-profile-class-verification
description: Use when per-cell allelic state posteriors (P/B/M) from a beta-binomial
  mixture on F1-hybrid scRNA-Seq cells are available and genes must be assigned to
  seven ASE profile classes and validated.
when_to_use_negative:
- The input is raw allele-specific read counts rather than per-cell P/B/M posterior
  state classifications — the state classification must be run first via the scBASE
  beta-binomial mixture model.
- The data are not from a phase-known F1 hybrid with distinguishable alleles (e.g.,
  no CAST×B6 cross or diploid transcriptome alignment), so paternal/maternal monoallelic
  states cannot be defined.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: scBASE
  role: Produces the per-cell P/B/M probabilistic classifications via a beta-binomial
    mixture model; its output posterior states are aggregated into gene-level ASE
    profile classes
  repo: https://github.com/churchill-lab/scBASE
- name: EMASE
  role: EM-based weighted allocation of multi-mapping reads to alleles, upstream of
    the state classification that feeds class assignment
- name: g2gtools
  role: Reconstructs the CAST genome by incorporating SNPs and indels to build the
    phase-known diploid transcriptome used for alignment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:0a644d18334c46006d0457000da8c1f2a3279e01b79ae07576352100d5aa4a46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/churchill-lab/scBASE
  tools:
  - EMASE
  - g2gtools
  - scBASE
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-ase-profile-class-verification

## Summary

Classifies genes into scBASE's seven ASE profile classes (P, B, M, PB, BM, MP, PBM) from per-cell P/B/M posterior state assignments, then verifies exemplar gene calls and the blastocyst PBM fraction (56.9%) against the paper.

## When to use

Use when per-cell allelic state posteriors (P/B/M) from a beta-binomial mixture on F1-hybrid scRNA-Seq cells are available and genes must be assigned to seven ASE profile classes and validated.

## When NOT to use

- The input is raw allele-specific read counts rather than per-cell P/B/M posterior state classifications — the state classification must be run first via the scBASE beta-binomial mixture model.
- The data are not from a phase-known F1 hybrid with distinguishable alleles (e.g., no CAST×B6 cross or diploid transcriptome alignment), so paternal/maternal monoallelic states cannot be defined.

## Inputs

- Per-cell allelic expression state posteriors (P/B/M) from scBASE's beta-binomial mixture model for 286 Deng et al. CAST×B6 F1 pre-implantation embryo cells
- Gene and developmental-stage annotations for the pre-implantation time course

## Outputs

- Gene-to-ASE-class assignment table (gene, stage, mixture weights πg.s, assigned class among P, B, M, PB, BM, MP, PBM) written to ase_class_assignments_286cells.csv
- Fraction of blastocyst-stage genes in the PBM class (expected ≈ 56.9%)

## How to apply

Load scBASE per-cell posterior classifications for the 286 Deng et al. CAST×B6 F1 pre-implantation embryo cells, then for each gene g and stage s compute mixture weights πg.s (fraction of cells in each ASE state). Apply the paper's thresholds: if πg.s > 0.7 for a single state assign P, B, or M; if a two-state mixture exceeds 0.9 assign PB, BM, or MP; otherwise assign PBM. Verify exemplar genes against the published calls (Pacs2 = MP, Mtdh = B, Timm23 and Tulp3 in BM/PB, Akr1b3 = PBM). Finally compute the PBM fraction among blastocyst-stage genes and write the full gene-to-class table with stage, mixture weights, and assigned class to ase_class_assignments_286cells.csv.

## Related tools

- **scBASE** (Produces the per-cell P/B/M probabilistic classifications via a beta-binomial mixture model; its output posterior states are aggregated into gene-level ASE profile classes) — https://github.com/churchill-lab/scBASE
- **EMASE** (EM-based weighted allocation of multi-mapping reads to alleles, upstream of the state classification that feeds class assignment)
- **g2gtools** (Reconstructs the CAST genome by incorporating SNPs and indels to build the phase-known diploid transcriptome used for alignment)

## Examples

```
git clone git://github.com/kbchoi-jax/scbase  # then fit per-cell P/B/M posteriors, aggregate πg.s per gene/stage, assign classes, and write ase_class_assignments_286cells.csv
```

## Evaluation signals

- PBM-class fraction among blastocyst-stage genes falls within ~0.55–0.58 of the reported 0.569.
- Exemplar gene assignments match the paper's calls: Pacs2 = MP, Mtdh = B, Timm23 and Tulp3 in BM/PB regions, Akr1b3 = PBM.
- Every gene is assigned exactly one class from {P, B, M, PB, BM, MP, PBM}, consistent with the threshold logic (πg.s > 0.7 single-state; two-state mixture > 0.9; else PBM).
- Output CSV ase_class_assignments_286cells.csv contains gene, stage, mixture weights, and assigned class for all genes across the 286-cell time course.

## Limitations

- The class boundaries (0.7 single-state majority, 0.9 two-state mixture) are acknowledged as arbitrary; the classification is a simple descriptive summary, limiting reproducibility.
- The paper provides no GEO/SRA accession IDs for the Deng et al. 286-cell dataset (only platform GPL12112), so the input data must be sourced independently.
- MCMC settings (iterations, burn-in, priors) for the underlying scBASE fit are deferred to Supplementary Methods, not given in the main text.

## Evidence

- [discussion] Class thresholds applied to mixture weights: [evidence span withheld — anchor status: not_found]
- [discussion] The class boundaries are arbitrary but the aim of this classification is to provide a simple descriptive summary: "The class boundaries are arbitrary but the aim of this classification is to provide a simple descriptive summary"
- [discussion] Deng et al. dataset description: [evidence span withheld — anchor status: not_found]
- [discussion] scBASE probabilistic classification basis: [evidence span withheld — anchor status: not_found]
- [readme] scBASE repository from README: "a "soft" zero-and-one inflated model for cellular allelic proportions in which we tackle data sparsity and variability by harnessing information derived from cell's population context."
