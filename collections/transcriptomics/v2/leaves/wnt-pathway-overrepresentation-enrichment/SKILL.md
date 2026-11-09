---
name: wnt-pathway-overrepresentation-enrichment
description: Use when upregulated genes from a clone-versus-normal differential expression
  comparison (e.g., Clone 1 vs Normal in XClone-annotated TNBC1 scRNA-seq data) need
  to be tested for significant Wnt-pathway over-representation via an over-representation
  test.
when_to_use_negative:
- Input cells have not been assigned to clones (no Clone_ID annotation or XClone CNA
  result), so no clone-specific upregulated gene set can be defined
- The goal is general GSEA-style ranked pathway analysis over all genes rather than
  an over-representation test restricted to a specific upregulated gene set
- The sample is not a cancer dataset with allele-specific CNA/LOH structure — the
  rationale (linking LOH-region DEG burden to pathway impact) does not apply
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: XClone
  role: Produces the combined CNA/clone output for TNBC1 (Clone_ID annotations and
    LOH vs non-LOH region states) that defines the clone groups and regional stratification
    for DEG analysis
  repo: https://github.com/single-cell-genetics/XClone
- name: scanpy
  role: Runs rank_genes_groups (Wilcoxon rank-sum test) on the gene-by-cell expression
    matrix to obtain the upregulated Clone 1 gene list that is fed into the Wnt-pathway
    ORA
  repo: https://github.com/single-cell-genetics/XClone
- name: xcltk
  role: Preprocessing toolkit generating the RDR and BAF input matrices (AD/DP and
    total read depth) used to run XClone upstream of the DEG and pathway analysis
  repo: https://github.com/hxj5/xcltk
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:692c83ecd9f1a67281a5e024203252a0c3c1182d738d60bbb53cf728e8a73f7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - XClone
  - scanpy
  - xcltk
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# wnt-pathway-overrepresentation-enrichment

## Summary

Over-representation test of the Wnt signaling pathway on upregulated Clone 1 differentially expressed genes in the TNBC1 sample, following stratification of DEGs by XClone-detected LOH versus non-LOH regions. It connects clone-specific CNA burden (LOH enrichment of DEGs) to downstream pathway-level phenotypic impact.

## When to use

Use when upregulated genes from a clone-versus-normal differential expression comparison (e.g., Clone 1 vs Normal in XClone-annotated TNBC1 scRNA-seq data) need to be tested for significant Wnt-pathway over-representation via an over-representation test.

## When NOT to use

- Input cells have not been assigned to clones (no Clone_ID annotation or XClone CNA result), so no clone-specific upregulated gene set can be defined
- The goal is general GSEA-style ranked pathway analysis over all genes rather than an over-representation test restricted to a specific upregulated gene set
- The sample is not a cancer dataset with allele-specific CNA/LOH structure — the rationale (linking LOH-region DEG burden to pathway impact) does not apply

## Inputs

- Gene-by-cell expression matrix (scRNA-seq count matrix) from the TNBC1 triple-negative breast cancer droplet-based (10x Genomics) sample
- Combined XClone output with clone assignments stored under the Clone_ID cell-annotation key (Normal, Clone 1, Clone 2) and per-gene/per-region CNA states (LOH vs non-LOH)
- Differential expression results from scanpy rank_genes_groups (Wilcoxon rank-sum test), specifically the upregulated Clone 1 gene list
- Wnt signaling pathway gene set for the over-representation test

## Outputs

- Over-representation test statistic and significance (p-value / adjusted p-value) for Wnt-pathway enrichment among upregulated Clone 1 genes
- Ranked differential expression table per clone (Clone 1 vs Normal, Clone 2 vs Normal) with upregulated gene sets
- Supporting context: per-clone DEG proportions in LOH vs non-LOH regions with Fisher's exact p-values (Clone 1: 33.9% vs 21.4%, p = 1.1e-21; Clone 2: 27.8% vs 21.8%, p = 1.7e-3)

## How to apply

