---
name: single-cell-feature-importance-attribution
description: Use when you have a pretrained Pollock module and annotated scRNA-seq
  data, and need to know WHICH genes drive a cell type/state classification, e.g.
  to feed top-ranked DWGs into GO enrichment.
when_to_use_negative:
- Input expression matrix is already normalized — explain mode requires raw counts
  stored in .X or Seurat counts slot
- You only need cell type labels, not the genes driving them — use `pollock predict`
  instead of explain mode
- The input is a precomputed feature/gene importance table rather than raw single-cell
  expression data
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: Pollock
  role: explain-mode CLI that computes per-gene feature attribution scores for a pretrained
    VAE classification module
  repo: https://github.com/ding-lab/pollock
- name: captum
  role: PyTorch interpretability library installed in the pollock conda environment,
    underlying feature attribution score generation
- name: scanpy
  role: provides the .h5ad anndata input format for from_scanpy explain runs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:b8b5562ea75123987fd0bcbd97f688f35a52950e7ba23505c7e937f58e9d5eaf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Pollock
  - captum
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-feature-importance-attribution

## Summary

Computes per-gene feature attribution scores for single-cell classification models using Pollock's explain mode (VAE with linear classification head, backed by captum), then ranks genes per predicted cell state to extract differentially weighted genes (DWGs) driving each state's classification.

## When to use

Use when you have a pretrained Pollock module and annotated scRNA-seq data, and need to know WHICH genes drive a cell type/state classification, e.g. to feed top-ranked DWGs into GO enrichment.

## When NOT to use

- Input expression matrix is already normalized — explain mode requires raw counts stored in .X or Seurat counts slot
- You only need cell type labels, not the genes driving them — use `pollock predict` instead of explain mode
- The input is a precomputed feature/gene importance table rather than raw single-cell expression data

## Inputs

- Pretrained Pollock module directory (e.g. scRNAseq_brca_panimmune from pretrained_models.tar.gz at zenodo.org/record/5895221)
- Scanpy .h5ad or Seurat .RDS object with raw (unnormalized) counts and predicted cell state labels in the column given by --predicted-key
- Background expression object (.h5ad/.rds) with raw counts, sampled to --background-sample-size cells

## Outputs

- Per-gene feature attribution/interpretability scores for each predicted cell state
- Rank-ordered list of top discriminative weighted genes (DWGs) per state (top 20 in the reference workflow)
- dwg_go_enrichment_per_state.tsv: rank-ordered significant GO Biological Process terms per state from ToppFun

## How to apply

Run `pollock explain from_scanpy` (or `from_seurat`) supplying the pretrained module via --module-filepath and a raw-count anndata/Seurat object via --explain-filepath; expression must be raw counts (unnormalized). Supply a background expression object via --background-filepath with --background-sample-size 100 (the default, sufficient in most use cases) and point --predicted-key at the obs column holding predicted state labels. Explain mode has excessive runtimes for large cell numbers (~100 cells with a background of ~100 cells takes ~15 minutes), so downsample cells to explain. Rank genes by their feature importance score per state and retain the top 20 discriminative genes (DWGs); in the task_007 workflow these were submitted to ToppFun GO Biological Process over-representation, yielding concordant terms (cell cycle for proliferating CD8 T cells, leukocyte activation regulation for Tregs, cell killing for NK cells).

## Related tools

- **Pollock** (explain-mode CLI that computes per-gene feature attribution scores for a pretrained VAE classification module) — https://github.com/ding-lab/pollock
- **captum** (PyTorch interpretability library installed in the pollock conda environment, underlying feature attribution score generation)
- **scanpy** (provides the .h5ad anndata input format for from_scanpy explain runs)

## Examples

```
pollock explain from_scanpy --module-filepath <module_dir> --explain-filepath counts_annotated.h5ad --background-filepath counts_background.h5ad --background-sample-size 100 --predicted-key cell_state
```

## Evaluation signals

- Explain run logs an explicit background sample size (default 100) and processes the expected number of cells, e.g. ~100 cells with ~100 background cells finishing in roughly 15 minutes
- Attribution scores are produced for every cell state in the module (e.g. 17 classes including 'B cell', 'CD4 T cell', 'Monocyte', 'NK', 'Malignant')
- Top-20 DWG lists yield GO terms concordant with known biology: cell cycle terms for proliferating CD8 T cells, regulation of leukocyte activation for Tregs, cell activation/response to stress for exhausted CD8 T cells, cell killing for NK cells
- Input object passes the raw-counts check (counts in .X or @assays$RNA@counts, not log-normalized)

## Limitations

- Explain mode can have excessive runtimes for very large numbers of cells; the README recommends downsampling the cells to be explained
- Expression data must be raw counts (unnormalized); normalized inputs are invalid for explain mode
- Runtime scales with cell count: ~100 explained cells with a background sample size of ~100 takes approximately 15 minutes

## Evidence

- [intro] Pollock is a deep learning method for single cell classification.: "Pollock is a deep learning method for single cell classification."
- [abstract] Pollock provides interpretability scores identifying genes driving classifications: "provides a set of pretrained human cancer reference models, and reports interpretability scores that identify the genes that drive cell type classifications"
- [methods] Explain mode description from article workflow: [evidence span withheld — anchor status: not_found]
- [readme] Raw counts required for explain mode inputs: "Filepath to seurat .rds object or scanpy .h5ad anndata object containing cells to be explained. Expression data must be raw counts (i.e. unnormalized)."
- [readme] Background sample size default and sufficiency: "Number of cells to sample as background samples. The default of 100 cells is sufficient in most use cases."
- [readme] Explain mode runtime guidance: "running ~100 cells with a background sample size of ~100 cells results in a runtime of approximately 15 minutes"
- [results] Top-20 DWG GO enrichment concordance in reference workflow: [evidence span withheld — anchor status: not_found]
