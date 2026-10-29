---
name: cell-marker-database-querying
description: Use when you have a per-cluster marker table (Entrez IDs, cluster labels,
  expression scores such as logFC from Seurat's FindAllMarkers) and need automated
  cell-type annotations from public marker databases.
when_to_use_negative:
- Input is a raw count matrix or Seurat object without a per-cluster marker table
  — run clustering and FindAllMarkers first (only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25)
  before querying marker databases
- The cell type of interest lies outside the tissues covered by CellMarker/PanglaoDB/Clustermole,
  or the species is not Human/Mouse for the selected database
- You need to overlap gene lists without ranking information — the article warns this
  can lead to unreliable annotation results
tools:
- name: EasyCellType
  role: Runs the database query via easyct() against CellMarker/PanglaoDB/Clustermole
    with GSEA or modified Fisher's exact test and p_cut=0.3; provides plot_dot/plot_bar
    summaries
  repo: https://github.com/rx-li/EasyCellType
- name: clusterProfiler
  role: Performs the gene set enrichment analysis backend for test='GSEA' in easyct
- name: Seurat
  role: Source of the input marker lists via FindAllMarkers on the clustered PBMC
    data
- name: scCATCH
  role: Alternative marker-database annotation toolkit (findmarkergene/findcelltype
    over CellMatch) used as a comparison point
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:f8504765b6bbff2b95ba16bf6b5c19e1102f047064e65209946a219e6888187e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - CellMarker
  - EasyCellType
  - Seurat
  - clusterProfiler
  - scCATCH
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cell-marker-database-querying

## Summary

Queries public cell-marker gene databases (CellMarker, PanglaoDB, Clustermole) with a per-cluster differential marker table to produce ranked cell-type annotation recommendations, using EasyCellType's GSEA or score-weighted modified Fisher's exact test.

## When to use

Use when you have a per-cluster marker table (Entrez IDs, cluster labels, expression scores such as logFC from Seurat's FindAllMarkers) and need automated cell-type annotations from public marker databases.

## When NOT to use

- Input is a raw count matrix or Seurat object without a per-cluster marker table — run clustering and FindAllMarkers first (only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) before querying marker databases
- The cell type of interest lies outside the tissues covered by CellMarker/PanglaoDB/Clustermole, or the species is not Human/Mouse for the selected database
- You need to overlap gene lists without ranking information — the article warns this can lead to unreliable annotation results

## Inputs

- Per-cluster differential marker table (e.g. FindAllMarkers output from Seurat) with Entrez IDs, cluster labels and expression scores
- gene_pbmc processed marker table shipped in the EasyCellType package
- Tissue keyword tables cellmarker_tissue, panglao_tissue, clustermole_tissue from the EasyCellType package

## Outputs

- Per-cluster ranked cell-type annotation recommendations (hard top-1 and soft top-5)
- fisher_annotations.csv per-cluster annotation table
- annotation_accuracy_comparison.csv database/test accuracy table
- Dot-plot (plot_dot) and bar-plot (plot_bar) visualizations of per-cluster recommendations

## How to apply

Format the marker table as an EasyCellType input data frame of Entrez IDs, cluster labels and expression scores, sorting genes by score in descending order within each cluster. Inspect available tissue keywords with data(cellmarker_tissue), data(panglao_tissue) and data(clustermole_tissue), then apply a tissue filter (e.g. blood/peripheral-blood for PBMCs) to restrict candidate cell types. Call easyct with db='cellmarker', species='Human', p_cut=0.3, choosing test='GSEA' (enrichment scoring by clusterProfiler) or test='fisher' (modified Fisher's exact test combined with expression significance). For the Fisher route, adjust p-values across candidate cell types with Benjamini-Hochberg within each cluster and order passing candidates by marker expression score rather than raw overlap size. The ranking-aware design matters: the paper notes that simply overlapping the two lists without considering gene ranking might lead to unreliable results.

## Related tools

- **EasyCellType** (Runs the database query via easyct() against CellMarker/PanglaoDB/Clustermole with GSEA or modified Fisher's exact test and p_cut=0.3; provides plot_dot/plot_bar summaries) — https://github.com/rx-li/EasyCellType
- **clusterProfiler** (Performs the gene set enrichment analysis backend for test='GSEA' in easyct)
- **Seurat** (Source of the input marker lists via FindAllMarkers on the clustered PBMC data)
- **scCATCH** (Alternative marker-database annotation toolkit (findmarkergene/findcelltype over CellMatch) used as a comparison point) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
annot.GSEA <- easyct(input.d, db="cellmarker", species="Human", tissue="blood", test="GSEA", p_cut=0.3)
```

## Evaluation signals

- On the 2400-cell PBMC simulation, the annotation_accuracy_comparison.csv must reproduce CellMarker attaining the highest and PanglaoDB the lowest accuracy under both test='GSEA' and test='fisher'
- On gene_pbmc with easyct(input.d, db='cellmarker', test='fisher'), top-ranked annotations must be consistent with known PBMC populations as displayed by plot_dot
- Each cluster's input markers must be sorted in descending rank order by expression score before testing
- Fisher p-values must be Benjamini-Hochberg adjusted across candidate cell types within each cluster before the p_cut=0.3 filter
- Output must contain hard (top-1) and soft (top-5) ranked recommendations per cluster, not just an unranked overlap list

## Limitations

- Database choice materially changes accuracy: CellMarker achieved the highest accuracy on the 2400-cell PBMC simulation, especially with GSEA, while PanglaoDB was lowest under both tests
- The paper notes 'simply overlapping the two lists without considering gene ranking might lead to unreliable results', so ranking-aware tests are essential
- Tissue filtering depends on manually selected tissue keywords (blood/peripheral-blood for PBMCs); the paper only fully enumerates '3 public available marker gene data bases' without naming all tissue subsetting options
- Software version numbers and the full tissue database names were not documented in the methods section

## Evidence

- [intro] Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance"
- [methods] easyct call against CellMarker: "annot.GSEA <- easyct(input.d, db="cellmarker", species="Human","
- [methods] Three public marker databases: "The package refers to 3 public available marker gene data bases,"
- [abstract] Automatic database querying with graphical outcomes: "which automatically checks an input marker list obtained by differential expression analysis against the databases and provides annotation recommendations in graphical outcomes"
- [readme] README on package purpose and tests: "EasyCellType is a R package developed to automatically examine the input marker
lists obtained from existing software such as Seurat over the cell marker databases."
