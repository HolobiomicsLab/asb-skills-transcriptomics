---
name: hypergeometric-gene-set-enrichment
description: Use when you have DE genes from clustering corrected scRNA-Seq data (e.g.,
  2-means clustering at FDR 0.01) and a known signature gene set, and need enrichment
  statistics to judge biological meaningfulness.
when_to_use_negative:
- When the goal is general GO/pathway enrichment of a ranked gene list rather than
  testing one specific curated signature against DE genes from cell clustering
- When clustering the cells has not produced a binary split (or discrete groups) defining
  a DE comparison — the 2-means/t-test FDR 0.01 DE step is a prerequisite
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ccRemover
  role: Produces the corrected matrix (cutoff=3, max_it=4, nboot=200, ntop=10) whose
    DE genes are tested for TH2 signature enrichment; expected OR 3.439
  repo: cran/ccRemover
- name: scLVM
  role: Baseline correction method (default K=1 and scree-plot K) whose DE genes yield
    a comparison OR of 2.382
- name: org.Mm.eg.db
  role: Mouse annotation database used to retrieve the cell-cycle gene set that defines
    control genes for the corrected matrices feeding the enrichment test
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:ea06052de1ae73cd8ddcee970b61089cdcb072276aa7812431f8a5b61789ab7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Python
  - ccRemover
  - org.Mm.eg.db
  - scLVM
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hypergeometric-gene-set-enrichment

## Summary

Tests whether a curated gene signature (e.g., the 122 TH2 signature genes) is over-represented among differentially expressed genes identified from clustering single-cell RNA-Seq data, yielding odds ratios, hypergeometric p-values, and 95% confidence intervals used to compare correction methods such as ccRemover and scLVM.

## When to use

Use when you have DE genes from clustering corrected scRNA-Seq data (e.g., 2-means clustering at FDR 0.01) and a known signature gene set, and need enrichment statistics to judge biological meaningfulness.

## When NOT to use

- When the goal is general GO/pathway enrichment of a ranked gene list rather than testing one specific curated signature against DE genes from cell clustering
- When clustering the cells has not produced a binary split (or discrete groups) defining a DE comparison — the 2-means/t-test FDR 0.01 DE step is a prerequisite

## Inputs

- Corrected or original gene expression matrix (e.g., 81 cells x 14,147 genes, genes as rows, gene-wise centered)
- DE gene list from 2-means clustering with t-test at FDR 0.01
- Curated signature gene set (e.g., 122 TH2 signature genes, mouse symbols)
- Background gene universe size (all genes tested for differential expression)

## Outputs

- Odds ratio for signature-gene enrichment per correction method
- Hypergeometric p-value (e.g., 2.373e-9 for ccRemover TH2 enrichment)
- 95% confidence interval bounds on the odds ratio (e.g., lower bound 2.297)
- Enrichment statistics table (e.g., th2_enrichment_results.csv)

## How to apply

First identify differentially expressed genes by applying 2-means clustering to the cells and a t-test with false discovery rate 0.01, as done for the 81-cell × 14,147-gene T-helper dataset. Then compute the hypergeometric over-representation of the 122 curated TH2 signature genes among the DE genes, reporting the odds ratio, hypergeometric p-value, and lower 95% CI bound. Apply this identically to the original data and each corrected matrix (ccRemover xhat with gene means re-added, scLVM K=1 and scree-plot K) so the statistics are comparable across methods. Rationale: the odds ratio quantifies whether the clustering captures physiological substructure — on original data the OR was < 1, while ccRemover correction yielded OR 3.439 versus scLVM's 2.382, demonstrating that enrichment strength discriminates between correction methods.

## Related tools

- **ccRemover** (Produces the corrected matrix (cutoff=3, max_it=4, nboot=200, ntop=10) whose DE genes are tested for TH2 signature enrichment; expected OR 3.439) — cran/ccRemover
- **scLVM** (Baseline correction method (default K=1 and scree-plot K) whose DE genes yield a comparison OR of 2.382)
- **org.Mm.eg.db** (Mouse annotation database used to retrieve the cell-cycle gene set that defines control genes for the corrected matrices feeding the enrichment test)

## Examples

```
phyper(q = length(intersect(de_genes, th2_genes)) - 1, m = 122, n = n_genes - 122, k = length(de_genes), lower.tail = FALSE)
```

## Evaluation signals

- On ccRemover-corrected T-helper data the TH2 odds ratio should be 3.439 (lower 95% CI 2.297, hypergeometric p ≈ 2.373e-9)
- scLVM-corrected data should give a lower OR (2.382), confirming the ordering ccRemover > scLVM
- Original (uncorrected) data should yield an odds ratio less than 1, the negative-control baseline
- The same clustering (2-means, t-test FDR 0.01) and gene universe must be applied to every matrix so ORs are comparable
- Enrichment statistics should be exported as a complete table (e.g., th2_enrichment_results.csv) with OR, p-value, and CI per method

## Limitations

- Enrichment results are only interpretable if the upstream clustering is biologically meaningful; on the original T-helper data the OR < 1 indicated the clustering was unlikely to be physiologically meaningful
- The test depends on completeness of the annotation/signature gene set, and annotation databases 'are always incomplete and inaccurate'
- The comparison is specific to a binary 2-means split; other clustering resolutions would change the DE gene set and the OR

## Evidence

- [results] DE gene identification procedure: "we applied 2-means clustering and use a t-test with false discovery rate 0.01 to identify differentially expressed genes"
- [results] Baseline negative result on original data: "On the original data, the odds ratio is less than 1, indicating that the clustering of cells is unlikely to be physiologically meaningful"
- [abstract] Rationale for testing corrected data: "ccRemover preserves other biological signals of interest in the data and thus can serve as an important pre-processing step for many scRNA-Seq data analyses."
