---
name: differential-pseudotime-analysis
description: Use when scRNA-seq data from multiple biological samples per condition
  need differential pseudotime analysis of gene expression, cell abundance, or trajectory
  topology, and sample-to-sample variation must be modeled to avoid false discoveries.
when_to_use_negative:
- Single-sample scRNA-seq data with no biological replicates per condition — cross-sample
  variability cannot be modeled (condiments-style one-sample assumption is what Lamian
  is designed to avoid)
- Data from Monocle2, Monocle3, or Slingshot trajectories — Lamian modules 1 and 2
  do not natively support these trajectory types
- Systematic comparison of sample-harmonization methods is needed — this is explicitly
  beyond the article's scope
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Lamian
  role: 'Core framework for differential multi-sample pseudotime analysis: TDE, XDE,
    TCD, XCD, differential topology testing, gene clustering and plotting'
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Default pseudotime construction via cluster-based minimum spanning tree; install
    from GitHub zji90/TSCAN
  repo: https://github.com/Winnie09/Lamian
- name: Seurat
  role: Sample harmonization by embedding cells into a common low-dimensional space
- name: condiments
  role: Competing pseudotime method benchmarked for XDE reproducibility and FDR control
    (assumes one sample per condition)
- name: tradeSeq
  role: Competing differential trajectory/expression method used as benchmark baseline
- name: Monocle
  role: Baseline method adapted into monocle2TrajTestCorr for XDE testing via null/full
    GAM likelihood-ratio comparison
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:f2ea984b7dcf64a71a77c199a1a942a12b22b6059f1df3ec201b7aae66f58de0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - DAseq
  - Lamian
  - Monocle
  - Phenopath
  - Seurat
  - Slingshot
  - TSCAN
  - condiments
  - milo
  - tradeSeq
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# differential-pseudotime-analysis

## Summary

A multi-sample differential pseudotime analysis workflow implemented in Lamian, which constructs pseudotime (TSCAN cluster-based minimum spanning tree), harmonizes samples with Seurat, and tests for topology, cell density (TCD/XCD), and gene expression (TDE/XDE) changes across conditions while accounting for cross-sample variability via permutation-based likelihood ratio tests.

## When to use

Use when scRNA-seq data from multiple biological samples per condition need differential pseudotime analysis of gene expression, cell abundance, or trajectory topology, and sample-to-sample variation must be modeled to avoid false discoveries.

## When NOT to use

- Single-sample scRNA-seq data with no biological replicates per condition — cross-sample variability cannot be modeled (condiments-style one-sample assumption is what Lamian is designed to avoid)
- Data from Monocle2, Monocle3, or Slingshot trajectories — Lamian modules 1 and 2 do not natively support these trajectory types
- Systematic comparison of sample-harmonization methods is needed — this is explicitly beyond the article's scope

## Inputs

- Gene-by-cell expression matrix (e.g., hca_bm_saver example data)
- Cell-to-sample annotation (e.g., hca_bm_cellanno)
- Low-dimensional representation / PCA of cells (e.g., hca_bm_pca)
- Pseudotime vector per cell
- Design matrix with intercept in column 1 and sample-group covariate in column 2 (testvar = 2)

## Outputs

- Per-gene TDE/XDE statistics, p-values, and FDR values with significant genes called at FDR.overall < 0.05
- TCD/XCD test p-values for cell density/proportion changes along pseudotime
- Gene clusters with GO enrichment per cluster
- Trajectory/branch structure with branch uncertainty (detection rate in 1000 bootstrap samples)
- Result tables such as lamian_fdr_sensitivity_benchmark.csv or monocle2_xde_results.csv

## How to apply

