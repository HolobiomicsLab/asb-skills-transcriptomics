---
name: accuracy-macrof1-metric-computation
description: Use when a benchmark produces per-cell predicted cell-type labels that
  must be scored against ground-truth annotations to rank annotation tools by accuracy
  and MacroF1.
when_to_use_negative:
- The goal is per-cluster differential expression or marker discovery rather than
  scoring fixed label assignments against ground truth
- Query data lack curated ground-truth cell-type annotations, so accuracy/MacroF1
  cannot be computed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SELINA
  role: Annotated method whose predictions (query_predictions.txt, --prob-cutoff 0.9)
    are scored with accuracy and MacroF1 against comparator tools
  repo: https://github.com/SELINA-team/SELINA-reference
- name: SingleR
  role: Reference-based comparator whose per-cell labels are included in the accuracy/MacroF1
    benchmark table
- name: SingleCellNet
  role: Reference-based comparator scored alongside SELINA in single- and multi-platform
    leave-one-dataset-out evaluations
- name: scmap
  role: Reference-based comparator included in the 9-tool Allen Brain Atlas benchmark
    ranking
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:d9c379ae22d0f8b7a70c6efeb1e452b05c74d255f194b9476a8df794973a9cf1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SELINA-team/SELINA.py
  repo_url: https://github.com/SELINA-team/SELINA-reference
  tools:
  - ACTINN
  - CHETAH
  - CellTypist
  - Garnett
  - Python
  - R
  - SCINA
  - SELINA
  - Seurat
  - SingleCellNet
  - SingleR
  - mtSC
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5357142857142857
n_claims: 28
---

# accuracy-macrof1-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute overall accuracy and Macro F1 to benchmark cell-type annotation tools (e.g., SELINA against 8–13 comparators) on held-out query datasets, then rank methods by averaged metrics across datasets or tissues.

## When to use

Use when a benchmark produces per-cell predicted cell-type labels that must be scored against ground-truth annotations to rank annotation tools by accuracy and MacroF1.

## When NOT to use

- The goal is per-cluster differential expression or marker discovery rather than scoring fixed label assignments against ground truth
- Query data lack curated ground-truth cell-type annotations, so accuracy/MacroF1 cannot be computed

## Inputs

- Per-cell prediction files (e.g., query_predictions.txt) from each annotation tool
- Ground-truth cell-type meta labels (e.g., GSE*_meta.txt files per dataset)
- Per-cell-type probability files (query_probability.txt) for threshold-based filtering
- List of cell types retained for scoring (e.g., 19 Allen Brain Atlas types with >1,000 cells)

## Outputs

- Accuracy and MacroF1 per tool, per dataset/tissue
- Averaged accuracy and MacroF1 across datasets or tissues (e.g., 93.32%/0.9336 for SELINA on Allen Brain Atlas)
- Ranked method comparison table matching reported figure ordering

## How to apply

For each tool and query split, compare the per-cell predictions (e.g., SELINA's query_predictions.txt produced with --prob-cutoff 0.9) against the curated meta labels, computing overall accuracy and Macro F1 where each cell type contributes equally regardless of size — critical given the imbalanced cell-type abundance noted in the SELINA reference. Score each tool per dataset (e.g., the two Allen Brain Atlas datasets restricted to 19 cell types with >1,000 cells) and per tissue across the 9 single-platform and 14 multi-platform leave-one-dataset-out splits. Average the metrics across datasets/tissues (e.g., SELINA averaging 93.32% accuracy and 0.9336 MacroF1 on Allen Brain Atlas; 61.51%/0.5126 single-platform and 64.42%/0.5082 multi-platform), then rank methods and compare against published figure values within tolerance.

## Related tools

- **SELINA** (Annotated method whose predictions (query_predictions.txt, --prob-cutoff 0.9) are scored with accuracy and MacroF1 against comparator tools) — https://github.com/SELINA-team/SELINA-reference
- **SingleR** (Reference-based comparator whose per-cell labels are included in the accuracy/MacroF1 benchmark table)
- **SingleCellNet** (Reference-based comparator scored alongside SELINA in single- and multi-platform leave-one-dataset-out evaluations)
- **scmap** (Reference-based comparator included in the 9-tool Allen Brain Atlas benchmark ranking)

## Examples

```
selina predict --query-expr ./res/query_expr.txt --model ./res/pre-trained_params.pt --prob-cutoff 0.9  # then compute accuracy/MacroF1 of query_predictions.txt vs GSE*_meta.txt labels
```

## Evaluation signals

- Reproduced SELINA Allen Brain Atlas table matches Figure 5B ordering: SELINA second of 9 tools, average accuracy within tolerance of 93.32% and MacroF1 within tolerance of 0.9336
- Leave-one-dataset-out summary reproduces reported SELINA values: 61.51% accuracy/0.5126 MacroF1 (single-platform) and 64.42%/0.5082 (multi-platform)
- MacroF1 computed over the 19 Allen Brain Atlas cell types restricted to >1,000 cells so each type contributes equally despite class imbalance
- Per-cell prediction and probability files exist for every leave-one-dataset-out split before metric aggregation

## Limitations

- Cell-type abundance across tissues and datasets in the reference remains imbalanced, which can affect benchmark comparability
- Metrics depend on the prob-cutoff (0.9) used for SELINA predictions; cells below cutoff change scored label counts
- Only cell types above the 1,000-cell threshold are scored on Allen Brain Atlas, excluding rare types from the evaluation

## Evidence

- [results] SELINA ranks second out of the 9 tested tools with an average accuracy of 93.32% and an average MacroF1 of 0.9336.: "SELINA ranks second out of the 9 tested tools with an average accuracy of 93.32% and an average MacroF1 of 0.9336."
- [intro] the minority cell types are always ignored in the modeling process: "the cell numbers of different cell types are often imbalanced; therefore, the minority cell types are always ignored in the modeling process"
- [methods] Compute accuracy and MacroF1 for each of the 9 tools on each dataset across the 19 cell types.: [evidence span withheld — anchor status: not_found]
- [readme] query_predictions.txt: predicted cell type for each cell in the query data: "- `query_predictions.txt`: predicted cell type for each cell in the query data"
