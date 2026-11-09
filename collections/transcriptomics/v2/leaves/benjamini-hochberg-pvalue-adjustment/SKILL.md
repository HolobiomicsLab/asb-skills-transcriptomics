---
name: benjamini-hochberg-pvalue-adjustment
description: Use when a per-cluster marker gene list has been tested against cell-type
  gene sets (e.g. easyct with test='fisher') and the family of candidate cell-type
  p-values within each cluster must be FDR-adjusted before ranking annotations.
when_to_use_negative:
- Input is not a family of p-values from a single hypothesis family (e.g. you need
  per-gene DE q-values rather than per-cell-type overrepresentation p-values)
- The annotation route is GSEA (easyct with test='gsea'), where ranking comes from
  enrichment scores rather than Fisher test p-values
- Adjusting across all clusters pooled together — the correction family is the candidate
  cell types within one cluster, not the whole marker table
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: EasyCellType
  role: R package whose modified Fisher's exact test produces the p-value family;
    its easyct(test='fisher') call applies the overrepresentation test whose p-values
    are BH-adjusted
  repo: https://github.com/rx-li/EasyCellType
- name: Fisher's exact test
  role: Statistical test whose per-candidate cell-type p-values form the family corrected
    by Benjamini–Hochberg
- name: CellMarker
  role: Cell-type gene set database (Human, blood/peripheral-blood tissue subset)
    against which the Fisher test is evaluated per cluster
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:e27871dab3a8f25a7f9179a81750fb05e59e0b99246692cb61cd5f74ad49a61d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - CellMarker
  - EasyCellType
  - Fisher's exact test
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# benjamini-hochberg-pvalue-adjustment

## Summary

Benjamini–Hochberg multiple-testing correction applied to the modified Fisher's exact test p-values in EasyCellType, computed within each cell cluster across candidate CellMarker cell types, so that overrepresentation calls are ranked by marker expression significance rather than raw overlap size.

## When to use

Use when a per-cluster marker gene list has been tested against cell-type gene sets (e.g. easyct with test='fisher') and the family of candidate cell-type p-values within each cluster must be FDR-adjusted before ranking annotations.

## When NOT to use

- Input is not a family of p-values from a single hypothesis family (e.g. you need per-gene DE q-values rather than per-cell-type overrepresentation p-values)
- The annotation route is GSEA (easyct with test='gsea'), where ranking comes from enrichment scores rather than Fisher test p-values
- Adjusting across all clusters pooled together — the correction family is the candidate cell types within one cluster, not the whole marker table

## Inputs

- Per-cluster marker table (e.g. gene_pbmc from EasyCellType package data) with Entrez IDs, cluster labels, and expression scores
- Family of raw Fisher's exact test p-values, one per candidate CellMarker cell type within each cluster

## Outputs

- BH-adjusted p-values per candidate cell type per cluster
- Cell-type candidates passing p_cut=0.3, ordered by marker expression score
- Per-cluster annotation table (e.g. fisher_annotations.csv) supporting top-1 and top-5 recommendations

## How to apply

In the EasyCellType workflow, per-cluster marker tables such as gene_pbmc (Entrez IDs, cluster labels, expression scores sorted in descending order) are passed to the modified Fisher's exact test (easyct with test='fisher') against CellMarker Human blood/peripheral-blood gene sets with p_cut=0.3. Because the Fisher test is run once per candidate cell type per cluster, the resulting p-value family is a per-cluster set of candidate cell types; apply Benjamini–Hochberg across that family within each cluster rather than pooling clusters. After adjustment, retain cell types whose adjusted p-values pass the p_cut cutoff and order the surviving candidates by the markers' expression score (e.g. log fold change) significance, not by raw overlap size — the paper warns that simply overlapping lists without considering gene ranking leads to unreliable results. The corrected, score-sorted table then feeds top-1 (hard) and top-5 (soft) annotation recommendations and can be written out as fisher_annotations.csv.

## Related tools

- **EasyCellType** (R package whose modified Fisher's exact test produces the p-value family; its easyct(test='fisher') call applies the overrepresentation test whose p-values are BH-adjusted) — https://github.com/rx-li/EasyCellType
- **Fisher's exact test** (Statistical test whose per-candidate cell-type p-values form the family corrected by Benjamini–Hochberg)
- **CellMarker** (Cell-type gene set database (Human, blood/peripheral-blood tissue subset) against which the Fisher test is evaluated per cluster)

## Examples

```
data("gene_pbmc"); annot.fisher <- easyct(gene_pbmc, db="cellmarker", species="Human", tissue="blood", test="fisher")
```

## Evaluation signals

- All adjusted p-values fall in [0,1] and the BH adjustment preserves monotonicity with the raw p-values (lower raw p ⇒ lower or equal adjusted p)
- The number of candidate cell types passing p_cut=0.3 after adjustment is at most the number passing on raw p-values, and ordering among passing candidates follows the markers' expression score, not raw overlap size
- Reproducibility check: re-running easyct(input.d, db='cellmarker', test='fisher') on gene_pbmc regenerates the same adjusted p-values and top-ranked annotations
- Top-ranked per-cluster annotations are consistent with known PBMC populations as displayed by the package's dot-plot summary
- The output annotation table is emitted per cluster with hard (top-1) and soft (top-5) ranked cell types

## Limitations

- Simply overlapping gene lists without considering gene ranking might lead to unreliable results, so the adjusted p-values alone are insufficient — score-based ordering of passing candidates is required
- The EasyCellType package documentation only names '3 public available marker gene data bases' without fully enumerating tissue database names, limiting database-family coverage
- No software version numbers are recorded in the methods, so exact p-value implementation details must be checked against the installed package

## Evidence

- [abstract] modified Fisher's test p-values are the corrected family: "The package provides two statistical tests, gene set enrichment analysis and a modified version of Fisher’s exact test"
- [intro] ranking must combine significance with expression score: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance"
- [abstract] overlap without ranking is unreliable: "Additionally, simply overlapping the two lists without considering gene ranking might lead to unreliable results."
- [methods] easyct call against CellMarker: "annot.GSEA <- easyct(input.d, db="cellmarker", species="Human","
- [readme] package scope and tests from README: "Two quantification approaches are provided to annotate cell types: Gene set 
enrichment analysis (GSEA) and a fisher exact test combining with expression 
significance."
