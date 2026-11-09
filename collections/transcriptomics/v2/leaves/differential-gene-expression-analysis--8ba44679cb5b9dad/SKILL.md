---
name: differential-gene-expression-analysis
description: Use when single-cell RNA-seq case/control data must be reduced to per-cell-cluster
  DEG profiles before connectivity-based drug repurposing (ASGARD GetDrug/DrugScore),
  or when benchmarking which scRNA-seq DE method maximizes downstream AUC.
when_to_use_negative:
- When the input is already pseudo-bulk or bulk RNA-seq and the goal is to run the
  bulk baselines CLUE or DrInsight rather than per-cluster single-cell DE
- When a cell type has fewer than the minimum of 3 cells in either the case or control
  group — that cluster's DE contrast should be skipped
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: limma
  role: Primary DE method (Bayes model, e.g., typeCase - typeControl contrast) showing
    the best average AUC among the four tested
  repo: https://github.com/lanagarmire/Asgard
- name: Seurat
  role: scRNA-seq preprocessing, normalization, clustering (Louvain, resolution 0.4),
    and its Wilcoxon DE method as a benchmarked alternative
- name: DESeq2
  role: Benchmarked differential expression method for single-cell clusters
- name: edgeR
  role: Benchmarked differential expression method for single-cell clusters
- name: ASGARD
  role: Consumes the per-cluster DEG lists in GetDrug and aggregates them into multi-cluster
    DrugScore rankings
  repo: https://github.com/lanagarmire/ASGARD
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:ab7cefc04f7183da6e7068b10dd402251441a72e7fff16c2f35e10f36d4a7791
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/Asgard
  tools:
  - ASGARD
  - CLUE
  - DESeq2
  - DrInsight
  - Seurat
  - SingleR
  - TRANSACT
  - cmapR
  - edgeR
  - limma
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# differential-gene-expression-analysis

## Summary

Per-cluster/cell-type differential expression analysis of scRNA-seq case vs. control data (e.g., deceased vs. cured COVID-19 BALF, TNBC patients vs. healthy controls) using limma (Bayes), Seurat Wilcoxon, DESeq2, or edgeR, producing the DEG lists that feed ASGARD's GetDrug and DrugScore repurposing steps.

## When to use

Use when single-cell RNA-seq case/control data must be reduced to per-cell-cluster DEG profiles before connectivity-based drug repurposing (ASGARD GetDrug/DrugScore), or when benchmarking which scRNA-seq DE method maximizes downstream AUC.

## When NOT to use

- When the input is already pseudo-bulk or bulk RNA-seq and the goal is to run the bulk baselines CLUE or DrInsight rather than per-cluster single-cell DE
- When a cell type has fewer than the minimum of 3 cells in either the case or control group — that cluster's DE contrast should be skipped

## Inputs

- Processed scRNA-seq expression matrix (e.g., GSE161529 TNBC, GSE145926/GSE158055 COVID-19 BALF) as a Seurat object or gene-by-cell matrix
- Cell metadata with cluster/cell-type annotations (SingleR) and case/control labels
- Case vs. control contrast specification (e.g., typeCase - typeControl for deceased vs. cured patients)
- Per-dataset filter thresholds: min 3 cells per condition, <200 unique genes cell cutoff, COVID-19 6000-gene / 10% mitochondrial cutoffs

## Outputs

- Per-cluster DEG lists (gene symbols with log fold-change, BH-adjusted P values) passed as Gene.list to ASGARD GetDrug
- Comparison tables of DE method performance (limma Bayes vs. Seurat Wilcoxon vs. DESeq2 vs. edgeR) via downstream ROC/AUC computed with pROC

## How to apply

After Seurat preprocessing (remove genes seen in fewer than 3 cells, cells with <200 unique genes; for COVID-19 BALF also >6000 genes or >10% mitochondrial genes, then Louvain clustering at resolution 0.4 and SingleR annotation), compute per-cluster case-vs-control contrasts with limma's Bayes model, keeping only cell types with more than min.cells=3 cells in each group. In ASGARD this DE method choice is a decision point: Seurat Wilcoxon, DESeq2, edgeR, and limma (Bayes) were benchmarked, and limma (Bayes) showed the best average AUC. Apply BH FDR adjustment to P values (e.g., TNBC GSE161529 analyses) and pass the ranked Gene.list per cluster as gene.data to GetDrug with connectivity='negative'. Because single-cell sparsity and heterogeneity cause false DE discoveries, select the DE method per dataset rather than fixing one globally.

## Related tools

- **limma** (Primary DE method (Bayes model, e.g., typeCase - typeControl contrast) showing the best average AUC among the four tested) — https://github.com/lanagarmire/Asgard
- **Seurat** (scRNA-seq preprocessing, normalization, clustering (Louvain, resolution 0.4), and its Wilcoxon DE method as a benchmarked alternative)
- **DESeq2** (Benchmarked differential expression method for single-cell clusters)
- **edgeR** (Benchmarked differential expression method for single-cell clusters)
- **ASGARD** (Consumes the per-cluster DEG lists in GetDrug and aggregates them into multi-cluster DrugScore rankings) — https://github.com/lanagarmire/ASGARD

## Examples

```
fit <- eBayes(lmFit(expr_matrix, design)); degs <- topTable(fit, coef="typeCase-typeControl", adjust="BH", number=Inf)
```

## Evaluation signals

- Only cell types with more than min.cells=3 cells in each case and control group yield DE results; all other clusters are excluded
- Output DEG tables contain BH-adjusted P values and finite statistics for genes passing the fewer-than-3-cells gene filter
- Feeding limma-based DEGs into ASGARD and computing ROC/AUC with pROC should reproduce the reported ordering, with limma (Bayes) showing the best average AUC versus the other three methods
- The multi-cluster DrugScore built from these DEGs should show a significantly better AUC (Student's t-test, P < 0.05) than individual-cluster predictions
- For task_008, the resulting ranking should place rescinnamine at rank 2 and enalapril at rank 4 in covid19_balf_drug_score_ranking.csv

## Limitations

- Sparsity and heterogeneity of single-cell data usually cause false discoveries of differentially expressed genes, so the DE method should be selected per dataset
- AUC performance varies with cell population sizes, proportions, and differential expression levels, requiring robustness simulations
- Obtaining proper normal-tissue controls may not be feasible for some diseases, forcing substitute control samples

## Evidence

- [discussion] Seurat18, DESeq226, edgeR27, and Limma25 are among the top methods in discovering differentially expressed genes using single-cell data: "Seurat18, DESeq226, edgeR27, and Limma25 are among the top methods in discovering differentially expressed genes using single-cell data"
- [discussion] Limma (Bayes) best average AUC: "Limma (Bayes) method showed the best average AUC performance compared to all other three methods"
- [discussion] gene filter threshold: "genes identified in fewer than three cells are removed from the dataset"
- [discussion] sparsity caveat: "Sparsity and heterogeneity are two major challenges in analyzing single-cell data, and usually cause false discoveries of differentially expressed genes"
- [readme] hardware requirement for ASGARD reference pipeline: "Asgard package requires only a standard computer with enough RAM (>64GB) to support the in-memory operations."