First derive the gene set: using the combined XClone output for TNBC1 with clone labels under the Clone_ID cell-annotation key, run differential expression with scanpy's rank_genes_groups (Wilcoxon rank-sum test) for Clone 1 vs Normal (and Clone 2 vs Normal as a parallel comparison) on the gene-by-cell expression matrix, then select the upregulated Clone 1 genes. Second, run a Wnt-pathway over-representation test (ORA) on this upregulated gene set, testing whether Wnt pathway members occur among upregulated genes more often than expected by chance given the background gene list. Third, interpret the ORA result alongside the DEG-by-region analysis: report the per-clone DEG proportions in LOH vs non-LOH regions (Clone 1: 33.9% vs 21.4%, p = 1.1e-21; Clone 2: 27.8% vs 21.8%, p = 1.7e-3, Fisher's exact test) to show that genes in XClone-detected LOH regions carry a significant expression burden, and use the Wnt ORA on the upregulated Clone 1 genes to identify a candidate pathway driving the Clone 1 phenotype.

## Related tools

- **XClone** (Produces the combined CNA/clone output for TNBC1 (Clone_ID annotations and LOH vs non-LOH region states) that defines the clone groups and regional stratification for DEG analysis) — https://github.com/single-cell-genetics/XClone
- **scanpy** (Runs rank_genes_groups (Wilcoxon rank-sum test) on the gene-by-cell expression matrix to obtain the upregulated Clone 1 gene list that is fed into the Wnt-pathway ORA) — https://github.com/single-cell-genetics/XClone
- **xcltk** (Preprocessing toolkit generating the RDR and BAF input matrices (AD/DP and total read depth) used to run XClone upstream of the DEG and pathway analysis) — https://github.com/hxj5/xcltk

## Examples

```
sc.tl.rank_genes_groups(adata, groupby='Clone_ID', groups=['Clone 1'], reference='Normal', method='wilcoxon'); up1 = sc.get.rank_genes_groups_df(adata, group='Clone 1').query('scores > 0')['names']
```

## Evaluation signals

- Reproduced DEG proportions in LOH vs non-LOH regions match the reported values: Clone 1 33.9% vs 21.4% (p = 1.1e-21) and Clone 2 27.8% vs 21.8% (p = 1.7e-3) by Fisher's exact test, with elevated DEG proportion in LOH regions for both clones
- The upregulated gene set analyzed is specifically the upregulated Clone 1 (vs Normal) gene set, not all DEGs or Clone 2 upregulated genes
- The over-representation test reports a significant enrichment of Wnt-pathway genes among upregulated Clone 1 genes
- DEG calls are reproducible: scanpy rank_genes_groups with Wilcoxon rank-sum test on the Clone_ID-stratified gene-by-cell matrix
- Clone assignments and region CNA states trace back to the combined XClone output for TNBC1 (10x Genomics, Gao et al. 2021 source data)

## Limitations

- The quantitative DEG-proportion and Fisher's p-value details underlying the discussion-level claim ('elevated proportion of deferentially expressed genes in LOH regions in TNBC1') are not present in the paper's discussion section, so reproduction depends on the workflow values rather than printed article tables
- The paper does not report a data accession identifier for TNBC1 in the discussion; the sample must be sourced via the xclone-data repository examples (TNBC1_scRNA folder / TNBC1_XClone_tutorials.ipynb)
- ORA is a categorical test on a single gene set (Wnt pathway) and does not quantify graded pathway activity; it depends on the quality of the upstream clone labeling and Wilcoxon DEG ranking

## Evidence

- [other] Run differential expression with scanpy's rank_genes_groups (Wilcoxon rank-sum test) for Clone 1 vs normal and Clone 2 vs normal on the gene-by-cell expression matrix.: [evidence span withheld — anchor status: not_found]
- [other] Perform Wnt-pathway enrichment analysis (over-representation test) on the upregulated Clone 1 genes.: [evidence span withheld — anchor status: not_found]
- [discussion] Elevated proportion of differentially expressed genes observed in LOH regions in TNBC1: "as evidenced in the elevated proportion of deferentially expressed genes in LOH regions in TNBC1"
- [methods] XClone RDR module run command from the methods: "RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file = xconfig)"
