---
name: f1-precision-metric-computation
description: Use when benchmarking a cell type annotation method against baselines
  and you need per-class F1/precision distributions that are fair across methods with
  different reference sets.
when_to_use_negative:
- If the question is aggregate detection-quality rather than per-cell-type classification,
  average precision is preferred — the paper itself notes F1-score performance lags
  average precision and 'better calibration needed for binary yes-no decisions'
- When the goal is annotating a new dataset rather than benchmarking classifiers against
  a labeled test set with ontology-grounded ground truth
- Do not compare methods on all Cell Ontology terms — comparing a method on cell types
  absent from its own training set produces unfair metrics
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellO
  role: Hierarchical classifier whose per-cell probabilities and Cell Ontology term
    IDs form one arm of the benchmark; installed via 'pip install cello-classify'
  repo: https://github.com/deweylab/CellO
- name: scMatch
  role: Ready-to-run baseline classifier run with the FANTOM5 reference; labels projected
    onto Cell Ontology term IDs
- name: SingleR
  role: Ready-to-run baseline classifier run with Blueprint+ENCODE and HPCA references;
    labels projected onto Cell Ontology term IDs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:7c7eeffda7281554d120076b90c9ab7f499008412d68a57b2cd0a12bccb4ef22
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deweylab/CellO
  repo_url: https://github.com/deweylab/CellO
  tools:
  - CellO
  - SingleR
  - scMatch
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# f1-precision-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-cell-type F1-score and precision for hierarchical cell type classifiers (CellO, scMatch, SingleR) over a common single-cell test set, restricting evaluation to cell types present in each method's own prepackaged reference set. This reproduces the Figure 5B benchmark methodology from the CellO paper.

## When to use

Use when benchmarking a cell type annotation method against baselines and you need per-class F1/precision distributions that are fair across methods with different reference sets.

## When NOT to use

- If the question is aggregate detection-quality rather than per-cell-type classification, average precision is preferred — the paper itself notes F1-score performance lags average precision and 'better calibration needed for binary yes-no decisions'
- When the goal is annotating a new dataset rather than benchmarking classifiers against a labeled test set with ontology-grounded ground truth
- Do not compare methods on all Cell Ontology terms — comparing a method on cell types absent from its own training set produces unfair metrics

## Inputs

- Non-droplet single-cell RNA-seq test set from the Zenodo deposit 10.5281/zenodo.4289064
- CellO per-cell classification probability matrix (NxM TSV over Cell Ontology terms)
- scMatch labels from the FANTOM5 reference
- SingleR labels from Blueprint+ENCODE and HPCA references
- Ontology-grounded ground truth cell type labels for the test set

## Outputs

- TSV table of per-cell-type F1-score and precision for each of the three methods
- Figure 5-style summary plots of F1-score and precision distributions across cell types
- Qualitative comparison supporting the finding that CellO outperforms scMatch and SingleR

## How to apply

Classify the non-droplet single-cell test set with each method (CellO with the default isotonic regression algorithm 'IR', scMatch with the FANTOM5 reference, SingleR with Blueprint+ENCODE and HPCA references), capturing per-cell probabilities from the CellO `<output_prefix>.probability.tsv` output. Project the flat scMatch and SingleR labels onto Cell Ontology term IDs (using CellO's `-b` term-id output convention) so all methods can be scored against the same ontology-grounded ground truth. For each method, restrict evaluation to cell types present in that method's training set — this is the key fairness decision — then compute per-cell-type F1-score and precision over the test cells. Save the per-cell-type metrics as a TSV table and render F1 and precision distributions across cell types as Figure 5-style summary plots.

## Related tools

- **CellO** (Hierarchical classifier whose per-cell probabilities and Cell Ontology term IDs form one arm of the benchmark; installed via 'pip install cello-classify') — https://github.com/deweylab/CellO
- **scMatch** (Ready-to-run baseline classifier run with the FANTOM5 reference; labels projected onto Cell Ontology term IDs)
- **SingleR** (Ready-to-run baseline classifier run with Blueprint+ENCODE and HPCA references; labels projected onto Cell Ontology term IDs)

## Examples

```
cello_predict -a IR -d 10x -u COUNTS -s 3_PRIME example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- The per-cell-type metrics TSV is populated (non-empty F1 and precision values) for all three methods: CellO, scMatch, and SingleR
- F1 and precision distributions qualitatively match Figure 5B of the paper, with CellO competitive with or superior to scMatch and SingleR
- Each method is evaluated only on cell types present in that method's own training set — verify the restricted term list per method before scoring
- All method outputs are scored against the same Cell Ontology-grounded ground truth after projecting flat labels onto ontology term IDs

## Limitations

- Calibration gap: CellO's F1-score performance lags its average precision; the paper notes techniques were developed 'for closing the gap between the performance of CellO when evaluated with average precision versus when evaluated with F1-score'
- Optimal clustering resolution for scRNA-seq annotation is an unsolved problem, which affects cluster-based annotation accuracy
- Only cell types present in each method's prepackaged reference set can be scored, so the evaluated term sets differ per method

## Evidence

- [results] CellO outperformed the existing ready-to-run methods scMatch and SingleR on the non-droplet-based single-cell test set (Figure 5B): [evidence span withheld — anchor status: not_found]
