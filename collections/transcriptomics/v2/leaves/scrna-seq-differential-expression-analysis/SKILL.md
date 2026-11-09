---
name: scrna-seq-differential-expression-analysis
description: Use when a cell-by-gene expression matrix has cells labeled by clone/copy-state
  (e.g., XClone Clone_ID annotation) and the goal is to test whether DEGs between
  clones are enriched in LOH regions, as in TNBC1.
when_to_use_negative:
- CNA inference has not yet been run and no per-region LOH/CNA-state annotation exists
  — run XClone (RDR + BAF modules) first, since this skill depends on the CNA-state
  stratification
- The analysis goal is detecting copy-neutral LOH itself from scRNA-seq — use XClone
  directly, as methods like InferCNV and CopyKAT cannot detect copy-neutral LOH
- Data are from single-cell DNA-seq pipelines (e.g., ACT, CHISEL) rather than gene-by-cell
  scRNA-seq expression matrices, so there are no expression values to test
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: XClone
  role: Provides clone assignments (Clone_ID) and per-gene/per-region CNA states (LOH
    vs non-LOH) that define the groups compared and the genomic stratification used
    in the DEG enrichment test
  repo: https://github.com/single-cell-genetics/XClone
- name: scanpy
  role: Performs differential expression with rank_genes_groups (Wilcoxon rank-sum)
    for Clone 1 vs Normal and Clone 2 vs Normal, and supports over-representation
    analysis on upregulated genes
- name: xcltk
  role: Preprocessing toolkit that generates the AD/DP (BAF) and total read depth
    (RDR) matrices consumed by XClone before clonal annotation and DE analysis
  repo: https://github.com/hxj5/xcltk
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:12f72047370671c05d984a35d4c77099151dd022724ff74476417ce666796c66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - XClone
  - scanpy
  - xcltk
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scrna-seq-differential-expression-analysis

## Summary

Identifies differentially expressed genes (DEGs) between clonal populations in an XClone-annotated single-cell dataset (e.g., Clone 1 vs Normal and Clone 2 vs Normal in the TNBC1 triple-negative breast cancer sample), and links DEG burden to genomic region state (copy-neutral LOH vs non-LOH) detected by XClone.

## When to use

Use when a cell-by-gene expression matrix has cells labeled by clone/copy-state (e.g., XClone Clone_ID annotation) and the goal is to test whether DEGs between clones are enriched in LOH regions, as in TNBC1.

## When NOT to use

- CNA inference has not yet been run and no per-region LOH/CNA-state annotation exists — run XClone (RDR + BAF modules) first, since this skill depends on the CNA-state stratification
- The analysis goal is detecting copy-neutral LOH itself from scRNA-seq — use XClone directly, as methods like InferCNV and CopyKAT cannot detect copy-neutral LOH
- Data are from single-cell DNA-seq pipelines (e.g., ACT, CHISEL) rather than gene-by-cell scRNA-seq expression matrices, so there are no expression values to test

## Inputs

- Combined XClone output (h5ad) for the TNBC1 sample with cell annotations under the Clone_ID key (normal, Clone 1, Clone 2)
- Gene-by-cell scRNA-seq expression matrix (10x Genomics droplet-based TNBC1 data from Gao et al. 2021)
- Per-gene/per-region CNA-state assignments from XClone's combined BAF and RDR modules (LOH vs copy-neutral/copy-gain labels)

## Outputs

- Ranked differential expression results per clone vs normal (Wilcoxon rank-sum, via scanpy rank_genes_groups)
- DEG proportions in LOH vs non-LOH regions per clone (Clone 1: 33.9% vs 21.4%; Clone 2: 27.8% vs 21.8%)
- Fisher's exact test p-values for DEG enrichment in LOH regions (Clone 1 p = 1.1e-21; Clone 2 p = 1.7e-3)
- Over-representation analysis of Wnt-pathway terms on upregulated Clone 1 genes

## How to apply

Load the combined XClone output for the TNBC1 droplet-based (10x Genomics) sample, labeling cells as normal, Clone 1, or Clone 2 via the Clone_ID cell-annotation key. Run differential expression with scanpy's rank_genes_groups using the Wilcoxon rank-sum test for Clone 1 vs Normal and Clone 2 vs Normal. Annotate each gene with its region's CNA state from the combined XClone results to stratify genes into LOH (copy-neutral) vs non-LOH genomic regions. Compute the proportion of DEGs per stratum and apply Fisher's exact tests; the expected reproduction is Clone 1: 33.9% vs 21.4% of genes DEG in LOH vs non-LOH (p = 1.1e-21) and Clone 2: 27.8% vs 21.8% (p = 1.7e-3). Optionally follow with an over-representation test of Wnt-pathway terms on upregulated Clone 1 genes, since elevated DEG burden in LOH regions reflects functional impact of allele-specific CNAs.

## Related tools

- **XClone** (Provides clone assignments (Clone_ID) and per-gene/per-region CNA states (LOH vs non-LOH) that define the groups compared and the genomic stratification used in the DEG enrichment test) — https://github.com/single-cell-genetics/XClone
- **scanpy** (Performs differential expression with rank_genes_groups (Wilcoxon rank-sum) for Clone 1 vs Normal and Clone 2 vs Normal, and supports over-representation analysis on upregulated genes)
- **xcltk** (Preprocessing toolkit that generates the AD/DP (BAF) and total read depth (RDR) matrices consumed by XClone before clonal annotation and DE analysis) — https://github.com/hxj5/xcltk

## Examples

```
import scanpy as sc; sc.tl.rank_genes_groups(adata, groupby='Clone_ID', groups=['Clone 1','Clone 2'], reference='Normal', method='wilcoxon')
```

## Evaluation signals

- Reproduced Clone 1 DEG proportions in LOH vs non-LOH regions match 33.9% vs 21.4% with Fisher's exact p = 1.1e-21
- Reproduced Clone 2 DEG proportions match 27.8% vs 21.8% with Fisher's exact p = 1.7e-3
- Both clones show significantly elevated DEG proportion in LOH regions (direction of enrichment consistent across Clone 1 and Clone 2)
- Every gene in the analysis carries a valid CNA-state label (LOH vs non-LOH) from the combined XClone output; no genes dropped due to missing region annotation
- Wnt-pathway over-representation on upregulated Clone 1 genes is significant

## Limitations

- XClone was demonstrated on specific cancer samples (glioma BCH896, ATC, TNBC1, astrocytoma); performance on other compositions may vary
- The discussion cites the elevated DEG proportion in TNBC1 LOH regions without accompanying quantitative detail in that section, so exact thresholds and support must come from the companion workflow
- Sparse low-coverage scRNA-seq (10x Genomics droplet data) limits statistical power, which is why effective smoothing of read-depth and allelic-imbalance signals is required upstream

## Evidence

- [discussion] as evidenced in the elevated proportion of deferentially expressed genes in LOH regions in TNBC1: "as evidenced in the elevated proportion of deferentially expressed genes in LOH regions in TNBC1"
- [intro] InferCNV and CopyKAT cannot detect copy-neutral LOH: "However, both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH)."
- [readme] XClone infers allele-specific copy numbers from sparse single-cell RNA-seq: "XClone is an algorithm to infer allele- and haplotype-specific copy numbers 
in individual cells from low-coverage and sparse single-cell RNA sequencing data"
- [intro] XClone combines BAF and RDR modules to generate final CNA states: "this method accounts for two modules: the B-allele frequency (BAF) of heterozygous variants and the sequencing read depth ratio (RDR) of individual genes"
