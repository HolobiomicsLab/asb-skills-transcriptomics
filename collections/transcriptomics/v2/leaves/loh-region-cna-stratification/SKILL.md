---
name: loh-region-cna-stratification
description: Use when single-cell clone assignments and per-gene CNA/LOH states (e.g.,
  from XClone) are available and you need to test whether DEGs between a tumour clone
  and normal cells concentrate in LOH regions.
when_to_use_negative:
- When CNA calls do not include allelic information (e.g., InferCNV or CopyKAT outputs),
  since these cannot detect copy-neutral LOH
- When the dataset lacks normal (non-malignant) reference cells, making clone-vs-normal
  DE testing undefined
- When the goal is CNA detection itself rather than downstream functional enrichment
  of DEGs in CNA regions
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0204
tools:
- name: XClone
  role: Detects allele-specific CNA states (including copy-neutral LOH) from scRNA-seq,
    providing per-gene LOH/non-LOH region annotations
  repo: https://github.com/single-cell-genetics/XClone
- name: scanpy
  role: Performs differential expression (rank_genes_groups, Wilcoxon rank-sum) for
    each clone versus normal cells
- name: xcltk
  role: Preprocessing toolkit generating the allele-specific AD/DP (BAF) and total
    read depth (RDR) matrices that XClone consumes
  repo: https://github.com/hxj5/xcltk
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:e8f895a82e7e223c0ce8032005cba4d55acd9dc1fc46017ff450f95adc90aade
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/XClone
  tools:
  - XClone
  - scanpy
  - xcltk
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# loh-region-cna-stratification

## Summary

Stratifies genes into copy-neutral LOH versus non-LOH genomic regions using combined XClone CNA calls, then tests whether differentially expressed genes (from scanpy rank_genes_groups) are enriched in LOH regions, as demonstrated on the TNBC1 breast cancer sample.

## When to use

Use when single-cell clone assignments and per-gene CNA/LOH states (e.g., from XClone) are available and you need to test whether DEGs between a tumour clone and normal cells concentrate in LOH regions.

## When NOT to use

- When CNA calls do not include allelic information (e.g., InferCNV or CopyKAT outputs), since these cannot detect copy-neutral LOH
- When the dataset lacks normal (non-malignant) reference cells, making clone-vs-normal DE testing undefined
- When the goal is CNA detection itself rather than downstream functional enrichment of DEGs in CNA regions

## Inputs

- Combined XClone output with per-gene CNA/LOH state calls and cell clone assignments (Clone_ID cell-annotation key), e.g., for TNBC1
- Gene-by-cell scRNA-seq expression matrix (e.g., 10x Genomics droplet data from Gao et al. 2021 TNBC1)
- Clone labels distinguishing normal cells from tumour subclones (Clone 1, Clone 2)

## Outputs

- Per-gene LOH/non-LOH region stratification table
- Per-clone DEG lists from rank_genes_groups (Clone vs Normal)
- DEG proportions in LOH vs non-LOH regions with Fisher's exact test p-values (reported: Clone 1 33.9% vs 21.4%, p = 1.1e-21; Clone 2 27.8% vs 21.8%, p = 1.7e-3)
- Optional pathway over-representation results (e.g., Wnt pathway) for upregulated clone genes

## How to apply

Load the combined XClone output for the sample (e.g., TNBC1), using cell annotations such as Clone_ID to label normal, Clone 1, and Clone 2 cells. Run differential expression with scanpy's rank_genes_groups (Wilcoxon rank-sum test) for each clone versus normal on the gene-by-cell expression matrix. Annotate each gene with its region's CNA state from the XClone results to stratify genes into LOH versus non-LOH (copy-neutral/copy-gain) regions. Compute the proportion of DEGs in LOH versus non-LOH regions per clone and apply Fisher's exact tests to assess enrichment. Optionally follow with pathway over-representation (e.g., Wnt pathway) on upregulated clone genes in LOH regions to link CNA events to clonal phenotypes.

## Related tools

- **XClone** (Detects allele-specific CNA states (including copy-neutral LOH) from scRNA-seq, providing per-gene LOH/non-LOH region annotations) — https://github.com/single-cell-genetics/XClone
- **scanpy** (Performs differential expression (rank_genes_groups, Wilcoxon rank-sum) for each clone versus normal cells)
- **xcltk** (Preprocessing toolkit generating the allele-specific AD/DP (BAF) and total read depth (RDR) matrices that XClone consumes) — https://github.com/hxj5/xcltk

## Examples

```
sc.tl.rank_genes_groups(adata, groupby='Clone_ID', groups=['Clone1','Clone2'], reference='Normal', method='wilcoxon')
```

## Evaluation signals

- Reproduced DEG proportions match reported values: Clone 1 33.9% (LOH) vs 21.4% (non-LOH), p = 1.1e-21
- Clone 2 enrichment reproduced: 27.8% (LOH) vs 21.8% (non-LOH), p = 1.7e-3
- Both Fisher's exact tests show significantly elevated DEG proportions in LOH regions versus non-LOH regions
- Wnt-pathway over-representation on upregulated Clone 1 genes is significant
- Per-gene LOH/non-LOH annotation is consistent with combined XClone CNA calls for TNBC1

## Limitations

- Enrichment conclusions depend on XClone's LOH calling quality; LOH detection requires allelic (BAF) information absent from InferCNV/CopyKAT
- Fisher's exact enrichment treats genes as independent despite genomic coordinate structure
- Quantitative details (p-values, proportions) supporting the discussion-level finding are not reported in the article's discussion section and must be reproduced from the workflow

## Evidence

- [discussion] Copy-neutral LOH regions concentrate DEGs in TNBC1: "as evidenced in the elevated proportion of deferentially expressed genes in LOH regions in TNBC1"
- [results] Clone 1 DEG enrichment in LOH regions with Fisher's exact test: [evidence span withheld — anchor status: not_found]
- [results] Clone 2 DEG enrichment in LOH regions: [evidence span withheld — anchor status: not_found]
- [methods] Workflow uses scanpy rank_genes_groups for clone-vs-normal DE: [evidence span withheld — anchor status: not_found]
- [intro] InferCNV/CopyKAT cannot detect copy-neutral LOH: "However, both InferCNV and CopyKAT do not consider allelic information and hence cannot detect copy-neutral loss of heterozygosity (LOH)."
- [readme] XClone installation from GitHub: "pip install git+https://github.com/single-cell-genetics/XClone"
