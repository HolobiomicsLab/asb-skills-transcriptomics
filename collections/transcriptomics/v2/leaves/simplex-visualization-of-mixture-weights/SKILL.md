---
name: simplex-visualization-of-mixture-weights
description: Use when gene-level scBASE mixture weights (πg·s proportions of the seven
  ASE pattern classes P, B, M, PB, BM, MP, PBM) must be plotted per developmental
  stage, e.g. to show a gene's trajectory from maternal monoallelic to bi-allelic
  expression.
when_to_use_negative:
- Data are bulk (not single-cell) allelic counts — the per-cell mixture-weight simplex
  interpretation relies on the scBASE hierarchical beta-binomial mixture over cells
- Allelic counts derive from the unique-reads method that discards multi-mapping reads,
  whose higher variability and spurious monoallelic calls would distort the plotted
  proportions
- The goal is burst-kinetics parameter inference rather than visualization of ASE
  state composition — txburst (k_on, k_off, k_syn) is the wrong tool for this plotting
  skill
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scBASE
  role: Produces the per-cell P/B/M classifications and per-gene/stage mixture weights
    from which the simplex trajectory is constructed; installable via 'python setup.py
    install' and imported with 'import scbase'
  repo: https://github.com/kbchoi-jax/scbase
- name: EMASE
  role: EM-based weighted allocation of multi-mapping reads, generating the allele-specific
    counts that feed the scBASE mixture model upstream of the simplex plot
- name: g2gtools
  role: Reconstructs the CAST genome by incorporating known SNPs/indels to build the
    phase-known diploid transcriptome for alignment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:11efe6ec41cfe657fba9185522ad999129718fe8fc583a93107eaae3f0866612
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbchoi-jax/scbase
  tools:
  - EMASE
  - g2gtools
  - scBASE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# simplex-visualization-of-mixture-weights

## Summary

Visualization of scBASE gene-level mixture-weight trajectories (class proportions P/B/M, PB, BM, MP, PBM across stages) as points and paths inside the 3D mixture-weight simplex, used to depict stage-wise ASE pattern dynamics such as the Akr1b3 M→PBM→P→B transition in Deng et al. pre-implantation mouse embryo cells.

## When to use

Use when gene-level scBASE mixture weights (πg·s proportions of the seven ASE pattern classes P, B, M, PB, BM, MP, PBM) must be plotted per developmental stage, e.g. to show a gene's trajectory from maternal monoallelic to bi-allelic expression.

## When NOT to use

- Data are bulk (not single-cell) allelic counts — the per-cell mixture-weight simplex interpretation relies on the scBASE hierarchical beta-binomial mixture over cells
- Allelic counts derive from the unique-reads method that discards multi-mapping reads, whose higher variability and spurious monoallelic calls would distort the plotted proportions
- The goal is burst-kinetics parameter inference rather than visualization of ASE state composition — txburst (k_on, k_off, k_syn) is the wrong tool for this plotting skill

## Inputs

- Per-stage gene-level mixture weight/proportion matrix from scBASE (πg·s for the seven ASE classes P, B, M, PB, BM, MP, PBM)
- Stage labels for the 286 Deng et al. pre-implantation embryo cells (zygote through late blastocyst)
- Per-cell gene classification into paternal monoallelic (P), bi-allelic (B), or maternal monoallelic (M) states from the beta-binomial mixture EM step

## Outputs

- Stage-wise ASE pattern proportion table (e.g. figure7a_pattern_proportions.csv)
- Simplex trajectory plot of a gene's mixture-weight path across stages (e.g. figure7b_akr1b3_simplex.png) showing the M→PBM→P→B sequence

## How to apply

After running scBASE's MCMC on all 286 CAST×B6 cells and EM classification per stage, assign each gene to one of the seven ASE profile classes per stage using the majority-state threshold πg·s > 0.7. Tabulate the stage-wise proportions of the seven classes (e.g. to figure7a_pattern_proportions.csv). For a gene of interest such as Akr1b3, extract its class sequence across the nine developmental stages and plot its trajectory (M→PBM→P→B for Akr1b3, from maternal monoallelic at zygote/early 2-cell to bi-allelic at the late blastocyst) as a path in the 3D mixture-weight simplex (vertices corresponding to the three component states). The simplex representation makes the switching dynamics between monoallelic and bi-allelic states directly interpretable, mirroring Figure 7b of the article.

## Related tools

- **scBASE** (Produces the per-cell P/B/M classifications and per-gene/stage mixture weights from which the simplex trajectory is constructed; installable via 'python setup.py install' and imported with 'import scbase') — https://github.com/kbchoi-jax/scbase
- **EMASE** (EM-based weighted allocation of multi-mapping reads, generating the allele-specific counts that feed the scBASE mixture model upstream of the simplex plot)
- **g2gtools** (Reconstructs the CAST genome by incorporating known SNPs/indels to build the phase-known diploid transcriptome for alignment)

## Examples

```
$ git clone git://github.com/kbchoi-jax/scbase; cd scbase; python setup.py install  # then import scbase, classify stages, and plot the Akr1b3 M→PBM→P→B trajectory in the 3D mixture-weight simplex
```

## Evaluation signals

- Reproduced stage-wise pattern proportions match Fig 7a: maternal-dominated patterns in zygote/early 2-cell, a B peak at late 2-cell/4-cell, and PBM dominance from the 8-cell stage onward
- The Akr1b3 trajectory in the simplex reproduces the reported M→PBM→P→B sequence, reaching bi-allelic (B) by the late blastocyst stage
- Class assignments respect the threshold rule: a gene is assigned to a class only if a majority of cells (πg·s > 0.7) are in a particular ASE state
- Simplex coordinates for each stage are non-negative and sum to 1 across the plotted mixture-weight components
- Output files figure7a_pattern_proportions.csv and figure7b_akr1b3_simplex.png exist and contain seven ASE classes per stage

## Limitations

- The ASE class boundaries (πg·s > 0.7) are arbitrary and acknowledged as a simple descriptive summary, limiting reproducibility of gene classification
- The visualization reflects mixture-weight dynamics derived from pooled MCMC priors and per-stage EM; if multi-mapping reads are discarded, spurious monoallelic and dynamic calls can distort the trajectory
- No data accession IDs (only platform GPL12112 for Deng et al.) or MCMC settings (iterations, burn-in, priors) are given in the main text, deferring reproducibility details to Supplementary Methods

## Evidence

- [discussion] Classification of genes into ASE profile classes (P, B, M, PB, BM, MP, PBM) using mixture weight thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] Dataset for the developmental time course: [evidence span withheld — anchor status: not_found]
- [intro] Probabilistic classification step underlying the mixture weights: "Next, we compute a probabilistic classification of each gene in each cell into paternal monoallelic, bi-allelic, or maternal monoallelic expression states."
- [readme] Software provenance: "We propose a new method, **scBASE**, a “soft” zero-and-one inflated model for cellular allelic proportions"
