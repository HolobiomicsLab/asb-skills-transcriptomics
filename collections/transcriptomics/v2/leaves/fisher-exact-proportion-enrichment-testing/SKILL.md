---
name: fisher-exact-proportion-enrichment-testing
description: Use when a gene set (e.g., DEGs from rank_genes_groups) must be tested
  for over-representation within a genomic annotation stratum (e.g., XClone-defined
  LOH vs non-LOH regions) using proportional counts and an exact test.
when_to_use_negative:
- InferCNV or CopyKAT outputs are the only CNA annotations available, since both do
  not consider allelic information and hence cannot detect copy-neutral LOH, making
  LOH-vs-non-LOH stratification impossible
- The comparison involves continuous effect sizes rather than counts/proportions in
  discrete strata — Fisher's exact test requires a 2x2 table of counts, not per-gene
  expression statistics
- Strata contain too few genes for a meaningful contingency table (near-zero cell
  counts), where the exact test loses power
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3676
tools:
- name: XClone
  role: Provides the combined CNA-state output (RDR + BAF modules) used to annotate
    each gene's region as LOH vs non-LOH for stratum assignment
  repo: https://github.com/single-cell-genetics/XClone
- name: scanpy
  role: rank_genes_groups (Wilcoxon rank-sum test) generates the Clone 1 vs Normal
    and Clone 2 vs Normal DEG lists counted in the contingency tables
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:d286ad6a785cb40f0e3395208ef296e3e6a1fa0c5b7049e1637aeb50a01d356b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - XClone
  - scanpy
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3676
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# fisher-exact-proportion-enrichment-testing

## Summary

Uses Fisher's exact test on 2x2 contingency tables to test whether the proportion of differentially expressed genes (DEGs) is enriched in one genomic stratum (e.g., copy-neutral LOH regions) versus another (non-LOH regions), as applied to XClone CNA states in the TNBC1 triple-negative breast cancer sample.

## When to use

Use when a gene set (e.g., DEGs from rank_genes_groups) must be tested for over-representation within a genomic annotation stratum (e.g., XClone-defined LOH vs non-LOH regions) using proportional counts and an exact test.

## When NOT to use

- InferCNV or CopyKAT outputs are the only CNA annotations available, since both do not consider allelic information and hence cannot detect copy-neutral LOH, making LOH-vs-non-LOH stratification impossible
- The comparison involves continuous effect sizes rather than counts/proportions in discrete strata — Fisher's exact test requires a 2x2 table of counts, not per-gene expression statistics
- Strata contain too few genes for a meaningful contingency table (near-zero cell counts), where the exact test loses power

## Inputs

- Gene-by-cell expression matrix (e.g., 10x Genomics scRNA-seq counts for TNBC1) with cell-annotation key Clone_ID labeling normal, Clone 1, and Clone 2 cells
- Per-gene CNA state annotations from combined XClone output, stratifying genes into LOH vs non-LOH genomic regions
- DEG calls from scanpy rank_genes_groups (Wilcoxon rank-sum) for Clone 1 vs Normal and Clone 2 vs Normal

## Outputs

- Per-clone 2x2 contingency tables of DEG counts in LOH vs non-LOH regions
- DEG proportions per stratum (e.g., 33.9% vs 21.4% for Clone 1; 27.8% vs 21.8% for Clone 2)
- Fisher's exact test p-values per clone (e.g., p = 1.1e-21 and p = 1.7e-3)
- Optional over-representation test results for pathway enrichment (e.g., Wnt pathway on upregulated Clone 1 genes)

## How to apply

First stratify genes by an annotation layer derived from the combined XClone output (here, CNA states labeling LOH vs non-LOH copy-neutral/copy-gain regions). Then compute per-stratum counts of DEGs and non-DEGs — e.g., for the TNBC1 sample, Clone 1 vs Normal and Clone 2 vs Normal DEGs from scanpy rank_genes_groups (Wilcoxon rank-sum test) — and build a 2x2 table (DEG/not-DEG by LOH/non-LOH) for each clone. Apply Fisher's exact test per clone and report both the proportion of DEGs in each stratum and the p-value. In the reference analysis this reproduced Clone 1: 33.9% vs 21.4% DEGs in LOH vs non-LOH (p = 1.1e-21) and Clone 2: 27.8% vs 21.8% (p = 1.7e-3). A significant result (elevated DEG proportion in LOH regions) supports the finding that LOH regions carry an elevated burden of differentially expressed genes; a complementary over-representation test (e.g., Wnt pathway) on upregulated genes can add biological interpretation.

## Related tools

- **XClone** (Provides the combined CNA-state output (RDR + BAF modules) used to annotate each gene's region as LOH vs non-LOH for stratum assignment) — https://github.com/single-cell-genetics/XClone
- **scanpy** (rank_genes_groups (Wilcoxon rank-sum test) generates the Clone 1 vs Normal and Clone 2 vs Normal DEG lists counted in the contingency tables)

## Examples

```
import scanpy as sc; sc.tl.rank_genes_groups(adata, 'Clone_ID', groups=['Clone 1','Clone 2'], reference='Normal', method='wilcoxon'); from scipy.stats import fisher_exact; odds, p = fisher_exact([[deg_loh, deg_nonloh],[nondeg_loh, nondeg_nonloh]])
```

## Evaluation signals

- Reproduce Clone 1 DEG proportions of 33.9% (LOH) vs 21.4% (non-LOH) with Fisher's exact p = 1.1e-21
- Reproduce Clone 2 DEG proportions of 27.8% (LOH) vs 21.8% (non-LOH) with Fisher's exact p = 1.7e-3
- Both clones must show a significantly elevated DEG proportion in LOH versus non-LOH regions
- Contingency table row/column marginals must equal the total number of genes per stratum from the XClone CNA annotation
- Wnt-pathway over-representation test on upregulated Clone 1 genes should be significant, consistent with the reported finding

## Limitations

- Requires allele-aware CNA calls: methods that ignore allelic information (InferCNV, CopyKAT) cannot detect copy-neutral LOH, so the LOH stratum is unavailable
- Findings such as DEG enrichment in LOH regions are observational associations; the discussion reports the elevated proportion of differentially expressed genes in LOH regions without asserting a causal mechanism
- Independent use of individual SNPs (as in HoneyBADGER/CaSpER) suffers from high noise at low coverage, which can degrade the CNA strata feeding the enrichment test

## Evidence

- [discussion] Elevated proportion of differentially expressed genes observed in LOH regions in TNBC1: "as evidenced in the elevated proportion of deferentially expressed genes in LOH regions in TNBC1"
- [intro] InferCNV and CopyKAT cannot detect copy-neutral LOH: "However, both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH)."
- [intro] XClone combines BAF and RDR modules to generate final CNA states: "this method accounts for two modules: the B-allele frequency (BAF) of heterozygous variants and the sequencing read depth ratio (RDR) of individual genes"
- [discussion] XClone applied to TNBC1 among four cancer samples: "XClone demonstrated its robust and powerful capability in detecting allele-specific CNA states and capturing a broad spectrum of genetic variations"
- [readme] XClone available on PyPI for installation: "XClone package can be conveniently (1~2mins) installed via PyPI::

    pip install xclone"
