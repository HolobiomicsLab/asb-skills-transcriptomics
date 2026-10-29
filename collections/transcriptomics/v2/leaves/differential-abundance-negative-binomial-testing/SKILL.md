---
name: differential-abundance-negative-binomial-testing
description: Use when scRNA-Seq cells have been clustered into communities (Louvain
  or EVA) and you must test whether the abundance of specific cell-type/cluster labels
  changes between biological conditions while accounting for per-sample replicate
  variability.
when_to_use_negative:
- Cells have not been partitioned into communities or cell-state labels yet — the
  NB regression needs a discrete varDA cell-state label; run EVA/Louvain clustering
  and ProcessEVA first
- The dataset lacks replicate/sample structure (only one sample per condition) — the
  model's varREP (mouse/patient replicate) term is required to separate biological
  inter-subject variability from DA
- The goal is differential gene expression within a cell type rather than shifts in
  cell-type composition/abundance
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ELVAR
  role: R package whose DoDA function performs the negative-binomial-regression DA
    test (varDA = cell-state label, varREP = replicate); ProcessEVA supplies the attribute-enriched
    clusters fed into DoDA
  repo: https://github.com/aet21/ELVAR
- name: Seurat
  role: Provides the Seurat object (counts, metadata, cell attributes) passed to DoDA;
    upstream FindVariableFeatures/ScaleData/RunPCA/FindNeighbors prepare the state-manifold
- name: R
  role: Language in which the whole DA-testing pipeline (cell-attribute-aware clustering
    and NBR steps) is implemented
  repo: https://github.com/aet21/ELVAR
- name: MASS
  role: R package used (per task workflows) for fitting the negative binomial regression
    models underlying DoDA statistics
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:a8327335a2f683bcfac68039eecca15c4eceb97988e271926ea4580b9175ad84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - EVA
  - MASS
  - Matrix
  - R
  - Seurat
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# differential-abundance-negative-binomial-testing

## Summary

ELVAR's differential abundance (DA) testing step: after attribute-aware (EVA) or Louvain community detection identifies attribute-enriched cell-state clusters, negative binomial regression (DoDA) tests whether the frequency of these clusters shifts with condition (e.g. age, Covid-19 severity), regressing out replicate/sample effects and returning Wald and LRT statistics.

## When to use

Use when scRNA-Seq cells have been clustered into communities (Louvain or EVA) and you must test whether the abundance of specific cell-type/cluster labels changes between biological conditions while accounting for per-sample replicate variability.

## When NOT to use

- Cells have not been partitioned into communities or cell-state labels yet — the NB regression needs a discrete varDA cell-state label; run EVA/Louvain clustering and ProcessEVA first
- The dataset lacks replicate/sample structure (only one sample per condition) — the model's varREP (mouse/patient replicate) term is required to separate biological inter-subject variability from DA
- The goal is differential gene expression within a cell type rather than shifts in cell-type composition/abundance

## Inputs

- Seurat object with scRNA-Seq counts, cell metadata (condition/attribute, replicate/sample ID) and cluster/cell-state labels
- Set of significant attribute-enriched clusters from ProcessEVA (Bonferroni-corrected binomial enrichment tests)
- Simulated or real multi-sample scRNA-Seq data (e.g. Tabula Muris Senis lung CD4+ T-cells, Chua nasopharyngeal Covid-19 swabs, colon cancer datasets) with replicate structure (mouse/patient)

## Outputs

- Wald and LRT statistics with P-values from the negative binomial regression per cell type (DoDA object)
- Distributions of DA-test P-values across simulation replicates per method (ELVAR, LVdet, LVnonseq, DA-seq, Milo)
- One-sided Wilcoxon rank-sum P-values comparing ELVAR significance against baselines (e.g. fig2e_wilcoxon_pvalues.csv)
- Fraction of condition-enriched cells captured per method for sensitivity benchmarking

## How to apply

