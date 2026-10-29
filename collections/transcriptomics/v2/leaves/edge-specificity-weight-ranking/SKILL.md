---
name: edge-specificity-weight-ranking
description: Use when you have NATMI ExtractEdges output carrying both mean-expression
  and specificity weight columns and need to assess whether ranking metric choice
  alters the top-ranked ligand–receptor pairs per cell-type pair.
when_to_use_negative:
- Input data are not normalized to counts per million, or cell barcodes lack a mapping
  to cell-type labels in a metafile — ExtractEdges requires both
- The goal is predicting downstream signalling consequences, spatial communication,
  or ligand–receptor binding kinetics — NATMI explicitly does not model these
- The ligand–receptor pairs of interest are heteromeric complexes — NATMI does not
  model them
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: NATMI
  role: ExtractEdges.py extracts all ligand–receptor edges with both mean-expression
    and specificity weight columns used for the dual ranking comparison
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:8e13c746434f7314f6e5980b8bc1ce1ab9fc7d5074de84fb955becc49c1512c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# edge-specificity-weight-ranking

## Summary

Ranks predicted ligand–receptor edges from NATMI ExtractEdges output by mean-expression weight versus specificity weight and compares the resulting top-20 lists to expose how weighting choice changes prioritized communication pairs, as demonstrated on the Skelly cardiac dataset (126,738 edges, 12 cell types).

## When to use

Use when you have NATMI ExtractEdges output carrying both mean-expression and specificity weight columns and need to assess whether ranking metric choice alters the top-ranked ligand–receptor pairs per cell-type pair.

## When NOT to use

- Input data are not normalized to counts per million, or cell barcodes lack a mapping to cell-type labels in a metafile — ExtractEdges requires both
- The goal is predicting downstream signalling consequences, spatial communication, or ligand–receptor binding kinetics — NATMI explicitly does not model these
- The ligand–receptor pairs of interest are heteromeric complexes — NATMI does not model them

## Inputs

- CPM-normalized single-cell gene expression matrix (e.g. expression.matrix.txt with official gene symbols)
- Cell-annotation metafile mapping each cell barcode to one of 12 cardiac cell types (annotation.txt)
- connectomeDB2020 ligand–receptor pair list (2,293 curated pairs, bundled with NATMI / Docker image)

## Outputs

- Extracted edge list of 126,738 predicted ligand–receptor edges with mean-expression and specificity weight columns
- Top-20 ligand–receptor pair ranking by mean-expression weight
- Top-20 ligand–receptor pair ranking by specificity weight
- Comparison summary of rank changes between the two weightings (cf. Fig. 1 of the manuscript)

## How to apply

Load a CPM-normalized (counts per million UMIs/reads, rescaled by multiplying by 1,000,000) expression matrix plus a cell-annotation metafile mapping barcodes to cell types. Run ExtractEdges.py with the connectomeDB2020 ligand–receptor list (2,293 literature-supported pairs) and detection threshold 0.2, so a gene counts as expressed only if detected in more than 20% of cells of a cell type. Each extracted edge carries both a mean-expression weight and a specificity weight column; sort the edge list separately by each metric and extract the top-20 pairs per ranking. Because specificity weights are computed from the input dataset only, they are local/dataset-specific, whereas mean-expression weights favor highly and broadly expressed (often housekeeping) ligands—comparing the two rankings reveals that expression-ranked edges span a broader set of communicating cell-type pairs but retain less informative housekeeping interactions. Validate by confirming the extracted edge count matches the reference edge table (126,738 edges for the Skelly cardiac dataset in Supplementary Data 2).

## Related tools

- **NATMI** (ExtractEdges.py extracts all ligand–receptor edges with both mean-expression and specificity weight columns used for the dual ranking comparison) — https://github.com/forrest-lab/NATMI/

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder
```

## Evaluation signals

- Reproduced edge count equals 126,738 predicted edges, matching Supplementary Data 2 of the Skelly cardiac analysis
- Every edge in the extracted list carries both a mean-expression weight and a specificity weight column
- Top-20 pairs under each weighting are consistent with the Supplementary Data 2 edge table
- The two top-20 lists differ substantially, with expression-ranked pairs spread across a broader range of communicating cell-type pairs
- Detection threshold of 0.2 was applied, i.e. genes counted as expressed only when detected in more than 20% of a cell type's cells

## Limitations

- Specificity weights are calculated from the input dataset and are therefore local/dataset-specific; global specificities (e.g. Human Cell Atlas based) are deferred to future NATMI versions
- Predictions rely on mRNA expression rather than protein concentration and are subject to dropouts; no binding kinetics are estimated
- NATMI does not model heteromeric complexes, heterodimerization, or co-receptor requirements, and highlighted edges may come from experiments without replicates

## Evidence

- [discussion] detection threshold 0.2: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [abstract] connectomeDB2020 pair count: "NATMI uses connectomeDB2020 (a database of 2293 manually curated ligand-receptor pairs with literature support)"
- [discussion] specificity is dataset-local: "the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific"
- [readme] ExtractEdges invocation: [evidence span withheld — anchor status: not_found]
- [methods] detectionThreshold parameter default: "--detectionThreshold DETECTIONTHRESHOLD
                        do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
