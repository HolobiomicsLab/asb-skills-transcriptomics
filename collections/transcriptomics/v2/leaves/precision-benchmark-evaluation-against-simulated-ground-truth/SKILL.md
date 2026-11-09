---
name: precision-benchmark-evaluation-against-simulated-ground-truth
description: Use when you need to quantify and compare per-cell precision of trait-relevant
  cell identification methods against a known ground truth, e.g. testing whether scPagwas-prioritized
  genes improve scDRS.
when_to_use_negative:
- There is no simulated or known ground truth for trait-relevant cells, so precision
  cannot be computed against held-out labels
- The goal is cell-type-level GWAS enrichment (e.g. LDSC-SEG, RolyPoly, MAGMA-based
  or EPIC tissue/cell-type inference) rather than single-cell-level identification
  of trait-relevant individual cells
- Input GWAS trait lacks sufficient sample size or gene-level association signal to
  prioritize a reliable top-1,000 gene set for scDRS
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: scDesign2
  role: Simulates the ground-truth scRNA-seq dataset (version 1.0.0) with five cell
    types matching bulk-sorted populations, enabling precision benchmarking
  repo: https://github.com/JSB-UCLA/scDesign2
- name: scPagwas
  role: Prioritizes the top 1,000 trait-relevant genes from monocyte count GWAS and
    computes per-cell TRS via Seurat AddModuleScore with Get_CorrectBg_p background
    correction
  repo: https://github.com/dengchunyu/scPagwas
- name: scDRS
  role: Version 1.0.2; run twice with MAGMA- vs scPagwas-prioritized top-1,000 genes
    to compare per-cell trait-relevance identification
  repo: https://github.com/martinjzhang/scDRS
- name: Seurat
  role: AddModuleScore function computes the transcriptional relevance score (TRS)
    per cell with control gene sets matched by expression bins
  repo: https://github.com/satijalab/seurat
- name: MAGMA
  role: Gene-based association test producing the baseline top-1,000 prioritized gene
    set used in the scDRS comparison arm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:5f24c218f1f465ccfe83f4d1827fadbd6e1192b37bcf3d42791253ab592226ac
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dengchunyu/scPagwas
  repo_url: https://github.com/JSB-UCLA/scDesign2
  tools:
  - MAGMA
  - Seurat
  - scDRS
  - scDesign2
  - scPagwas
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# precision-benchmark-evaluation-against-simulated-ground-truth

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark how well per-cell trait-relevance methods (scPagwas, scDRS with MAGMA- vs scPagwas-prioritized genes) recover trait-relevant cells, using an scDesign2-simulated ground-truth scRNA-seq dataset in which the trait-relevant cell type is held out for validation.

## When to use

Use when you need to quantify and compare per-cell precision of trait-relevant cell identification methods against a known ground truth, e.g. testing whether scPagwas-prioritized genes improve scDRS.

## When NOT to use

- There is no simulated or known ground truth for trait-relevant cells, so precision cannot be computed against held-out labels
- The goal is cell-type-level GWAS enrichment (e.g. LDSC-SEG, RolyPoly, MAGMA-based or EPIC tissue/cell-type inference) rather than single-cell-level identification of trait-relevant individual cells
- Input GWAS trait lacks sufficient sample size or gene-level association signal to prioritize a reliable top-1,000 gene set for scDRS

## Inputs

- FACS-sorted bulk hematopoietic expression data from GEO accession GSE107011
- Monocyte count GWAS summary statistics (IEU ID: ieu-b-31)
- Simulated ground-truth scRNA-seq dataset generated with scDesign2 (five cell types)
- KEGG pathway list (Genes_by_pathway_kegg)
- Top 1,000 MAGMA-prioritized genes and top 1,000 scPagwas-prioritized genes

## Outputs

- precision_benchmark.csv containing a precision column per method (scPagwas ~95.9%, scDRS(MAGMA) = 0.940, scDRS(scPagwas genes) = 0.957)
- Per-cell trait-relevance calls at p < 0.05 from scPagwas TRS (AddModuleScore + Get_CorrectBg_p)
- scDRS per-cell scores run under two gene-prioritization inputs

## How to apply

Download FACS-sorted bulk hematopoietic expression data (GEO GSE107011) and monocyte count GWAS summary statistics (IEU ID: ieu-b-31), then simulate a ground-truth scRNA-seq dataset with scDesign2 v1.0.0 containing five cell types matching the bulk-sorted populations, holding out the trait-relevant cell type. Run scPagwas_main with the KEGG pathway list (Genes_by_pathway_kegg), marg = 10000 (20 kb SNP-to-gene window), maf_filter = 0.01, pathway size limits 5-300, and n_topgenes = 1000 to derive trait-relevant genes; compute per-cell TRS with Seurat AddModuleScore and background-corrected p-values via Get_CorrectBg_p (iters_singlecell = 100), calling trait-relevant cells at p < 0.05. Run scDRS v1.0.2 twice on the same simulated data, once with the top 1,000 MAGMA-prioritized genes and once with the top 1,000 scPagwas-prioritized genes. Compute precision against the scDesign2 ground truth for each method and write precision_benchmark.csv; the decision point is whether scDRS(scPagwas genes) precision exceeds scDRS(MAGMA) precision.

## Related tools

- **scDesign2** (Simulates the ground-truth scRNA-seq dataset (version 1.0.0) with five cell types matching bulk-sorted populations, enabling precision benchmarking) — https://github.com/JSB-UCLA/scDesign2
- **scPagwas** (Prioritizes the top 1,000 trait-relevant genes from monocyte count GWAS and computes per-cell TRS via Seurat AddModuleScore with Get_CorrectBg_p background correction) — https://github.com/dengchunyu/scPagwas
- **scDRS** (Version 1.0.2; run twice with MAGMA- vs scPagwas-prioritized top-1,000 genes to compare per-cell trait-relevance identification) — https://github.com/martinjzhang/scDRS
- **Seurat** (AddModuleScore function computes the transcriptional relevance score (TRS) per cell with control gene sets matched by expression bins) — https://github.com/satijalab/seurat
- **MAGMA** (Gene-based association test producing the baseline top-1,000 prioritized gene set used in the scDRS comparison arm)

## Examples

```
Pagwas_data <- scPagwas_main(Pagwas=NULL, gwas_data=ieu-b-31_summary.txt, Single_data=simulated_scDesign2.rds, Pathway_list=Genes_by_pathway_kegg, marg=10000, maf_filter=0.01, min.pathway.size=5, max.pathway.size=300, n_topgenes=1000, iters_singlecell=100, singlecell=T, celltype=T)
```

## Evaluation signals

- precision_benchmark.csv exists and contains a precision column
- scPagwas precision is within a few percentage points of the reported 95.9%
- scDRS precision with scPagwas-prioritized genes (0.957) exceeds scDRS precision with MAGMA-prioritized genes (0.940)
- Simulated dataset contains five cell types whose profiles match the bulk-sorted populations from GSE107011, with the trait-relevant cell type held out
- Trait-relevant cell calls are made at background-corrected p < 0.05 with iters_singlecell = 100

## Limitations

- Statistical association between traits and individual cells does not imply causality but may reflect indirect identification of causal associations
- SNPs are annotated to genes solely by proximal distance (20 kb window); functionally informed SNP-to-gene linking was deferred to future work
- Pathway analysis is restricted to canonical KEGG pathways, which limits pathway coverage

## Evidence

- [readme] scPagwas Install via devtools from sulab-wmu GitHub: [evidence span withheld — anchor status: external_doc]
