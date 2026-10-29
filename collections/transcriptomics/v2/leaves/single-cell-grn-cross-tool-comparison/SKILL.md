---
name: single-cell-grn-cross-tool-comparison
description: Use when two gene regulatory network inference tools (e.g., MINI-EX and
  SCENIC) have been run on the same single-cell atlas and their cell-type-specific
  regulon outputs must be compared and validated.
when_to_use_negative:
- Only one GRN inference tool has been run and no second tool's regulon output exists
  for cross-comparison
- The input is bulk transcriptomic data rather than a cell-clustered single-cell atlas,
  since both MINI-EX and the comparison workflow require cell-type-specific regulons
- The organism's cisTarget databases or TF motif resources (e.g., from JASPAR or PlantTFDB)
  are unavailable, so regulons cannot be inferred for the species
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: MINI-EX
  role: Nextflow DSL2 pipeline used to infer cell-type-specific gene regulatory networks
    from plant scRNA-seq data; its per-cluster regulon output is one side of the cross-tool
    comparison
  repo: https://github.com/VIB-PSB/MINI-EX
- name: scPlant
  role: Framework whose auxiliary script (MINIEX_preprocess.R) prepares MINI-EX inputs
    and whose visualization functions (enrich_exp_hmp, topRegulons_MINIEX, BordaRank_MINIEX)
    extract top regulons per cluster
  repo: https://github.com/compbioNJU/scPlant
- name: Seurat
  role: R package underlying scPlant's basic data processing; provides the pre-processed
    Seurat object that is converted into MINI-EX input files
- name: pySCENIC
  role: Python implementation of the SCENIC pipeline whose predicted regulons serve
    as the reference for Jaccard-similarity comparison against MINI-EX
  repo: https://github.com/aertslab/pySCENIC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:991dba37fd7da5487e05a4e0aa364f265c9c38f6f8ae5d5a4e2f37143456405c
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
  - Seurat
  - pySCENIC
  - scPlant
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# single-cell-grn-cross-tool-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the cross-tool comparison of cell-type-specific regulons inferred by MINI-EX and SCENIC on the Arabidopsis root single-cell atlas, quantifying agreement via Jaccard similarity of top regulons and confirming MINI-EX's higher signal-to-noise ratio of regulon enrichment.

## When to use

Use when two gene regulatory network inference tools (e.g., MINI-EX and SCENIC) have been run on the same single-cell atlas and their cell-type-specific regulon outputs must be compared and validated.

## When NOT to use

- Only one GRN inference tool has been run and no second tool's regulon output exists for cross-comparison
- The input is bulk transcriptomic data rather than a cell-clustered single-cell atlas, since both MINI-EX and the comparison workflow require cell-type-specific regulons
- The organism's cisTarget databases or TF motif resources (e.g., from JASPAR or PlantTFDB) are unavailable, so regulons cannot be inferred for the species

## Inputs

- Pre-processed Seurat object (SeuratObj.rds) of the Arabidopsis root single-cell atlas with seurat_clusters and celltype metadata
- SCENIC-inferred regulons per cell cluster from the prior GRN task
- MINI-EX config file (miniex.config) with generated input paths
- cisTarget databases for the relevant plant genome (e.g., Arabidopsis TAIR10)

## Outputs

- Per-cluster regulons from MINI-EX (regulons_output directory)
- regulon_jaccard_similarity.csv: Jaccard similarity of top 30 regulons per cell cluster between MINI-EX and SCENIC
- Signal-to-noise ratio comparison of regulon enrichment between MINI-EX and SCENIC (reproducing Figure 6E-6F and Supplemental Figure 3)

## How to apply

