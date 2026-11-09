---
name: blastocyst-stage-fraction-computation
description: Use when scBASE per-cell P/B/M posteriors are available for the Deng
  et al. CAST×B6 F1 pre-implantation data (286 cells) and you need per-gene, per-stage
  ASE class assignments or the PBM-region fraction at blastocyst.
when_to_use_negative:
- Input is raw allele-specific read counts rather than per-cell P/B/M posterior classifications
  — run scBASE's classification and partial pooling steps first
- Data are from a non-F1-hybrid cross or a dataset lacking allele-paired coverage,
  since paternal (P) vs maternal (M) state assignment is undefined
- Goal is burst-kinetics parameter inference (burst frequency/size) rather than categorical
  ASE class fractions — use the sandberg-lab/txburst tools instead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scBASE
  role: Produces the per-cell P/B/M posterior classifications via a beta-binomial
    mixture model; the resulting mixture weights πg.s are thresholded to assign genes
    to ASE profile classes
  repo: https://github.com/kbchoi-jax/scbase
- name: EMASE
  role: Upstream EM-based weighted allocation of multi-mapping reads to alleles, providing
    the allele-specific counts scBASE classifies
- name: g2gtools
  role: Reconstructs the CAST genome by incorporating SNPs/indels into the reference
    for diploid alignment upstream of counting
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:0acace3e78a91e4eae16455a9bd036f08df2066810c4ff78d07b6c50a6e5ddee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbchoi-jax/scbase
  tools:
  - EMASE
  - g2gtools
  - scBASE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# blastocyst-stage-fraction-computation

## Summary

Classify genes from scBASE's beta-binomial mixture posteriors into the seven ASE profile classes (P, B, M, PB, BM, MP, PBM) per developmental stage, then compute the fraction of blastocyst-stage genes assigned to the PBM class, reported as 56.9%.

## When to use

Use when scBASE per-cell P/B/M posteriors are available for the Deng et al. CAST×B6 F1 pre-implantation data (286 cells) and you need per-gene, per-stage ASE class assignments or the PBM-region fraction at blastocyst.

## When NOT to use

- Input is raw allele-specific read counts rather than per-cell P/B/M posterior classifications — run scBASE's classification and partial pooling steps first
- Data are from a non-F1-hybrid cross or a dataset lacking allele-paired coverage, since paternal (P) vs maternal (M) state assignment is undefined
- Goal is burst-kinetics parameter inference (burst frequency/size) rather than categorical ASE class fractions — use the sandberg-lab/txburst tools instead

## Inputs

- Per-cell allelic expression state posterior classifications (P/B/M) from scBASE for the 286 Deng et al. CAST×B6 F1 pre-implantation embryo cells
- Gene and developmental-stage annotations for the pre-implantation time course

## Outputs

- Gene-to-ASE-class assignment table with stage, mixture weights πg.s, and assigned class (written to ase_class_assignments_286cells.csv)
- Fraction of blastocyst-stage genes in the PBM class (expected ≈ 0.569)

## How to apply

Load the per-cell allelic expression state classifications (P/B/M posteriors) from scBASE's beta-binomial mixture model fitted to the 286 CAST×B6 F1 pre-implantation embryo cells. For each gene g and stage s, compute the mixture weight πg.s, i.e. the fraction of cells assigned to each ASE state. Apply the paper's thresholds: if πg.s > 0.7 for a single state assign that class (P, B, or M); if a two-state mixture exceeds 0.9 assign PB, BM, or MP; otherwise assign PBM. Restrict to blastocyst-stage genes (early, mid, late; 122 cells were used in the down-sampling assessment) and compute the fraction in the PBM class. Verify exemplar calls — Pacs2 = MP, Mtdh = B, Timm23 and Tulp3 in BM/PB, Akr1b3 = PBM — against the paper before trusting the table.

## Related tools

- **scBASE** (Produces the per-cell P/B/M posterior classifications via a beta-binomial mixture model; the resulting mixture weights πg.s are thresholded to assign genes to ASE profile classes) — https://github.com/kbchoi-jax/scbase
- **EMASE** (Upstream EM-based weighted allocation of multi-mapping reads to alleles, providing the allele-specific counts scBASE classifies)
- **g2gtools** (Reconstructs the CAST genome by incorporating SNPs/indels into the reference for diploid alignment upstream of counting)

## Examples

```
import pandas as pd; df = pd.read_csv('scbase_posteriors.csv'); pi = df.groupby(['gene','stage','state'])['assignment'].mean().unstack(); cls = pi.assign(PBM=pi.max(axis=1)<0.7).pipe(lambda d: d.idxmax(axis=1)); blast = df[df.stage.str.contains('blastocyst')].gene.nunique(); (cls['PBM'].loc[blast]/blast)
```

## Evaluation signals

- PBM-class fraction among blastocyst-stage genes must fall within ~0.55–0.58 of the reported 0.569
- Exemplar gene calls must match the paper: Pacs2 = MP, Mtdh = B, Timm23 and Tulp3 in BM/PB regions, Akr1b3 = PBM
- Every gene is assigned exactly one of the seven classes P, B, M, PB, BM, MP, PBM — no unclassified genes
- Mixture weights πg.s computed per gene per stage must sum to 1 across ASE states
- Output table ase_class_assignments_286cells.csv must contain gene, stage, mixture weights, and assigned class columns

## Limitations

- The class boundaries (πg.s > 0.7 single-state; > 0.9 two-state) are arbitrary and acknowledged as limiting reproducibility of the classification
- Classification quality depends on coverage; discarding multi-mapping reads raises variability in allelic proportion estimates and can spuriously increase apparent monoallelic expression, so use EMASE weighted allocation
- Low sequencing depth per cell is why mixture weights are computed across cells per stage rather than per cell

## Evidence

- [discussion] Classification of genes into ASE profile classes (P, B, M, PB, BM, MP, PBM) using mixture weight thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] If a majority of cells (πg⋅s>0.7) are in a particular ASE state... then we will assign the gene to the class: [evidence span withheld — anchor status: not_found]
- [discussion] Deng et al.8 sampled 286 pre-implantation embryo cells from an F1 hybrid of CAST×B6 along the stages of prenatal development: [evidence span withheld — anchor status: not_found]
- [discussion] The class boundaries are arbitrary but the aim of this classification is to provide a simple descriptive summary: "The class boundaries are arbitrary but the aim of this classification is to provide a simple descriptive summary"
