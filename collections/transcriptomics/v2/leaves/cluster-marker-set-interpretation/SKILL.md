---
name: cluster-marker-set-interpretation
description: Use when a scdrake pipeline run has produced per-cluster marker-gene
  tables and you need downstream biological interpretation of those marker sets via
  gene set enrichment (GO/KEGG) rather than plain marker lists.
when_to_use_negative:
- Your goal is within the standard scdrake scope (QC, filtering, normalization, HVG
  detection, dimensionality reduction, clustering, cell type annotation, marker detection,
  contrasts) — these are already provided by the released pipeline without an extension
  plan
- Input is not a cluster-level marker-gene set (e.g. you want single-cell-level scoring
  or differential expression between clusters, i.e. scdrake 'contrasts' — use the
  built-in contrast module instead)
- 'The organism of your data is not covered by the OrgDb configured in the main config
  (e.g. non-human data with ORGANISM: "human"), making symbol-to-ENTREZ mapping unreliable'
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scdrake
  role: Source pipeline whose per-cluster marker-gene tables, config (ORGANISM, BASE_OUT_DIR,
    CSS_FILE) and extensible drake plan the enrichment module builds on
  repo: https://github.com/bioinfocz/scdrake
- name: drake
  role: Pipeline framework in which the custom enrichment plan (plan_custom.R) is
    registered as additional targets so outputs are reproducible and up-to-date parts
    are skipped
  repo: https://github.com/bioinfocz/scdrake
- name: org.Hs.eg.db
  role: OrgDb annotation package used to map marker gene symbols to ENTREZ identifiers
    before GO/KEGG over-representation testing
- name: R
  role: Implementation language for the enrichment module and its HTML report rendering
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad089
    title: 'Scdrake: a reproducible and scalable pipeline for scRNA-seq data analysis'
  source_content_hash: sha256:1d87affa5dea1b0d5e28023c1b47ce52af6490c2333b3fc9d45efdfc80e0868c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinfocz/scdrake
  tools:
  - R
  - drake
  - org.Hs.eg.db
  - scdrake
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cluster-marker-set-interpretation

## Summary

Interprets per-cluster marker-gene sets from the scdrake scRNA-seq pipeline by running GO and KEGG over-representation enrichment per cluster and compiling results into a CSV table plus a standalone HTML report styled like scdrake's existing stage reports.

## When to use

Use when a scdrake pipeline run has produced per-cluster marker-gene tables and you need downstream biological interpretation of those marker sets via gene set enrichment (GO/KEGG) rather than plain marker lists.

## When NOT to use

- Your goal is within the standard scdrake scope (QC, filtering, normalization, HVG detection, dimensionality reduction, clustering, cell type annotation, marker detection, contrasts) — these are already provided by the released pipeline without an extension plan
- Input is not a cluster-level marker-gene set (e.g. you want single-cell-level scoring or differential expression between clusters, i.e. scdrake 'contrasts' — use the built-in contrast module instead)
- The organism of your data is not covered by the OrgDb configured in the main config (e.g. non-human data with ORGANISM: "human"), making symbol-to-ENTREZ mapping unreliable

## Inputs

- Per-cluster marker-gene table from the scdrake pipeline output directory (marker detection stage output)
- Main scdrake config specifying ORGANISM (e.g. "human")
- Pipeline stylesheet (CSS_FILE) for report rendering
- Existing scdrake drake plan object to be extended with plan_custom.R

## Outputs

- enrichment_results.csv with per-cluster rows: term ID, description, gene ratio, p-value, adjusted p-value
- enrichment_report.html standalone report styled with CSS_FILE under BASE_OUT_DIR

## How to apply

Load the per-cluster marker-gene table produced by the clustering/marker-detection stage of the scdrake pipeline from the pipeline output directory under BASE_OUT_DIR. Map the marker gene symbols to ENTREZ identifiers using the org.Hs.eg.db OrgDb selected via the ORGANISM: "human" main-config setting, since GO and KEGG over-representation testing operate on ENTREZ IDs. Run GO and KEGG enrichment separately for each cell cluster's marker set, collecting term ID, description, gene ratio, p-value, and adjusted p-value per term. Compile the per-cluster results into enrichment_results.csv and render a standalone enrichment_report.html using the pipeline's CSS_FILE stylesheet via a custom extension plan (plan_custom.R) registered as an additional drake target, so the output style matches existing stage reports. This is implemented as a drake plan extension because scdrake's pipeline definition is an R object that can be arbitrarily extended; gene set enrichment analysis is one of the modules the authors explicitly committed to as a future extension, so it is not part of the released pipeline out of the box.

## Related tools

- **scdrake** (Source pipeline whose per-cluster marker-gene tables, config (ORGANISM, BASE_OUT_DIR, CSS_FILE) and extensible drake plan the enrichment module builds on) — https://github.com/bioinfocz/scdrake
- **drake** (Pipeline framework in which the custom enrichment plan (plan_custom.R) is registered as additional targets so outputs are reproducible and up-to-date parts are skipped) — https://github.com/bioinfocz/scdrake
- **org.Hs.eg.db** (OrgDb annotation package used to map marker gene symbols to ENTREZ identifiers before GO/KEGG over-representation testing)
- **R** (Implementation language for the enrichment module and its HTML report rendering)

## Examples

```
Rscript -e 'source("plan_custom.R"); drake::make(plan)'
```

## Evaluation signals

- enrichment_report.html renders without error and uses the same CSS_FILE styling as existing scdrake stage reports under BASE_OUT_DIR
- enrichment_results.csv contains at least one row per tested cell cluster, with enrichment term identifiers and an adjusted p-value column present
- Every gene symbol in a cluster marker set is mapped to a valid ENTREZ identifier from org.Hs.eg.db before enrichment testing (unmapped symbols are accounted for)
- GO and KEGG tests are run as separate per-cluster analyses (not pooled across clusters), matching the workflow definition
- The enrichment targets rerun only when the upstream marker table changes, as expected from drake dependency tracking

## Limitations

- Gene set enrichment analysis was only a planned future module of scdrake (alongside trajectory inference and spatial transcriptomics support) — it must be implemented as a custom drake plan extension, not invoked from the released pipeline
- Symbol-to-ENTREZ mapping depends on the ORGANISM setting and the chosen OrgDb; marker symbols absent from the OrgDb will be silently lost, biasing gene ratios
- Over-representation testing is per-cluster on the marker list only, so it does not capture graded, gene-level quantitative signals the way full-rank GSEA would

## Evidence

- [abstract] detection of marker genes: "The pipeline includes quality control, cell and gene filtering, normalization, detection of highly variable genes, dimensionality reduction, clustering, cell type annotation, detection of marker"
- [readme] pipeline is extendable: [evidence span withheld — anchor status: external_doc]
- [readme] cluster markers and contrasts: [evidence span withheld — anchor status: external_doc]
- [methods] drake dependency tracking: "Because `{drake}` knows the inner relationships between targets in your plan, it also knows which targets are independent of each other"