First produce MINI-EX input files from the pre-processed Seurat object using the scPlant auxiliary script (MINIEX_preprocess.R, specifying --cluster seurat_clusters and --celltype celltype), then run the MINI-EX Nextflow pipeline to obtain per-cluster regulons in the regulons_output directory. Inspect MINI-EX results with scPlant visualization functions (enrich_exp_hmp, topRegulons_MINIEX, BordaRank_MINIEX) to extract the top 30 regulons per cell cluster by Borda ranking. Compute the Jaccard similarity of these top-30 regulon sets against the regulons inferred by SCENIC in the prior GRN task, saving the result as regulon_jaccard_similarity.csv. Verify cross-tool consistency by checking that bZIP44 appears among companion-cell regulons and ANAC030 among xylem regulons in both tools, and compare the regulon enrichment signal-to-noise ratio between MINI-EX and SCENIC to reproduce Figure 6E-6F and Supplemental Figure 3.

## Related tools

- **MINI-EX** (Nextflow DSL2 pipeline used to infer cell-type-specific gene regulatory networks from plant scRNA-seq data; its per-cluster regulon output is one side of the cross-tool comparison) — https://github.com/VIB-PSB/MINI-EX
- **scPlant** (Framework whose auxiliary script (MINIEX_preprocess.R) prepares MINI-EX inputs and whose visualization functions (enrich_exp_hmp, topRegulons_MINIEX, BordaRank_MINIEX) extract top regulons per cluster) — https://github.com/compbioNJU/scPlant
- **Seurat** (R package underlying scPlant's basic data processing; provides the pre-processed Seurat object that is converted into MINI-EX input files)
- **pySCENIC** (Python implementation of the SCENIC pipeline whose predicted regulons serve as the reference for Jaccard-similarity comparison against MINI-EX) — https://github.com/aertslab/pySCENIC

## Examples

```
Rscript MINIEX_preprocess.R --seuratObj SeuratObj.rds --cluster seurat_clusters --celltype celltype -o INPUTS && nextflow -C miniex.config run miniex.nf
```

## Evaluation signals

- regulon_jaccard_similarity.csv confirms recovery of bZIP44 among companion-cell regulons and ANAC030 among xylem regulons in the expected cell types
- Jaccard similarity values of the top 30 regulons per cluster show substantial agreement between MINI-EX and SCENIC, consistent with the published figures
- MINI-EX regulon enrichment exhibits a higher signal-to-noise ratio than SCENIC, matching Figure 6E-6F and Supplemental Figure 3
- regulons_output directory from the Nextflow run contains per-cluster regulons for all expected seurat_clusters
- Borda ranking from topRegulons_MINIEX/BordaRank_MINIEX yields a coherent top-30 regulon set per cluster that can be matched against SCENIC outputs

## Limitations

- MINI-EX is distributed under a dual license (proprietary model as well as open source model), which may constrain some reuse
- The comparison depends on prior construction of cisTarget databases, which the create_cisTarget_databases README notes can be very memory intensive (e.g., ~37 GB RAM for a 1,000,000-region rankings database with 10000 motifs)
- Consistency of bZIP44 and ANAC030 is validated specifically on the Arabidopsis thaliana root atlas (Zhang et al., 2019); other datasets may yield different regulon agreements

## Evidence

- [results] MINI-EX and SCENIC predicted consistent cell-type-specific regulons: both tools identified bZIP44 as a specific regulon TF for companion cells and ANAC030 as a regulon TF for xylem cells in the Arabidopsis root atlas.: [evidence span withheld — anchor status: not_found]
- [methods] Produce MINI-EX input files from the pre-processed Seurat object with the scPlant auxiliary script.: [evidence span withheld — anchor status: not_found]
- [methods] Compute Jaccard similarity of the top 30 regulons per cell cluster against the regulons inferred by SCENIC.: [evidence span withheld — anchor status: not_found]
- [readme] MINI-EX uses a dual license to offer the distribution of the software under a proprietary model as well as an open source model.: [evidence span withheld — anchor status: external_doc]
- [readme] MINI-EX pipeline purpose is to infer cell-type specific gene regulatory network using scRNA-Seq data in plants.: [evidence span withheld — anchor status: external_doc]
