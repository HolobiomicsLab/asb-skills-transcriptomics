---
name: cell-type-ground-truth-precision-evaluation
description: Use when a real scRNA-seq dataset has trusted cell-type labels serving
  as ground truth and you need to quantify how precisely a gene-set or pathway score
  (TRS, scDRS) retrieves the trait-relevant cell type.
when_to_use_negative:
- No trusted ground-truth cell-type labels exist for the dataset — precision cannot
  be computed and a simulation-based benchmark (e.g., scDesign2-simulated data) should
  be used instead
- The goal is discovering novel trait-relevant cell types rather than quantifying
  retrieval accuracy of an already-annotated population
- The trait has no biologically plausible correspondence to any cell type in the dataset,
  making precision against a forced ground truth uninterpretable
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: scPagwas
  role: Computes the trait-relevance score (TRS) via pathway activation transformation
    of GWAS + scRNA-seq data, whose retrieval precision is benchmarked
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat AddModuleScore function
  role: Computes the per-cell TRS with control gene sets matched by expression bins
  repo: https://github.com/satijalab/seurat
- name: scDRS 1.0.2
  role: Baseline cell-scoring comparator run twice (MAGMA genes vs scPagwas top 1,000
    genes) to test gene-set quality
  repo: https://github.com/martinjzhang/scDRS
- name: MAGMA
  role: Supplies the GWAS-derived disease gene set used as the scDRS baseline
  repo: https://github.com/gusevlab/fusion_twas
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:5b276fd7ff4815fea7512b769eeefe3903eff8d8db8ac790a41c1489f23311e7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dengchunyu/scPagwas
  repo_url: https://github.com/dengchunyu/scPagwas
  tools:
  - MAGMA
  - Seurat AddModuleScore function
  - scDRS 1.0.2
  - scPagwas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cell-type-ground-truth-precision-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a single-cell trait-relevance scoring method against a real ground-truth scRNA-seq dataset by ranking cells on TRS or scDRS scores and computing retrieval precision at a q<0.05 threshold, as done for monocyte count in the BMMC dataset.

## When to use

Use when a real scRNA-seq dataset has trusted cell-type labels serving as ground truth and you need to quantify how precisely a gene-set or pathway score (TRS, scDRS) retrieves the trait-relevant cell type.

## When NOT to use

- No trusted ground-truth cell-type labels exist for the dataset — precision cannot be computed and a simulation-based benchmark (e.g., scDesign2-simulated data) should be used instead
- The goal is discovering novel trait-relevant cell types rather than quantifying retrieval accuracy of an already-annotated population
- The trait has no biologically plausible correspondence to any cell type in the dataset, making precision against a forced ground truth uninterpretable

## Inputs

- Seurat scRNA-seq object with ground-truth cell-type labels (BMMC: 5,000 monocytes vs T, B, DC, NK cells)
- Preprocessed, pruned GWAS summary statistics text file with six tab-delimited columns (monocyte count, IEU ID ieu-b-31)
- MAGMA-derived disease gene set (for scDRS baseline)
- scPagwas-prioritized top 1,000 trait-relevant genes (scGet_PCC output)
- KEGG pathway list (Genes_by_pathway_kegg) and per-chromosome LD matrices

## Outputs

- Per-cell TRS scores with background-corrected p-values (Get_CorrectBg_p, 100 iterations)
- Per-cell scDRS scores for MAGMA-gene and scPagwas-gene runs
- Retrieval precision against ground-truth monocyte labels at q<0.05 (target: ~0.982 scPagwas; 0.787 vs 0.984 for scDRS)
- Ranked cell lists by TRS / scDRS score

## How to apply

Assemble a ground-truth dataset where one cell type (e.g., 5,000 monocytes in BMMC) is the known trait-relevant population versus non-relevant types (T, B, DC, NK cells), paired with the matching GWAS summary statistics (monocyte count, IEU ID ieu-b-31, pruned tab-delimited file). Run the scoring pipeline — scPagwas with singlecell=TRUE using defaults (maf_filter=0.01, marg=10000 for the 20 kb SNP-to-gene window, pathways of 5-300 genes, n_topgenes=1000, iters_singlecell=100, KEGG pathway list) — prioritizing genes by Pearson correlation of expression with summed gPAS (scGet_PCC) and computing per-cell TRS with Seurat AddModuleScore plus background-corrected p-values via Get_CorrectBg_p over 100 iterations. Define positive cells at q<0.05, rank cells by TRS or scDRS score, and compute retrieval precision against ground-truth labels. For head-to-head gene-set comparison, run scDRS 1.0.2 twice: once with MAGMA-derived genes and once with the scPagwas-prioritized top 1,000 genes, and compare precision. Correct application reproduces the reported calibration: ~98.2% scPagwas precision (Seurat) and scDRS precision improving from ~0.787 (MAGMA genes) to ~0.984 (scPagwas genes).

## Related tools

- **scPagwas** (Computes the trait-relevance score (TRS) via pathway activation transformation of GWAS + scRNA-seq data, whose retrieval precision is benchmarked) — https://github.com/dengchunyu/scPagwas
- **Seurat AddModuleScore function** (Computes the per-cell TRS with control gene sets matched by expression bins) — https://github.com/satijalab/seurat
- **scDRS 1.0.2** (Baseline cell-scoring comparator run twice (MAGMA genes vs scPagwas top 1,000 genes) to test gene-set quality) — https://github.com/martinjzhang/scDRS
- **MAGMA** (Supplies the GWAS-derived disease gene set used as the scDRS baseline) — https://github.com/gusevlab/fusion_twas

## Examples

```
Pagwas <- scPagwas_main2(gwas_data='monocyte_count_pruned.txt', Single_data='BMMC.rds', singlecell=TRUE, n_topgenes=1000, iters_singlecell=100, Pathway_list=Genes_by_pathway_kegg, chrom_ld=chrom_ld)
```

## Evaluation signals

- scPagwas TRS-based retrieval precision on the BMMC ground truth reproduces approximately 98.2% (Figure 3F)
- scDRS precision with scPagwas top 1,000 genes reaches ~0.984, versus ~0.787 with MAGMA genes (Figures 3G-3H)
- Positive cells defined at q<0.05 are overwhelmingly monocytes (5,000 monocytes) rather than non-monocyte T, B, DC, or NK cells
- Ranked top cells by TRS score overlap the ground-truth monocyte labels; Get_CorrectBg_p runs over 100 iterations without divergence

## Limitations

- Identification of a statistical association between a trait and individual cells does not imply causality but may reflect indirect identification of causal associations
- Pathway analysis is restricted to canonical KEGG pathways only
- The default TRS relies on Seurat's AddModuleScore cell-scoring method for Seurat compatibility, which may constrain alternative scoring schemes
- SNPs are annotated to genes solely by proximal distance (20 kb window), so functionally informed SNP-to-gene linking is not captured

## Evidence

- [results] precision at q threshold: [evidence span withheld — anchor status: not_found]
- [results] scDRS gene-set comparison: [evidence span withheld — anchor status: not_found]