First obtain attribute-enriched clusters via ProcessEVA (binomial enrichment tests with Bonferroni correction on the vertex attribute, e.g. 'age' or Covid-19 severity), merging enriched clusters per condition group. Then call DoDA on the Seurat object and significant-cluster set, specifying the DA variable (varDA, e.g. a binary naive/mature cell-state label or disease severity) and the replicate variable (varREP, e.g. 'mouse' or patient sample) so the negative binomial model treats each mouse/patient as an independent replicate. DoDA returns Wald and likelihood-ratio-test (LRT) statistics per cell type. To benchmark clustering strategies (ELVAR/EVA vs LVdet or LVnonseq Louvain), run the same DoDA step across ~100 simulation replicates and compare the resulting distributions of NB-regression P-values with one-sided Wilcoxon rank-sum tests; smaller P-values indicate stronger DA signal (as in Fig. 2e and Fig. 5 comparisons vs DA-seq and Milo, which also report the fraction of condition-enriched cells captured).

## Related tools

- **ELVAR** (R package whose DoDA function performs the negative-binomial-regression DA test (varDA = cell-state label, varREP = replicate); ProcessEVA supplies the attribute-enriched clusters fed into DoDA) — https://github.com/aet21/ELVAR
- **Seurat** (Provides the Seurat object (counts, metadata, cell attributes) passed to DoDA; upstream FindVariableFeatures/ScaleData/RunPCA/FindNeighbors prepare the state-manifold)
- **R** (Language in which the whole DA-testing pipeline (cell-attribute-aware clustering and NBR steps) is implemented) — https://github.com/aet21/ELVAR
- **MASS** (R package used (per task workflows) for fitting the negative binomial regression models underlying DoDA statistics)

## Examples

```
da.o <- DoDA(seu.o, sigcl.o, varDA="naive", varREP="mouse");
```

## Evaluation signals

- DoDA returns both Wald and LRT statistics per cell type; check both are finite and concordant in direction of effect
- On the simulated 993-cell multi-mouse age-shift data (perturbation 0.25 in young vs 0.5 in old), ELVAR's DA P-value distribution must be significantly smaller than LVdet and LVnonseq by one-sided Wilcoxon rank-sum tests (Fig. 2e)
- On real data (e.g. Chua nasopharyngeal Covid-19), ELVAR must reproduce significant Wald/LRT increase in neutrophil and decrease in moDC abundance in critical vs moderate cases (Supplementary Fig. S5)
- Validation passes only if the replicate variable (varREP = mouse/patient) is correctly specified; DA significance must hold across replicates, not within a single sample
- When benchmarking vs DA-seq/Milo, ELVAR should show larger z-statistics and higher fraction of condition-enriched cells captured (Fig. 5a–i)

## Limitations

- Naive DA analyses can be confounded by biological inter-subject variability and technical batch effects; the NBR replicate term mitigates this but ELVAR itself is sensitive to false positives in the clustering attribute (sample/phenotype label), which can degrade community quality
- Benchmark comparisons were made against Milo and DA-seq but not miloDE; the authors flag this as an open question
- ELVAR/DoDA has increased computational complexity and runtimes compared to Milo and the Louvain-analog, though deemed not practically prohibitive

## Evidence

- [methods] DoDA invocation: "da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");"
- [methods] ProcessEVA extraction of enriched clusters feeding DoDA: "sigcl.o <- ProcessEVA(eva.o,seu.o,attrName="age");"
- [abstract] NB regression framing: "Here we present a differential abundance testing paradigm called ELVAR that uses cell attribute aware clustering when inferring differentially enriched communities within the single-cell manifold."
- [results] Benchmark result supporting Wilcoxon comparison of DA P-values: [evidence span withheld — anchor status: not_found]
- [intro] Confounding motivation for replicate-aware testing: "biological inter-subject variability, as well as technical batch effects, can potentially confound naïve DA-analyses"
- [readme] Package scope from README: "ELVAR is an R-package for differential abundance (DA) testing of cell-types in single-cell RNA-Seq data."
