---
name: scdrs-gene-set-based-cell-scoring
description: Use when you need to score individual cells for association with a GWAS
  trait using a fixed disease/gene set, or to test whether alternative gene-prioritization
  methods improve scDRS cell identification.
when_to_use_negative:
- Input is a bulk tissue-level expression dataset — scDRS is designed for single-cell
  resolution and existing GWAS+scRNA-seq methods like LDSC-SEG/MAGMA cover bulk cell-type
  inference
- No GWAS-prioritized gene set is available and you cannot run MAGMA or scPagwas gene
  prioritization — scDRS accuracy relies heavily on the disease-specific gene set
- The goal is pathway-level association with LD-aware statistics rather than per-cell
  scoring — scPagwas's polygenic regression on pathway activity transformation is
  the primary method
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: scDRS
  role: Compute per-cell disease-associated scores from GWAS-prioritized gene sets
    on the scRNA-seq data (version 1.0.2)
  repo: https://github.com/martinjzhang/scDRS
- name: scPagwas
  role: Prioritize the top 1,000 trait-relevant genes via polygenic regression on
    pathway activity-transformed scRNA-seq data, used as the improved scDRS input
    gene set
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat AddModuleScore function
  role: Compute per-cell trait-relevant scores (TRS) for the scPagwas comparison arm,
    with background correction via Get_CorrectBg_p (iters_singlecell = 100)
  repo: https://github.com/satijalab/seurat
- name: scDesign2
  role: Simulate the ground-truth scRNA-seq dataset (five cell types matching FACS-sorted
    bulk populations from GSE107011) used for precision benchmarking (version 1.0.0)
  repo: https://github.com/JSB-UCLA/scDesign2
- name: MAGMA
  role: Gene-based GWAS association test providing the baseline top 1,000 gene set
    for the scDRS comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:81dda9ca8c4b2b0c6086ac5df9122af4aad6ab9709b0b6c42f0437375eb75629
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dengchunyu/scPagwas
  repo_url: https://github.com/martinjzhang/scDRS
  tools:
  - MAGMA
  - Seurat AddModuleScore function
  - scDRS
  - scDesign2
  - scPagwas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# scDRS gene-set-based cell scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply scDRS (v1.0.2) to compute per-cell trait-relevance scores from a GWAS-prioritized gene set on scRNA-seq data, and benchmark how the choice of input genes (MAGMA vs scPagwas) affects precision of trait-relevant cell identification against a scDesign2-simulated ground truth.

## When to use

Use when you need to score individual cells for association with a GWAS trait using a fixed disease/gene set, or to test whether alternative gene-prioritization methods improve scDRS cell identification.

## When NOT to use

- Input is a bulk tissue-level expression dataset — scDRS is designed for single-cell resolution and existing GWAS+scRNA-seq methods like LDSC-SEG/MAGMA cover bulk cell-type inference
- No GWAS-prioritized gene set is available and you cannot run MAGMA or scPagwas gene prioritization — scDRS accuracy relies heavily on the disease-specific gene set
- The goal is pathway-level association with LD-aware statistics rather than per-cell scoring — scPagwas's polygenic regression on pathway activity transformation is the primary method

## Inputs

- scRNA-seq expression matrix (e.g., Seurat object or simulated scDesign2 dataset with five cell types)
- GWAS summary statistics (e.g., monocyte count, IEU ID ieu-b-31)
- Trait-relevant gene set: top 1,000 MAGMA-prioritized genes and/or top 1,000 scPagwas-prioritized genes
- Ground-truth cell-type labels from the simulated dataset

## Outputs

- Per-cell scDRS disease scores for each gene set tested
- Trait-relevant cell calls at p < 0.05 (scPagwas arm, via Get_CorrectBg_p)
- Precision benchmark table (precision_benchmark.csv) with precision per method (e.g., scPagwas = 95.9%, scDRS(MAGMA) = 0.940, scDRS(scPagwas genes) = 0.957)

## How to apply

Obtain a scRNA-seq dataset and a trait gene set (e.g., top 1,000 MAGMA-prioritized genes or top 1,000 scPagwas-prioritized genes derived with KEGG pathways, 20 kb SNP-to-gene window, MAF filter 0.01, pathway size 5-300). Run scDRS (version 1.0.2) on the normalized single-cell matrix with each gene set to assign per-cell disease scores. For comparison against scPagwas, compute per-cell TRS with Seurat AddModuleScore, background-correct p-values via Get_CorrectBg_p with iters_singlecell = 100, and call trait-relevant cells at p < 0.05. Evaluate precision against a scDesign2 (v1.0.0)-simulated ground truth in which the trait-relevant cell type is held out; the article found scDRS precision rose from 0.940 (MAGMA genes) to 0.957 (scPagwas genes), versus 95.9% for scPagwas itself. Rationale: scDRS accuracy depends heavily on the quality of GWAS-prioritized disease-specific genes, so gene-set provenance is the key experimental variable.

## Related tools

- **scDRS** (Compute per-cell disease-associated scores from GWAS-prioritized gene sets on the scRNA-seq data (version 1.0.2)) — https://github.com/martinjzhang/scDRS
- **scPagwas** (Prioritize the top 1,000 trait-relevant genes via polygenic regression on pathway activity-transformed scRNA-seq data, used as the improved scDRS input gene set) — https://github.com/dengchunyu/scPagwas
- **Seurat AddModuleScore function** (Compute per-cell trait-relevant scores (TRS) for the scPagwas comparison arm, with background correction via Get_CorrectBg_p (iters_singlecell = 100)) — https://github.com/satijalab/seurat
- **scDesign2** (Simulate the ground-truth scRNA-seq dataset (five cell types matching FACS-sorted bulk populations from GSE107011) used for precision benchmarking (version 1.0.0)) — https://github.com/JSB-UCLA/scDesign2
- **MAGMA** (Gene-based GWAS association test providing the baseline top 1,000 gene set for the scDRS comparison)

## Examples

```
Rscript -e 'scPagwas_main(gwas_data="ieu-b-31.txt",Single_data="simulated.rds",Pathway_list=Genes_by_pathway_kegg,marg=10000,maf_filter=0.01,min.pathway.size=5,max.pathway.size=300,n_topgenes=1000,iters_singlecell=100)'
```

## Evaluation signals

- precision_benchmark.csv exists and contains a precision column
- scPagwas precision falls within a few percentage points of the reported 95.9%
- scDRS precision using scPagwas-prioritized genes (0.957) exceeds scDRS precision using MAGMA-prioritized genes (0.940)
- Both scDRS runs use the same simulated dataset and differ only in the input gene set (top 1,000 MAGMA vs top 1,000 scPagwas genes)
- Trait-relevant cells called at p < 0.05 from background-corrected scores agree with the scDesign2 ground-truth cell-type labels

## Limitations

- scDRS accuracy relies heavily on the set of disease-specific genes identified from GWAS data using gene-based association tests such as MAGMA; poor gene prioritization degrades cell identification
- Top genetic association signals at specific loci may be absent from most cells because of extensive sparsity and technical noise in single-cell data, making per-cell-based inference difficult for gene-scoring methods
- The benchmark uses a scDesign2-simulated ground truth holding out a single trait-relevant cell type; results may not transfer to real data without known truth labels

## Evidence
