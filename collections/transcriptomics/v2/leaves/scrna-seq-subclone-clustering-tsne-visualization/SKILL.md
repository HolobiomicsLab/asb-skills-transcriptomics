---
name: scrna-seq-subclone-clustering-tsne-visualization
description: Use when deposited scRNA-seq count matrices (e.g., drug-sensitive vs.
  resistant HMCL pairs like U266P vs U266VR) must be clustered into subclones and
  screened for drug-target gene enrichment to prioritize combination therapies.
when_to_use_negative:
- The goal is in silico secDrug prediction from pharmacogenomics data via the greedy
  set cover algorithm, not single-cell target-gene enrichment screening.
- The input is bulk RNA-seq or CyTOF proteomics data rather than single-cell transcriptomic
  count matrices requiring clustering and t-SNE embedding.
- The candidate drug combination's target genes are not known or mappable onto the
  scRNA-seq feature space, so enrichment cannot be computed.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0154
tools:
- name: R
  role: Implementation language for the Seurat-based preprocessing, clustering, t-SNE
    embedding, and enrichment analysis workflow
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: secDrug
  role: Repository providing deposited HMCL scRNA-seq count matrices and LINCS-derived
    target-gene sets used for the enrichment screen
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: Package exporting AssignGroup(x, l) to bin per-cluster target-gene expression
    and PropTest(p1, p2, n) for between-cluster proportion testing
  repo: github.com/Ujjal-Mukherjee/secDrug
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:b3451caf8226be50d0f6f03a311424ed3b4961e9f3a9d1c8f1887b2f16c29594
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CombinationDrugMyeloma
  - R
  - secDrug
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# scrna-seq-subclone-clustering-tsne-visualization

## Summary

A single-cell RNA-seq screening skill that preprocesses and clusters myeloma cell-line scRNA-seq count matrices with Seurat, embeds subclones via t-SNE, and maps LINCS-derived drug-target genes (HSP90AA1/HSP90AB1 for 17-AAG, NAMPT for FK866) onto subclonal clusters to rank secDrug combination candidates.

## When to use

Use when deposited scRNA-seq count matrices (e.g., drug-sensitive vs. resistant HMCL pairs like U266P vs U266VR) must be clustered into subclones and screened for drug-target gene enrichment to prioritize combination therapies.

## When NOT to use

- The goal is in silico secDrug prediction from pharmacogenomics data via the greedy set cover algorithm, not single-cell target-gene enrichment screening.
- The input is bulk RNA-seq or CyTOF proteomics data rather than single-cell transcriptomic count matrices requiring clustering and t-SNE embedding.
- The candidate drug combination's target genes are not known or mappable onto the scRNA-seq feature space, so enrichment cannot be computed.

## Inputs

- Deposited HMCL scRNA-seq count matrices (e.g., U266P drug-sensitive vs U266VR drug-resistant pairs) from github.com/Ujjal-Mukherjee/secDrug
- LINCS-derived drug-target gene lists (17-AAG targets: HSP90AA1, HSP90AB1; FK866 target: NAMPT)

## Outputs

- Per-sample t-SNE cluster map annotated with target-gene enrichment (reproducing Fig. 2)
- CSV table of target-gene enrichment scores per subclone per sample
- Between-cluster proportion test results for target-gene enrichment

## How to apply

Load the deposited HMCL scRNA-seq count matrices (e.g., U266P vs U266VR pairs) from github.com/Ujjal-Mukherjee/secDrug into R. Preprocess each sample with Seurat — QC filtering and normalization — then cluster and embed subclones using t-SNE dimensionality reduction. Map the LINCS-derived 17-AAG target genes (HSP90AA1, HSP90AB1) and the FK866 target gene (NAMPT) onto the t-SNE-defined subclonal clusters of each sample. Use the secDrug/CombinationDrugMyeloma package functions — AssignGroup(x, l) to bin per-cluster target-gene expression into groups and PropTest(p1, p2, n) for between-cluster proportion testing — to compute target-gene enrichment per subclone per sample. Export per-sample t-SNE cluster target-gene enrichment maps as CSV enrichment-score tables plus an annotated t-SNE figure reproducing Fig. 2.

## Related tools

- **R** (Implementation language for the Seurat-based preprocessing, clustering, t-SNE embedding, and enrichment analysis workflow) — github.com/Ujjal-Mukherjee/secDrug
- **secDrug** (Repository providing deposited HMCL scRNA-seq count matrices and LINCS-derived target-gene sets used for the enrichment screen) — github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (Package exporting AssignGroup(x, l) to bin per-cluster target-gene expression and PropTest(p1, p2, n) for between-cluster proportion testing) — github.com/Ujjal-Mukherjee/secDrug

## Examples

```
library(CombinationDrugMyeloma); grp <- AssignGroup(cluster_expr, target_genes); pt <- PropTest(p1, p2, n)
```

## Evaluation signals

- The majority of single-cell subclonal clusters show enrichment for HSP90AA1/HSP90AB1 and NAMPT target genes, matching the pattern reported in Fig. 2 of the paper.
- Seurat preprocessing (QC filtering, normalization) yields per-sample subclonal clusters that are visually separable on the t-SNE embedding.
- Exported enrichment-score CSV contains one score per subclone per sample with no missing cluster entries.
- AssignGroup output bins every cluster into an expression group, and PropTest returns a valid between-cluster proportion comparison (p1, p2, n) for each sample pair.

## Limitations

- The screen is a prediction tool: enrichment of target genes predicts combination efficacy, but in vitro/ex vivo validation is still required, as performed for 17-AAG + FK866.
- The pipeline currently supports testing two-drug combinations; model systems for testing more than two secDrugs are not yet developed.
- Validation in a wider panel of HMCLs representing refractory and clonally derived acquired resistant cell lines is not yet done.

## Evidence

- [abstract] scRNAseq as screening tool: "we used single-cell RNA sequencing (scRNAseq) as a screening tool to predict top combination candidates based on the enrichment of target genes"
- [methods] AssignGroup function: "- `AssignGroup(x, l)`"
- [methods] PropTest function: "- `PropTest(p1, p2, n)`"
- [intro] subclonal expression research question: "we introduced single-cell transcriptomics as a novel screening tool for prioritizing secDrug combinations based on the subclonal expression of the drug targets"
- [discussion] pipeline limitation to two-drug combinations: "A logical extension of this pipeline would be the development of model systems where a combination of more than two secDrugs can be effectively tested."
