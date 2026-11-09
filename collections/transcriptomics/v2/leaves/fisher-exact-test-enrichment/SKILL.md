---
name: fisher-exact-test-enrichment
description: Use when a per-cluster differential marker list (genes + cluster labels
  + expression scores such as logFC) needs to be converted into ranked cell-type annotations
  via overrepresentation testing against a marker database.
when_to_use_negative:
- The analysis goal is rank-aware enrichment of a full ordered gene list rather than
  marker overrepresentation testing — EasyCellType's GSEA option (annot.GSEA <- easyct(input.d,
  db="cellmarker", ...)) is the alternative test in that case.
- The input is raw counts or an unprocessed Seurat object with no per-cluster marker
  list — the upstream Seurat workflow (FindAllMarkers, mapIds to Entrez IDs) must
  run first.
- Markers come from a tissue with no matching CellMarker tissue subset, since annotation
  quality depends on tissue-specific marker gene sets.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: EasyCellType
  role: R package providing the modified Fisher's exact test (easyct with test='fisher'),
    BH adjustment, score-based ranking, and dot-plot visualization of annotation results
  repo: https://github.com/rx-li/EasyCellType
- name: Seurat
  role: Upstream source of the per-cluster differential marker lists (FindAllMarkers)
    that feed the Fisher test
- name: CellMarker
  role: Reference database of cell-type marker gene sets queried for overrepresentation
    (db='cellmarker')
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:914cba73ee9b5dde4a69b27427087e613cc7b79e803df3cd111dc8bea889fb4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - CellMarker
  - EasyCellType
  - Seurat
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# fisher-exact-test-enrichment

## Summary

EasyCellType's modified Fisher's exact test assesses overrepresentation of a cluster's differential marker genes within cell-type marker gene sets (e.g. CellMarker), then ranks passing candidates by the markers' expression significance (e.g. log fold change) rather than raw overlap size, yielding per-cluster cell-type annotation recommendations.

## When to use

Use when a per-cluster differential marker list (genes + cluster labels + expression scores such as logFC) needs to be converted into ranked cell-type annotations via overrepresentation testing against a marker database.

## When NOT to use

- The analysis goal is rank-aware enrichment of a full ordered gene list rather than marker overrepresentation testing — EasyCellType's GSEA option (annot.GSEA <- easyct(input.d, db="cellmarker", ...)) is the alternative test in that case.
- The input is raw counts or an unprocessed Seurat object with no per-cluster marker list — the upstream Seurat workflow (FindAllMarkers, mapIds to Entrez IDs) must run first.
- Markers come from a tissue with no matching CellMarker tissue subset, since annotation quality depends on tissue-specific marker gene sets.

## Inputs

- Per-cluster differential marker table with Entrez IDs, cluster labels, and expression scores (e.g. data(gene_pbmc) from EasyCellType, derived from Seurat FindAllMarkers output)
- CellMarker cell-type gene sets filtered by species (Human) and tissue subset (blood/peripheral blood)

## Outputs

- Per-cluster ranked cell-type annotation table (e.g. fisher_annotations.csv)
- Top-1 (hard) and top-5 (soft) cell-type recommendations per cluster
- BH-adjusted Fisher test p-values per candidate cell type

## How to apply

Load a processed per-cluster marker table such as gene_pbmc from the EasyCellType package, which carries Entrez IDs, cluster labels, and expression scores sorted in descending rank order within each cluster. For each cluster, run the modified Fisher's exact test (easyct with test='fisher') against CellMarker cell-type gene sets for the chosen species (Human) and tissue subset (blood/peripheral blood), using p_cut=0.3 as the cutoff for candidate marker-set enrichment. Adjust Fisher p-values across candidate cell types within each cluster using Benjamini–Hochberg. Rank the passing candidates by the markers' expression score significance rather than overlap size — the article explicitly warns that simply overlapping gene lists without considering gene ranking might lead to unreliable results — then emit top-1 (hard) and top-5 (soft) annotation recommendations per cluster and write the per-cluster table (e.g. fisher_annotations.csv).

## Related tools

- **EasyCellType** (R package providing the modified Fisher's exact test (easyct with test='fisher'), BH adjustment, score-based ranking, and dot-plot visualization of annotation results) — https://github.com/rx-li/EasyCellType
- **Seurat** (Upstream source of the per-cluster differential marker lists (FindAllMarkers) that feed the Fisher test)
- **CellMarker** (Reference database of cell-type marker gene sets queried for overrepresentation (db='cellmarker'))

## Examples

```
annot.fisher <- easyct(input.d = gene_pbmc, db = "cellmarker", species = "Human", tissue = "blood", test = "fisher", p_cut = 0.3)
```

## Evaluation signals

- Reproducibility: re-running easyct(input.d, db='cellmarker', test='fisher') on gene_pbmc should regenerate the same per-cluster annotation table.
- p-value adjustment check: adjusted p-values across candidate cell types within each cluster follow Benjamini–Hochberg ordering.
- Ranking sanity: top-ranked annotations are ordered by the markers' expression score significance (logFC), not by overlap count.
- Biological consistency: top-ranked annotations match known PBMC populations when displayed with the package's dot-plot summary.
- Output schema: fisher_annotations.csv contains one row set per cluster with ranked candidate cell types.

## Limitations

- Simply overlapping the two gene lists without considering gene ranking might lead to unreliable results — ranking-aware selection is essential to this test.
- Annotation quality depends on the marker database coverage; the package refers to 3 public available marker gene data bases and results vary by tissue subset chosen (here blood/peripheral blood).
- The Fisher test path needs Entrez-mapped markers (mapIds with org.Hs.eg.db) and an adequate p_cut; the demonstrated setting is p_cut=0.3.
- Results should still be reviewed by the researcher using the package's visualizations before picking final annotations.

## Evidence

- [abstract] two statistical tests offered: "The package provides two statistical tests, gene set enrichment analysis and a modified version of Fisher’s exact test"
- [abstract] ranking caveat: "Additionally, simply overlapping the two lists without considering gene ranking might lead to unreliable results."
- [intro] the two approaches: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance"
- [methods] easyct invocation pattern: "annot.GSEA <- easyct(input.d, db="cellmarker", species="Human","
- [other] easyct is the entry function; fisher is a test option per the task workflow (test_fisher / easyct with test='fisher'): [evidence span withheld — anchor status: not_found]
