---
name: developmental-timecourse-ase-dynamics
description: Use when analyzing scRNA-Seq allelic read counts from cells sampled along
  a developmental time course (e.g., 286 CAST×B6 F1 pre-implantation embryo cells)
  to quantify how stage-wise ASE patterns change.
when_to_use_negative:
- Input is bulk RNA-Seq or lacks a phase-known diploid transcriptome / F1 hybrid allele
  informativeness — the mixture model requires per-cell allelic counts from a CAST×B6-style
  cross
- Data already have allelic states pre-classified, or the goal is burst-kinetics parameter
  estimation rather than ASE state/trajectory classification (use txburst instead)
- Coverage is so low that allelic counts are essentially absent for the genes of interest
  — the article notes low depth per cell limits discerning expression dynamics, and
  the method pools across cells to compensate
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scBASE
  role: 'Hierarchical beta-binomial mixture model: MCMC estimation of state priors
    on pooled cells, EM classification of P/B/M states per gene per cell, and majority-rule
    assignment to the seven ASE profile classes'
  repo: https://github.com/churchill-lab/scBASE
- name: EMASE
  role: EM-based weighted allocation of multi-mapping reads to produce allele-specific
    counts fed into scBASE
- name: g2gtools
  role: Reconstruction of the CAST genome by incorporating known SNPs and indels into
    the reference genome to build the diploid transcriptome
- name: bowtie v1.0.0
  role: Alignment of reads to the phase-known diploid transcriptome with --all --best
    --strata -v 3
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:70fd23c5b22061ae860d7aa248a779e9076b1513d8ca97e4d72001e08de18c47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/churchill-lab/scBASE
  tools:
  - EMASE
  - bowtie v1.0.0
  - g2gtools
  - scBASE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# developmental-timecourse-ase-dynamics

## Summary

Tracks allele-specific expression (ASE) pattern dynamics across the stages of pre-implantation mouse embryo development by classifying genes into seven ASE profile classes (P, B, M, PB, BM, MP, PBM) with scBASE, including per-gene trajectories such as Akr1b3's M→PBM→P→B transition.

## When to use

Use when analyzing scRNA-Seq allelic read counts from cells sampled along a developmental time course (e.g., 286 CAST×B6 F1 pre-implantation embryo cells) to quantify how stage-wise ASE patterns change.

## When NOT to use

- Input is bulk RNA-Seq or lacks a phase-known diploid transcriptome / F1 hybrid allele informativeness — the mixture model requires per-cell allelic counts from a CAST×B6-style cross
- Data already have allelic states pre-classified, or the goal is burst-kinetics parameter estimation rather than ASE state/trajectory classification (use txburst instead)
- Coverage is so low that allelic counts are essentially absent for the genes of interest — the article notes low depth per cell limits discerning expression dynamics, and the method pools across cells to compensate

## Inputs

- Allele-specific read counts (maternal/paternal) per gene per cell from F1 hybrid scRNA-Seq (e.g., Deng et al. CAST×B6, 286 pre-implantation embryo cells across nine developmental stages)
- Stage/cell-type metadata assigning each of the 286 cells to one of nine developmental stages (zygote through late blastocyst)
- MCMC-derived state-specific beta-binomial priors pooled across all cells

## Outputs

- Per-stage classification of each gene into P, B, or M expression states (EM classification with fixed MCMC priors)
- Per-stage majority-rule ASE profile class assignment (P, B, M, PB, BM, MP, PBM) per gene at threshold πg·s > 0.7
- Stage-wise proportion table of the seven ASE patterns (e.g., figure7a_pattern_proportions.csv)
- Per-gene developmental trajectory of ASE classes (e.g., Akr1b3 M→PBM→P→B plotted in the 3D mixture-weight simplex, figure7b_akr1b3_simplex.png)

## How to apply

Install scBASE from source (clone the kbchoi-jax/scbase repo, 'python setup.py install') and 'import scbase'. First resolve multi-mapping reads via weighted allocation with EMASE (rather than discarding multi-reads, which increases variability and spurious monoallelic calls), then run the MCMC stage of the hierarchical beta-binomial mixture model on all 286 cells pooled across the nine developmental stages to estimate state-specific priors. For each stage subpopulation, run the EM classification with the MCMC priors fixed to probabilistically classify each gene in each cell into paternal monoallelic (P), bi-allelic (B), or maternal monoallelic (M) states. Assign genes to one of the seven ASE profile classes (P, B, M, PB, BM, MP, PBM) per stage using the majority-state threshold πg·s > 0.7. Tabulate stage-wise proportions of the seven allelic patterns and extract per-gene class sequences (e.g., Akr1b3) for trajectory visualization in the mixture-weight simplex.

## Related tools

- **scBASE** (Hierarchical beta-binomial mixture model: MCMC estimation of state priors on pooled cells, EM classification of P/B/M states per gene per cell, and majority-rule assignment to the seven ASE profile classes) — https://github.com/churchill-lab/scBASE
- **EMASE** (EM-based weighted allocation of multi-mapping reads to produce allele-specific counts fed into scBASE)
- **g2gtools** (Reconstruction of the CAST genome by incorporating known SNPs and indels into the reference genome to build the diploid transcriptome)
- **bowtie v1.0.0** (Alignment of reads to the phase-known diploid transcriptome with --all --best --strata -v 3)

## Examples

```
git clone git://github.com/kbchoi-jax/scbase; cd scbase; python setup.py install
```

## Evaluation signals

- Reproduced stage-wise pattern proportions match Fig 7a: maternal-dominated patterns in zygote/early 2-cell, a B peak at late 2-cell/4-cell, and PBM dominance from the 8-cell stage onward
- Akr1b3 follows the reported M→PBM→P→B class sequence across stages, visualizable in the 3D mixture-weight simplex (Fig 7b)
- Every gene receives exactly one of the seven ASE profile classes (P, B, M, PB, BM, MP, PBM) per stage, with majority-state fraction πg·s > 0.7 recorded for class assignments
- Allele-specific counts were derived with multi-read weighted allocation (EMASE), not unique-reads-only counting — the article shows discarding multi-reads inflates sampling zeros and spurious monoallelic/dynamic calls
- Seven developmental stage subpopulations (nine stages, 286 cells total) are each classified against the same pooled MCMC priors, so stage-wise comparisons use a consistent state model

## Limitations

- The ASE class boundaries (πg·s > 0.7) are acknowledged as arbitrary, providing only a simple descriptive summary of gene classification
- MCMC settings (number of iterations, burn-in, prior specifics) are deferred to Supplementary Methods, limiting out-of-the-box reproducibility
- No GEO/SRA accession IDs are given for the Deng et al. dataset (only Illumina HiSeq 2000 platform GPL12112), so input acquisition requires external lookup

## Evidence

- [discussion] If a majority of cells (πg⋅s>0.7) are in a particular ASE state: [evidence span withheld — anchor status: not_found]
- [discussion] Deng et al. 286 cells: [evidence span withheld — anchor status: not_found]
- [abstract] hierarchical pooling rationale: "combining information across cells using a hierarchical mixture model reduces sampling variability without sacrificing cell-to-cell heterogeneity"
- [abstract] multi-read discarding caveat: "discarding multi-mapping reads leads to higher variability in estimates of allelic proportions, an increased frequency of sampling zeros"
- [readme] scBASE method description from README: "a "soft" zero-and-one inflated model for cellular allelic proportions in which we tackle data sparsity and variability by harnessing information derived from cell's population context."
