---
name: tuft-cell-marker-gene-enrichment
description: Use when a GapClust (or similar) rare-cluster detection run on scRNA-seq
  data yields candidate minor clusters whose identity must be confirmed by enrichment
  of known marker genes (e.g. Dclk1/Trpm5 for tuft cells).
when_to_use_negative:
- Input is not a raw scRNA-seq count matrix (e.g. already-normalized, aggregated,
  or bulk expression data), since GapClust operates on cell-level expression profiles
- The goal is to detect rare clusters de novo without any known marker genes available
  for identity validation
- Rare cell types of interest have already been identified and removed — per the article,
  GapClust should be rerun to discover additional hidden clusters rather than revalidating
  known ones
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GapClust
  role: Detects the rare cell clusters (e.g. 29-cell R2) from the intestine crypt
    expression profiles using nearest-neighbour gap statistics; installs via devtools::install_github("fabotao/GapClust")
  repo: https://github.com/fabotao/GapClust
- name: Seurat
  role: Provides FindMarkers-style differential expression to test enrichment of Dclk1
    and Trpm5 in the GapClust-detected rare cluster; requires Seurat >= 3.1.0
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:972aa426d028cd9747a6dda8c8c51bcfb69b38a12696ea5db64e409ed05b385a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabotao/GapClust
  tools:
  - GapClust
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tuft-cell-marker-gene-enrichment

## Summary

Validates rare-cell clusters detected by GapClust — such as the 29-cell R2 tuft-cell cluster in the 4491-cell non-irradiated intestinal crypt dataset from Arshad et al. — by checking enrichment of established marker genes (Dclk1, Trpm5) via Seurat-style differential expression.

## When to use

Use when a GapClust (or similar) rare-cluster detection run on scRNA-seq data yields candidate minor clusters whose identity must be confirmed by enrichment of known marker genes (e.g. Dclk1/Trpm5 for tuft cells).

## When NOT to use

- Input is not a raw scRNA-seq count matrix (e.g. already-normalized, aggregated, or bulk expression data), since GapClust operates on cell-level expression profiles
- The goal is to detect rare clusters de novo without any known marker genes available for identity validation
- Rare cell types of interest have already been identified and removed — per the article, GapClust should be rerun to discover additional hidden clusters rather than revalidating known ones

## Inputs

- scRNA-seq count matrix of 4491 non-irradiated intestine crypt cells (Arshad et al.), pandas/Scanpy-compatible IO
- GapClust rare-cluster detection output (cluster IDs, memberships, cluster sizes, e.g. 29-cell R2 cluster)
- Known marker gene list (Dclk1, Trpm5) from Arshad et al.

## Outputs

- Rare-cluster membership table (cell barcode, cluster ID, cluster size, marker status) written to gapclust_rare_cells.csv
- Differential-expression marker table for the rare cluster vs. remaining cells, confirming Dclk1/Trpm5 enrichment

## How to apply

Run GapClust on the scRNA-seq count matrix (e.g. 4491 non-irradiated intestine crypt cells) using nearest-neighbour distances and the second-order derivative gap score, scanning candidate k and retaining candidates where skewness<k> > 2. Record the sizes of detected rare clusters and extract the membership of the target cluster (e.g. the 29-cell R2 cluster). Then perform Seurat/FindMarkers-style differential expression comparing cluster R2 cells against all remaining cells, and inspect the top marker genes for tuft-cell markers Dclk1 and Trpm5. The rationale, per Fa et al. (Nat Commun 2021), is that GapClust detects rare clusters purely from expression-profile geometry, so known biological markers provide independent validation of cluster identity as reported for the intestinal dataset (matching Fig. 5a-c). Write the membership table (cell barcode, cluster ID, cluster size, marker status) to gapclust_rare_cells.csv for traceability.

## Related tools

- **GapClust** (Detects the rare cell clusters (e.g. 29-cell R2) from the intestine crypt expression profiles using nearest-neighbour gap statistics; installs via devtools::install_github("fabotao/GapClust")) — https://github.com/fabotao/GapClust
- **Seurat** (Provides FindMarkers-style differential expression to test enrichment of Dclk1 and Trpm5 in the GapClust-detected rare cluster; requires Seurat >= 3.1.0)

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- The output must contain an R2 cluster of exactly 29 cells among the 4491 non-irradiated intestine crypt cells
- Top marker genes of cluster R2 must include the tuft-cell markers Dclk1 and Trpm5, consistent with Arshad et al. and Fig. 5a-c
- GapClust parameter scan reports candidate k values flagged by skewness<k> > 2, and R2 appears among the detected rare clusters
- gapclust_rare_cells.csv parses cleanly with columns cell barcode, cluster ID, cluster size, marker status
- GapClust applied to the intestine dataset should yield three rare cell types, matching the reported finding

## Limitations

- No parameter settings for GapClust runs (e.g. K, skewness threshold) are explicitly reported for the intestinal analysis, so values like skewness<k> > 2 must be adopted from the method description
- Validation depends on prior biological knowledge of marker genes (Dclk1, Trpm5); clusters lacking known markers cannot be confirmed this way
- Hidden clusters require rerunning GapClust with previously identified rare cell types removed before additional discoveries can be made

## Evidence

- [discussion] Tuft cells are identified in the intestinal dataset, validated by existing maker genes including Dclk1 and Trpm5: "Tuft cells are identified in the intestinal dataset, validated by existing maker genes including Dclk1 and Trpm5"
- [abstract] GapClust identifies tuft cells in the intestine dataset: "When applying our algorithm to an intestine and 68 k PBMC datasets, GapClust identifies the tuft cells"
- [methods] Candidate k identified by skewness threshold: "Candidate k can be identified if the skewness<k> > 2."
- [readme] GapClust design principle from README: "GapClust takes advantage of the gap between minor cluster and neighbouring abundant cluster to let rare cells within minor cluster stand out"
