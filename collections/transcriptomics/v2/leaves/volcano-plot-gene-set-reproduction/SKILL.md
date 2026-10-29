---
name: volcano-plot-gene-set-reproduction
description: Use when a publication reports a volcano-plot-style set of up/down-regulated
  genes within specific cell clusters (e.g. COVID vs healthy in Macrophage clusters
  3, 5, 12) and the goal is to regenerate that exact gene list with direction labels.
when_to_use_negative:
- The analysis goal is novel cell discovery/annotation itself rather than post-hoc
  DE within already-detected clusters — use scNovel's detection pipeline first.
- There is only a single cluster or the clusters are not biologically dependent, so
  cross-cluster consistency intersection is meaningless.
- Input is bulk RNA-seq or a pre-computed feature table with no per-cell cluster and
  condition labels to subset by.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCANPY
  role: Provides sc.tl.rank_genes_groups for condition-vs-control differential expression
    within selected clusters and sc.pp.normalize_total/log1p preprocessing
  repo: https://github.com/chuanyang-Zheng/scNovel
- name: scNovel
  role: Produces the novel Macrophage subtype clusters (3, 5, 12) whose member cells
    are the population in which DE is computed
  repo: https://github.com/chuanyang-Zheng/scNovel
- name: Python
  role: Scripting environment for the DE pipeline and compiling the final gene/direction
    table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:0df1f22aa30a0040a34739f91904893987b105b0bf32e0ed6c1405b95cfca771
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chuanyang-Zheng/scNovel
  tools:
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# volcano-plot-gene-set-reproduction

## Summary

Reproduces a figure-style differential gene expression gene set by running pairwise condition-vs-control DE within detected cell clusters and intersecting biomarkers across dependent clusters, as in scNovel's COVID-19 Macrophage analysis (Figure 6H).

## When to use

Use when a publication reports a volcano-plot-style set of up/down-regulated genes within specific cell clusters (e.g. COVID vs healthy in Macrophage clusters 3, 5, 12) and the goal is to regenerate that exact gene list with direction labels.

## When NOT to use

- The analysis goal is novel cell discovery/annotation itself rather than post-hoc DE within already-detected clusters — use scNovel's detection pipeline first.
- There is only a single cluster or the clusters are not biologically dependent, so cross-cluster consistency intersection is meaningless.
- Input is bulk RNA-seq or a pre-computed feature table with no per-cell cluster and condition labels to subset by.

## Inputs

- scNovel-annotated single-cell expression matrix (Scanpy AnnData) with cluster labels, patient ID, and COVID/healthy status (GSE171524, 116,314 bronchoalveolar immune cells)
- Cluster identity mapping (e.g. Macrophage subtype clusters 3, 5, 12 identified by high CD163 expression)
- Condition labels (COVID-19 vs healthy control) per cell or per patient

## Outputs

- Per-cluster ranked DE gene tables from rank_genes_groups (top 8 biomarkers per cluster, Figure 6G style)
- Final compiled table of consistently differentially expressed genes with direction labels (7 up-regulated, 9 down-regulated; Figure 6H style)

## How to apply

Load the cluster-annotated single-cell object (e.g. the GSE171524 bronchoalveolar immune atlas of 116,314 cells from 7 healthy and 20 COVID-19 samples) in Scanpy, retaining patient and COVID status annotations, and select the target clusters — here the CD163-high Macrophage subtype clusters 3, 5, and 12 identified by scNovel. Run sc.tl.rank_genes_groups separately within each cluster, contrasting COVID against healthy samples, and take the top biomarkers per cluster (top 8 per cluster as in Figure 6G). Intersect the biomarker lists across the three dependent clusters to keep only genes with consistent differential expression in all of them, reproducing the Figure 6H gene set. Compile the final table with expression direction: expected up-regulated genes are ADAMTS2, CD163, CEBPB, CTSD, LDLRAD3, NEAT1, TPST1 and down-regulated genes are ATG77, CSF2RA, DST, FARP1, FRMD4A, HLA-DPB1, HLA-DRB1, RASAL2, TFRC.

## Related tools

- **SCANPY** (Provides sc.tl.rank_genes_groups for condition-vs-control differential expression within selected clusters and sc.pp.normalize_total/log1p preprocessing) — https://github.com/chuanyang-Zheng/scNovel
- **scNovel** (Produces the novel Macrophage subtype clusters (3, 5, 12) whose member cells are the population in which DE is computed) — https://github.com/chuanyang-Zheng/scNovel
- **Python** (Scripting environment for the DE pipeline and compiling the final gene/direction table)

## Examples

```
import scanpy as sc; sub = adata[adata.obs['cluster'].isin([3,5,12])]; sc.tl.rank_genes_groups(sub, groupby='COVID_status', groups=['COVID'], reference='healthy')
```

## Evaluation signals

- Output table contains exactly 7 genes labeled up-regulated and 9 genes labeled down-regulated.
- Up-regulated gene symbols match the reported Figure 6H set: ADAMTS2, CD163, CEBPB, CTSD, LDLRAD3, NEAT1, TPST1 (contains_substring checks).
- Down-regulated gene symbols match the reported Figure 6H set: ATG77, CSF2RA, DST, FARP1, FRMD4A, HLA-DPB1, HLA-DRB1, RASAL2, TFRC.
- Every reported gene satisfies the differential-expression criteria in all three clusters (3, 5, and 12), i.e. the intersection across dependent clusters is non-empty and consistent.
- Per-cluster biomarker lists contain the expected top 8 genes per cluster matching Figure 6G.

## Limitations

- The consistency criterion across dependent clusters is defined by the article's Figure 6H selection; different top-N cutoffs may not reproduce the exact 7/9 gene split.
- The upstream novel Macrophage clusters depend on scNovel's detection; the article notes 'a better score function could improve scNovel performance', so cluster membership may vary and affect the DE result.
- The method does not incorporate additional biological information (e.g. chromatin accessibility data), as acknowledged by the authors as a limitation.

## Evidence

- [abstract] three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns through deeper analysis: "three potential novel subtypes of Macrophages are identified, where the COVID-related differential genes are also detected to have consistent expression patterns"
- [discussion] We developed a pipeline for the identification of novel rare cells and the analysis of differential gene expression.: "We developed a pipeline for the identification of novel rare cells and the analysis of differential gene expression."
- [readme] scanpy normalize_total/log1p preprocessing: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
- [results] seven genes with higher expression in COVID patients: [evidence span withheld — anchor status: not_found]
