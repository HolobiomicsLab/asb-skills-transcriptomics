---
name: jaccard-similarity-analysis-of-regulon-rankings
description: Use when two gene regulatory network inference tools (e.g., MINI-EX and
  SCENIC) have produced per-cell-type regulons and you need to quantify their overlap
  per cluster and validate specific regulon TF calls.
when_to_use_negative:
- Only one regulon set is available — Jaccard similarity requires two tools' regulon
  predictions to compare
- The dataset is not plant scRNA-seq with defined cell-type annotations, since MINI-EX/scPlant
  regulon inference and the bZIP44/ANAC030 reference calls are plant-specific
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: MINI-EX
  role: Nextflow DSL2 pipeline that infers cell-type-specific gene regulatory networks
    from plant scRNA-seq data; supplies one side of the comparison
  repo: https://github.com/VIB-PSB/MINI-EX
- name: scPlant
  role: Provides visualization functions (enrich_exp_hmp, topRegulons_MINIEX, BordaRank_MINIEX)
    for inspecting MINI-EX results and extracting top regulons per cluster
  repo: https://github.com/compbioNJU/scPlant
- name: pySCENIC
  role: Supplies the reference SCENIC regulons against which MINI-EX regulons are
    compared
  repo: aertslab/pySCENIC
- name: Seurat
  role: Produces the pre-processed Seurat object used as MINI-EX input via the scPlant
    auxiliary preprocessing script
- name: Nextflow
  role: Execution engine for the MINI-EX pipeline (nextflow -C miniex.config run miniex.nf)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:2bed84e5e3cc7079784535e711c1c617aa8218b862d29e2f66f7ef2b259ee9c3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compbioNJU/scPlant
  repo_url: https://github.com/VIB-PSB/MINI-EX
  tools:
  - MINI-EX
  - Nextflow
  - Seurat
  - pySCENIC
  - scPlant
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Jaccard similarity analysis of regulon rankings

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies agreement between cell-type-specific regulons inferred by MINI-EX and SCENIC on the Arabidopsis root single-cell atlas by computing Jaccard similarity of the top 30 regulons per cell cluster, verifying cross-tool consistency (bZIP44 in companion cells, ANAC030 in xylem) and the higher signal-to-noise ratio of MINI-EX.

## When to use

Use when two gene regulatory network inference tools (e.g., MINI-EX and SCENIC) have produced per-cell-type regulons and you need to quantify their overlap per cluster and validate specific regulon TF calls.

## When NOT to use

- Only one regulon set is available — Jaccard similarity requires two tools' regulon predictions to compare
- The dataset is not plant scRNA-seq with defined cell-type annotations, since MINI-EX/scPlant regulon inference and the bZIP44/ANAC030 reference calls are plant-specific

## Inputs

- Pre-processed Seurat object (SeuratObj.rds) of the Arabidopsis root single-cell atlas with seurat_clusters and celltype metadata
- MINI-EX per-cluster regulons in the regulons_output directory (from the Nextflow pipeline run)
- SCENIC-inferred cell-type-specific regulons from the prior GRN task

## Outputs

- regulon_jaccard_similarity.csv table of Jaccard similarities of top 30 regulons per cell cluster between MINI-EX and SCENIC
- Ranked top regulons per cluster by Borda ranking (topRegulons_MINIEX, BordaRank_MINIEX)
- Signal-to-noise comparison of regulon enrichment between MINI-EX and SCENIC reproducing Figure 6E-6F and Supplemental Figure 3

## How to apply

First run the MINI-EX pipeline (nextflow -C miniex.config run miniex.nf) on the pre-processed Seurat object to obtain per-cluster regulons in regulons_output, and have the SCENIC regulons from the prior GRN task available. Inspect MINI-EX results with scPlant visualization functions (enrich_exp_hmp, topRegulons_MINIEX, BordaRank_MINIEX) to extract the top regulons per cluster by Borda ranking. Compute the Jaccard similarity of the top 30 regulons per cell cluster against the SCENIC-inferred regulons and save the table as regulon_jaccard_similarity.csv. Verify cross-tool consistency by checking that bZIP44 appears among companion-cell regulons and ANAC030 among xylem regulons in both tools, and compare the regulon enrichment signal-to-noise ratio between MINI-EX and SCENIC, reproducing Figure 6E-6F and Supplemental Figure 3.

## Related tools

- **MINI-EX** (Nextflow DSL2 pipeline that infers cell-type-specific gene regulatory networks from plant scRNA-seq data; supplies one side of the comparison) — https://github.com/VIB-PSB/MINI-EX
- **scPlant** (Provides visualization functions (enrich_exp_hmp, topRegulons_MINIEX, BordaRank_MINIEX) for inspecting MINI-EX results and extracting top regulons per cluster) — https://github.com/compbioNJU/scPlant
- **pySCENIC** (Supplies the reference SCENIC regulons against which MINI-EX regulons are compared) — aertslab/pySCENIC
- **Seurat** (Produces the pre-processed Seurat object used as MINI-EX input via the scPlant auxiliary preprocessing script)
- **Nextflow** (Execution engine for the MINI-EX pipeline (nextflow -C miniex.config run miniex.nf))

## Examples

```
Rscript MINIEX_preprocess.R --seuratObj SeuratObj.rds --cluster seurat_clusters --celltype celltype -o INPUTS; nextflow -C miniex.config run miniex.nf
```

## Evaluation signals

- regulon_jaccard_similarity.csv is produced with Jaccard values for the top 30 regulons of every cell cluster
- bZIP44 is recovered among companion-cell regulons in both MINI-EX and SCENIC
- ANAC030 is recovered among xylem regulons in both MINI-EX and SCENIC
- MINI-EX regulon enrichment shows a higher signal-to-noise ratio than SCENIC, consistent with published Figure 6E-6F and Supplemental Figure 3

## Limitations

- Comparing only the top 30 regulons per cluster means lower-ranked regulons are excluded from the agreement quantification
- The Jaccard comparison depends on SCENIC regulons from a prior task being available and computed on the same Arabidopsis root atlas

## Evidence

- [methods] Borda ranking of top 30 regulons per cluster vs SCENIC: [evidence span withheld — anchor status: not_found]
- [results] bZIP44 and ANAC030 consistency check: [evidence span withheld — anchor status: not_found]
- [readme] MINI-EX pipeline execution command: [evidence span withheld — anchor status: external_doc]
- [other] scPlant regulon inspection functions: [evidence span withheld — anchor status: not_found]
