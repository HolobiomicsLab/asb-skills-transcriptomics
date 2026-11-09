---
name: gene-ontology-pathway-enrichment-analysis
description: Use when you have a ranked or top-N gene list per class/state (e.g. top
  20 DWGs from Pollock feature attribution) and need to identify enriched GO pathways
  and verify biological plausibility.
when_to_use_negative:
- The goal is de novo differential expression testing between annotated groups rather
  than interpreting a model's discriminative genes via feature attribution
- Input is a full expression matrix with no ranked gene lists — attribution scores
  (explain mode) must be computed first
- The gene lists are not tied to discrete predicted cell states (explain mode requires
  a --predicted-key state label column)
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Pollock
  role: Deep learning VAE classifier run in explain mode to compute per-gene feature
    attribution scores that define the DWGs submitted to GO enrichment
  repo: https://github.com/ding-lab/pollock
- name: captum
  role: Attribution/interpretability backend underlying Pollock's explain-mode feature
    importance scores
- name: scanpy
  role: Loading and handling the annotated scRNA-seq .h5ad raw-count expression objects
    used as explain and background inputs
- name: ToppFun
  role: GO Biological Process over-representation analysis on the top-20 DWG gene
    lists
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:df0ba5820401264a2ede87e0cd5343f3edb12b5fffbd64258a66e7aa4be736ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Pollock
  - ToppFun
  - captum
  - scanpy
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gene-ontology-pathway-enrichment-analysis

## Summary

Runs GO Biological Process over-representation analysis on ranked discriminative gene lists (e.g. Pollock explain-mode DWGs) per cell state to identify significant pathways, then checks concordance with expected cell-state biology.

## When to use

Use when you have a ranked or top-N gene list per class/state (e.g. top 20 DWGs from Pollock feature attribution) and need to identify enriched GO pathways and verify biological plausibility.

## When NOT to use

- The goal is de novo differential expression testing between annotated groups rather than interpreting a model's discriminative genes via feature attribution
- Input is a full expression matrix with no ranked gene lists — attribution scores (explain mode) must be computed first
- The gene lists are not tied to discrete predicted cell states (explain mode requires a --predicted-key state label column)

## Inputs

- Per-gene feature attribution scores per predicted cell state from Pollock explain mode
- Ranked top-20 discriminative gene (DWG) lists per state (Treg, CD8-proliferating, CD8-exhausted, NK)
- Pretrained module scRNAseq_brca_panimmune and annotated raw-count scRNA-seq .h5ad expression object (Zenodo record 5895221)

## Outputs

- dwg_go_enrichment_per_state.tsv: rank-ordered significant GO Biological Process terms per cell state
- Concordance assessment against expected cell-state biology

## How to apply

Obtain per-gene feature attribution scores (e.g. via Pollock explain mode using captum-based explainability on the scRNAseq_brca_panimmune module from Zenodo record 5895221, with raw-count .h5ad input, --background-filepath and --background-sample-size 100). For each target state (Treg, CD8 T cell-proliferating, CD8 T cell-exhausted, NK), rank genes by feature importance and retain the top 20 discriminative genes (DWGs). Submit each top-20 gene list to ToppFun GO Biological Process over-representation analysis and collect significant terms ranked by enrichment significance. Assemble the rank-ordered significant GO pathways per state into dwg_go_enrichment_per_state.tsv and check concordance: cell cycle terms should appear for proliferating CD8 T cells, regulation of leukocyte activation for Tregs, cell activation and response to stress for exhausted CD8 T cells, and cell killing for NK cells.

## Related tools

- **Pollock** (Deep learning VAE classifier run in explain mode to compute per-gene feature attribution scores that define the DWGs submitted to GO enrichment) — https://github.com/ding-lab/pollock
- **captum** (Attribution/interpretability backend underlying Pollock's explain-mode feature importance scores)
- **scanpy** (Loading and handling the annotated scRNA-seq .h5ad raw-count expression objects used as explain and background inputs)
- **ToppFun** (GO Biological Process over-representation analysis on the top-20 DWG gene lists)

## Examples

```
pollock explain from_scanpy --module-filepath <module_dir> --explain-filepath <annotated.h5ad> --predicted-key <state_col> --background-filepath <background.h5ad> --background-sample-size 100
```

## Evaluation signals

- Significant GO BP terms are returned for all four target states (Treg, CD8-proliferating, CD8-exhausted, NK)
- Proliferating CD8 T cell DWGs enrich for cell cycle related pathways
- Treg DWGs enrich for regulation of leukocyte activation
- NK cell DWGs enrich for cell killing pathways; exhausted CD8 T cell DWGs enrich for cell activation and response to stress
- Output file dwg_go_enrichment_per_state.tsv exists with terms ranked by enrichment significance

## Limitations

- Explain mode can have excessive runtimes for very large numbers of cells; ~100 cells with a background sample size of ~100 takes approximately 15 minutes, so downsampling is recommended
- Feature attribution reflects the pretrained module's learned decision boundary — enrichment only reflects genes driving classification, not unbiased DE biology
- Concordance with expected biology (e.g. cell killing for NK) is a qualitative sanity check, not a quantitative validation

## Evidence

- [abstract] Pretrained models and datasets are available for download at https://zenodo.org/record/5895221.: "Pretrained models and datasets are available for download at https://zenodo.org/record/5895221."
- [methods] An example of explaining a model for a Seurat .RDS object that has cell type labels in a metadata column named 'cell_type': "An example of explaining a model for a Seurat .RDS object that has cell type labels in a metadata column named 'cell_type'"
- [readme] Number of cells to sample as background samples. The default of 100 cells is sufficient in most use cases.: "Number of cells to sample as background samples. The default of 100 cells is sufficient in most use cases."
- [abstract] provides a set of pretrained human cancer reference models, and reports interpretability scores that identify the genes that drive cell type classifications: "provides a set of pretrained human cancer reference models, and reports interpretability scores that identify the genes that drive cell type classifications"