Harmonize samples by embedding cells into a common low-dimensional space with Seurat, then construct pseudotime with TSCAN's cluster-based minimum spanning tree, optionally quantifying branch uncertainty via detection rate in 1000 bootstrap samples. Prepare four Lamian inputs: a gene-by-cell expression matrix, cell-to-sample annotation, a pseudotime vector, and a design matrix whose first column is the intercept. Run lamian_test with test.type='time' for TDE and test.type='variable', testvar=2 for XDE using the permutation-based likelihood ratio test (permuiter >= 100), and run cellPropTest with test.type='Time' (TCD) and test.type='Variable' (XCD) after dividing pseudotime into 100 equal-length intervals. Cluster significant genes with clusterGene and compute GO enrichment per cluster, calling TDE/XDE genes at FDR.overall < 0.05. The rationale is that Lamian draws inference after accounting for cross-sample variability, giving conservative FDR control, whereas competitors (condiments, Phenopath, monocle2TrajTestCorr, limma, tradeSeq) underestimate real FDR. Benchmarking and validation can use the COVID-19 CD8 T cell data (E-MTAB-9357), expecting 2195 TDE genes in 5 clusters, 1315 XDE genes in 14 clusters, TCD p < 2.22e-308, and XCD p = 1.38e-61.

## Related tools

- **Lamian** (Core framework for differential multi-sample pseudotime analysis: TDE, XDE, TCD, XCD, differential topology testing, gene clustering and plotting) — https://github.com/Winnie09/Lamian
- **TSCAN** (Default pseudotime construction via cluster-based minimum spanning tree; install from GitHub zji90/TSCAN) — https://github.com/Winnie09/Lamian
- **Seurat** (Sample harmonization by embedding cells into a common low-dimensional space)
- **condiments** (Competing pseudotime method benchmarked for XDE reproducibility and FDR control (assumes one sample per condition))
- **tradeSeq** (Competing differential trajectory/expression method used as benchmark baseline)
- **Monocle** (Baseline method adapted into monocle2TrajTestCorr for XDE testing via null/full GAM likelihood-ratio comparison)

## Examples

```
res <- lamian_test(expr=expdata, cellanno=cellanno, pseudotime=pseudotime, design=design, test.type='variable', testvar=2, permuiter=100)
```

## Evaluation signals

- Lamian's real (empirical) FDR must not exceed its reported FDR at every spike-in signal strength (conservative calibration), and its sensitivity-realFDR AUC must be highest among compared methods
- COVID-19 CD8 T cell reproduction should yield 2195 TDE genes in 5 clusters and 1315 XDE genes in 14 clusters with TBX21/ZEB2/interferon-gene patterns
- COVID severity TCD p-value < 2.22e-308 and XCD p-value = 1.38e-61
- Output tables contain a likelihood-ratio statistic, p-value, and FDR for every gene tested (e.g., all 100 genes in the expdata example)
- Random partition reproducibility: Lamian.pm and Lamian.chisq achieve the highest overlap proportion of XDE genes between two random partitions versus condiments, monocle2TrajTestCorr, limma, and tradeSeq

## Limitations

- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories; tailoring these methods is deferred to future work
- Simultaneously accounting for pseudotime inference uncertainty, gene expression and cell abundance uncertainty, and cross-sample/cross-cell variability remains a future research topic
- The systematic comparison of harmonization methods is beyond the scope of the study; Seurat is used by default
- EM algorithm fitting details are only in Supplementary Notes; TDE/TCD/XCD simulations are in Supplementary Notes

## Evidence

- [discussion] XDE detection using permutation-based likelihood ratio test comparing nested models: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [discussion] Sample harmonization with Seurat: "For sample harmonization, we used Seurat32 to embed cells into a common low-dimensional space."
- [discussion] TSCAN default pseudotime construction: "In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability."
- [discussion] TDE permutation within samples: "In each permutation, we first bootstrap the cells to account for pseudotime variability, and we then permute the pseudotime of the cells within each sample."
- [discussion] Cell density via 100 intervals: "we divide the pseudotime from 0 to its maximum into 100 consecutive intervals of equal lengths."
- [abstract] Cross-sample variability reduces false discoveries: "Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false"
- [discussion] COVID filter thresholds: "We filtered out cells with fewer than 2,000 reads or 500 expressed genes or more than 10% mitochondrial reads."
- [readme] README installation requires updated TSCAN and Lamian from GitHub: [evidence span withheld — anchor status: external_doc]
