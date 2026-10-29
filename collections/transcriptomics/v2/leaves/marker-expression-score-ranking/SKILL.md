---
name: marker-expression-score-ranking
description: Use when converting per-cluster differential marker lists (e.g. Seurat
  FindAllMarkers output with Entrez IDs and scores) into ranked cell-type annotations,
  where significance-adjusted candidates must be ordered by marker expression significance.
when_to_use_negative:
- When the marker list has no expression score / ranking information (e.g. only an
  unranked overlap gene list) — the article warns that ignoring gene ranking leads
  to unreliable results
- When a full GSEA-based ranked-test annotation is required instead of the Fisher-test-plus-ranking
  approach (EasyCellType offers both GSEA and the modified Fisher's exact test)
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EasyCellType
  role: R package implementing the modified Fisher's exact test combined with expression
    significance; its easyct(input.d, db='cellmarker', test='fisher') call produces
    the score-ranked per-cluster annotations
  repo: https://github.com/rx-li/EasyCellType
- name: Fisher's exact test
  role: Modified overrepresentation test whose per-cell-type p-values are BH-adjusted,
    after which passing candidates are ordered by marker expression score
- name: Seurat
  role: Generates the upstream per-cluster marker lists (FindAllMarkers) whose expression
    scores drive the ranking
- name: CellMarker
  role: Reference cell-type gene-set database queried for the Human blood/peripheral-blood
    tissue subset
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:c9cd170c76f1fd1f39d19f8ad4c056eeb1cd6b3ce85dd7f2bcf51ccac1dcfc23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - CellMarker
  - EasyCellType
  - Fisher's exact test
  - Seurat
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# marker-expression-score-ranking

## Summary

Rank cell-type candidates for each cluster by the markers' expression score (e.g. log fold change) rather than raw overlap size, so that the top-ranked marker–label associations become the hard (top-1) and soft (top-5) annotation recommendations in EasyCellType's modified Fisher's exact test workflow.

## When to use

Use when converting per-cluster differential marker lists (e.g. Seurat FindAllMarkers output with Entrez IDs and scores) into ranked cell-type annotations, where significance-adjusted candidates must be ordered by marker expression significance.

## When NOT to use

- When the marker list has no expression score / ranking information (e.g. only an unranked overlap gene list) — the article warns that ignoring gene ranking leads to unreliable results
- When a full GSEA-based ranked-test annotation is required instead of the Fisher-test-plus-ranking approach (EasyCellType offers both GSEA and the modified Fisher's exact test)

## Inputs

- Per-cluster marker table (e.g. gene_pbmc from the EasyCellType package) containing Entrez IDs, cluster labels, and expression scores sorted in descending rank order within each cluster
- CellMarker cell-type gene sets for the Human species (blood/peripheral-blood tissue subset)
- BH-adjusted Fisher's exact test results across candidate cell types per cluster

## Outputs

- Per-cluster ranked cell-type annotation table (e.g. fisher_annotations.csv)
- Hard (top-1) cell-type annotation per cluster
- Soft (top-5) ranked cell-type recommendations per cluster

## How to apply

After running the modified Fisher's exact test (test_fisher via easyct with test='fisher') on each cluster's marker list against CellMarker gene sets (e.g. Human, blood/peripheral-blood tissue subset, p_cut=0.3), adjust p-values across candidate cell types within each cluster using Benjamini-Hochberg. Then order the passing cell-type candidates by the markers' expression score significance (e.g. descending log fold change) rather than by overlap size alone — the article stresses that simply overlapping gene lists without considering gene ranking can lead to unreliable results. Select the top-1 candidate as the hard annotation and the top-5 as soft recommendations per cluster, and write the per-cluster annotation table (e.g. fisher_annotations.csv).

## Related tools

- **EasyCellType** (R package implementing the modified Fisher's exact test combined with expression significance; its easyct(input.d, db='cellmarker', test='fisher') call produces the score-ranked per-cluster annotations) — https://github.com/rx-li/EasyCellType
- **Fisher's exact test** (Modified overrepresentation test whose per-cell-type p-values are BH-adjusted, after which passing candidates are ordered by marker expression score)
- **Seurat** (Generates the upstream per-cluster marker lists (FindAllMarkers) whose expression scores drive the ranking)
- **CellMarker** (Reference cell-type gene-set database queried for the Human blood/peripheral-blood tissue subset)

## Examples

```
data("gene_pbmc"); annot <- easyct(input.d = gene_pbmc, db = "cellmarker", species = "Human", tissue = "blood", test = "fisher")
```

## Evaluation signals

- Outputs are reproducible with easyct(input.d, db='cellmarker', test='fisher') on the package's gene_pbmc dataset
- Top-ranked annotations are consistent with known PBMC populations as displayed by the package's dot-plot summary
- Per-cluster annotation table (fisher_annotations.csv) contains one ranked candidate list per cluster with hard top-1 and soft top-5 entries
- Candidate ordering reflects marker expression score significance, not raw overlap size
- Fisher p-values within each cluster are Benjamini-Hochberg adjusted before ranking

## Limitations

- Simply overlapping gene lists without considering gene ranking might lead to unreliable results — ranking by expression score is essential, not optional
-  tissue/database coverage depends on CellMarker's content; the article notes only '3 public available marker gene data bases' without fully enumerating tissue subsets
- No software version numbers are documented in the methods, complicating exact reproducibility

## Evidence

- [abstract] ranking rationale: "Additionally, simply overlapping the two lists without considering gene ranking might lead to unreliable results."
- [intro] two-test design: "Two quantification approaches are provided to annotate cell types: Gene set enrichment analysis (GSEA) and a fisher exact test combining with expression significance"
- [methods] easyct invocation: "annot.GSEA <- easyct(input.d, db="cellmarker", species="Human","
- [methods] package data input: "We have include the processed data in the package. It can be loaded with
data("gene_pbmc")"
- [readme] README skill description: "Two quantification approaches are provided to annotate cell types: Gene set 
enrichment analysis (GSEA) and a fisher exact test combining with expression 
significance."
