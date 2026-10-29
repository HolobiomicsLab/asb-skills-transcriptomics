---
name: exemplar-gene-call-reconciliation
description: Use when re-implementing scBASE's seven-region gene classification on
  the Deng et al. CAST×B6 F1 pre-implantation data and you need to validate your mixture-weight
  thresholds and P/B/M posterior assignments against the paper's exemplar gene calls.
when_to_use_negative:
- Input is raw FASTQ or unaligned reads — this skill operates on already-classified
  per-cell P/B/M posteriors, not on alignment or read counting (use EMASE weighted
  allocation first).
- Data are not from a CAST×B6 (or reciprocal) F1 hybrid with a phase-known diploid
  transcriptome — the P/B/M state labels and exemplar calls are allele-parent specific.
- The goal is burst-kinetics inference (k_on, k_off, k_syn) rather than descriptive
  ASE class assignment — txburst, not scBASE class thresholds, is the appropriate
  tool.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scBASE
  role: Produces the per-cell P/B/M posterior classifications and partial-pooled allelic
    proportions that the exemplar gene calls are reconciled against
  repo: https://github.com/kbchoi-jax/scbase
- name: EMASE
  role: Counts allele-specific reads via EM-based weighted allocation of multi-reads,
    feeding the state classification upstream of reconciliation
- name: txburst
  role: 'Non-goal comparator: burst-kinetics inference from allelic counts, distinct
    from the descriptive seven-region classification validated here'
  repo: https://github.com/sandberg-lab/txburst
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:016d158bb00fbce7cdea5d7c7de81d30278123b767dae5cea10c17dfd8e1257a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbchoi-jax/scbase
  tools:
  - EMASE
  - scBASE
  - txburst
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# exemplar-gene-call-reconciliation

## Summary

Reconciles gene-level ASE class assignments (P, B, M, PB, BM, MP, PBM) produced from scBASE posterior state probabilities against the exemplar gene calls reported in Choi et al. 2019 (Nature Communications), verifying that Pacs2, Mtdh, Timm23, Tulp3, and Akr1b3 fall in the same classes and that the blastocyst PBM-region fraction matches the reported 56.9%.

## When to use

Use when re-implementing scBASE's seven-region gene classification on the Deng et al. CAST×B6 F1 pre-implantation data and you need to validate your mixture-weight thresholds and P/B/M posterior assignments against the paper's exemplar gene calls.

## When NOT to use

- Input is raw FASTQ or unaligned reads — this skill operates on already-classified per-cell P/B/M posteriors, not on alignment or read counting (use EMASE weighted allocation first).
- Data are not from a CAST×B6 (or reciprocal) F1 hybrid with a phase-known diploid transcriptome — the P/B/M state labels and exemplar calls are allele-parent specific.
- The goal is burst-kinetics inference (k_on, k_off, k_syn) rather than descriptive ASE class assignment — txburst, not scBASE class thresholds, is the appropriate tool.

## Inputs

- Per-cell allelic expression state posterior probabilities (P/B/M) from scBASE's beta-binomial mixture model for the 286 Deng et al. CAST×B6 F1 pre-implantation embryo cells
- Per-gene, per-stage mixture weights πg.s (fraction of cells assigned to each ASE state)
- Exemplar gene identities from the paper: Pacs2, Mtdh, Timm23, Tulp3, Akr1b3
- Developmental stage annotation per cell (e.g. early, mid, late blastocyst)

## Outputs

- Gene-to-ASE-class assignment table with stage, mixture weights, and assigned class (written to ase_class_assignments_286cells.csv)
- Reconciled exemplar gene calls (Pacs2=MP, Mtdh=B, Timm23 and Tulp3 in BM/PB, Akr1b3=PBM)
- Blastocyst-stage PBM-class fraction (target ≈ 0.569)

## How to apply

After computing per-gene, per-stage mixture weights πg.s from the beta-binomial mixture classification of 286 CAST×B6 F1 pre-implantation embryo cells, apply the paper's class boundaries: assign a single-state class (P, B, or M) when πg.s > 0.7, a two-state class (PB, BM, or MP) when a two-state mixture exceeds 0.9, and otherwise assign PBM. Then look up the exemplar genes in the paper and check your calls: Pacs2 should be MP, Mtdh should be B, Timm23 and Tulp3 should sit in the BM/PB regions, and Akr1b3 should be PBM. Finally, restrict to blastocyst-stage genes, compute the fraction assigned to the PBM class, and confirm it falls within roughly 0.55–0.58 of the reported 0.569. Discrepancies indicate the state-classification step (e.g. read counting by EMASE weighted allocation vs. unique-reads) or the threshold application diverges from the original pipeline, so trace the mismatch back to the per-cell P/B/M posteriors before adjusting thresholds.

## Related tools

- **scBASE** (Produces the per-cell P/B/M posterior classifications and partial-pooled allelic proportions that the exemplar gene calls are reconciled against) — https://github.com/kbchoi-jax/scbase
- **EMASE** (Counts allele-specific reads via EM-based weighted allocation of multi-reads, feeding the state classification upstream of reconciliation)
- **txburst** (Non-goal comparator: burst-kinetics inference from allelic counts, distinct from the descriptive seven-region classification validated here) — https://github.com/sandberg-lab/txburst

## Examples

```
git clone git://github.com/kbchoi-jax/scbase  # then classify genes with scBASE posteriors and write ase_class_assignments_286cells.csv for blastocyst stages
```

## Evaluation signals

- Pacs2 assigns to class MP, Mtdh to class B, and Akr1b3 to class PBM under your thresholds (expert review against paper Fig. exemplar calls).
- Timm23 and Tulp3 fall within the BM or PB two-state mixture regions (π of a two-state mixture > 0.9 but no single state > 0.7).
- The fraction of blastocyst-stage genes assigned to PBM falls within ~0.55–0.58 of the reported 0.569.
- Every gene in the output table receives exactly one of the seven classes {P, B, M, PB, BM, MP, PBM}, and class assignment follows the stated π thresholds without ties or unassigned rows.
- Mixture weights in the CSV sum to 1.0 for every gene-stage pair and recompute correctly from the per-cell posterior state counts.

## Limitations

- The class boundaries are arbitrary by the authors' own statement, limiting reproducibility of the gene classification; exemplar reconciliation guards against threshold drift but cannot recover a uniquely 'correct' classing.
- Discarding multi-mapping reads (unique-reads counting) leads to higher variability in allelic proportion estimates and spurious monoallelic calls, which can flip borderline exemplar genes between classes.
- No GEO/SRA accession IDs are given for the Deng et al. 286-cell dataset (only platform GPL12112), so input provenance must be established independently before reconciliation.

## Evidence

- [discussion] Class thresholds applied to mixture weights: [evidence span withheld — anchor status: not_found]
- [discussion] Probabilistic state classification basis: [evidence span withheld — anchor status: not_found]
- [discussion] Arbitrary class boundaries caveat: "The class boundaries are arbitrary but the aim of this classification is to provide a simple descriptive summary"
- [discussion] Dataset description: [evidence span withheld — anchor status: not_found]
- [readme] scBASE repository and purpose: "We propose a new method, **scBASE**, a “soft” zero-and-one inflated model for cellular allelic proportions"
