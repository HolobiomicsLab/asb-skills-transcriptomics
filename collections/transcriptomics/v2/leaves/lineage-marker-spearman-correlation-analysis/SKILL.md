---
name: lineage-marker-spearman-correlation-analysis
description: Use when continuous cell-fate probabilities have been transferred to
  query cells (e.g., mouse Tusi reference to human Velten via Cell BLAST) and you
  need to quantify per-fate prediction quality against lineage marker expression.
when_to_use_negative:
- The task is discrete cell-type annotation rather than continuous cell-differentiation/fate
  prediction (use Cell Ontology term assignment via BLAST2CO-style confidence scoring
  instead)
- The reference dataset has fewer than 1000 cells, where the over-parameterized DIRECTi
  model may suffer from under-training and correlations would be unreliable
- Mouse and human genes cannot be reliably mapped via one-to-one orthologs, breaking
  the shared gene space required for marker-based scoring
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell BLAST
  role: Cell-querying framework used to transfer fate annotations from reference (Tusi)
    cells to query (Velten) cells via cell-to-cell similarity
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi (Cell_BLAST.directi.fit_DIRECTi)
  role: Deep generative model trained on the mouse reference with adversarial batch
    alignment-based online tuning before querying
- name: Cell_BLAST.blast.BLAST
  role: Index built from the fitted reference model; queries Velten cells and computes
    similarity between posterior distributions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:67b8fe6d5dda1af48f3ce2aec3e0e682e24f67d6857af04c999359597ce15b81
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - Cell_BLAST.blast.BLAST
  - DIRECTi (Cell_BLAST.directi.fit_DIRECTi)
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lineage-marker-spearman-correlation-analysis

## Summary

Validates transferred continuous cell-fate predictions by computing Spearman rank correlations between per-cell predicted fate probabilities (from Cell BLAST hits) and expression of known lineage marker genes, following the Velten inter-species transfer evaluation (Supplementary Fig. 17).

## When to use

Use when continuous cell-fate probabilities have been transferred to query cells (e.g., mouse Tusi reference to human Velten via Cell BLAST) and you need to quantify per-fate prediction quality against lineage marker expression.

## When NOT to use

- The task is discrete cell-type annotation rather than continuous cell-differentiation/fate prediction (use Cell Ontology term assignment via BLAST2CO-style confidence scoring instead)
- The reference dataset has fewer than 1000 cells, where the over-parameterized DIRECTi model may suffer from under-training and correlations would be unreliable
- Mouse and human genes cannot be reliably mapped via one-to-one orthologs, breaking the shared gene space required for marker-based scoring

## Inputs

- Query AnnData (h5ad) object with predicted per-cell fate probabilities (e.g., Velten human HSPC dataset)
- Ortholog-mapped gene expression matrix shared between mouse Tusi reference and human Velten query
- Known lineage marker gene lists from Supplementary Fig. 17
- Cell_BLAST.blast.BLAST hit list with reference cell annotations used for weighted fate-probability transfer

## Outputs

- Per-fate Spearman rank correlation table written to fate_probability_spearman.csv
- Predicted fate probability matrix per Velten query cell

## How to apply

After transferring fate annotations from reference hit cells to each query cell — yielding weighted predicted fate probabilities per Velten cell from the Cell_BLAST.blast.BLAST hit list — gather the known lineage marker gene lists (Supplementary Fig. 17). For each fate, compute the Spearman rank correlation between predicted fate probability across query cells and the expression of its lineage markers. Critically, first align mouse and human genes via one-to-one orthologs so both AnnData (h5ad) matrices share a common gene space, and train the DIRECTi model with online tuning (adversarial batch alignment) since this mode drove the significantly higher correlations reported. Write the per-fate correlation table to fate_probability_spearman.csv; correct application should show positive correlations for most lineages, with online-tuned Cell BLAST outperforming non-tuned baselines.

## Related tools

- **Cell BLAST** (Cell-querying framework used to transfer fate annotations from reference (Tusi) cells to query (Velten) cells via cell-to-cell similarity) — github.com/gao-lab/Cell_BLAST
- **DIRECTi (Cell_BLAST.directi.fit_DIRECTi)** (Deep generative model trained on the mouse reference with adversarial batch alignment-based online tuning before querying)
- **Cell_BLAST.blast.BLAST** (Index built from the fitted reference model; queries Velten cells and computes similarity between posterior distributions)

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST
```

## Evaluation signals

- fate_probability_spearman.csv contains one Spearman correlation per fate, and correlations are positive for most lineages
- Online-tuned (adversarial batch alignment) DIRECTi model yields significantly higher per-fate correlations than untuned baselines, as reported for the Velten transfer
- Gene matrices for Tusi and Velten share an identical gene space after one-to-one ortholog mapping before marker scoring
- Marker expression and fate probabilities are rank-correlated per query cell population (Spearman, not Pearson), matching Supplementary Fig. 17 evaluation design

## Limitations

- The DIRECTi model is over-parameterized and requires a reasonable number of samples to train properly; datasets with fewer than 1000 cells may suffer from under-training issues (3000 is generally sufficient)
- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition, which can distort transferred fate probabilities and their correlations
- Correlation with known lineage markers is an indirect validation, dependent on the completeness of the Supplementary Fig. 17 marker lists

## Evidence

- [abstract] abstract finding on continuous annotation: "we demonstrate the effectiveness of Cell BLAST in annotating discrete cell types and continuous cell differentiation potential, as well as identifying novel cell types"
- [discussion] discussion on robustness via batch modeling: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [readme] annotation transfer mechanism: "Annotations in reference cells, e.g. cell type, can then be transfered to query cells based on cell-to-cell similarities."
- [results] inter-species transfer result from source card: [evidence span withheld — anchor status: not_found]
- [readme] installation step from README: [evidence span withheld — anchor status: external_doc]
