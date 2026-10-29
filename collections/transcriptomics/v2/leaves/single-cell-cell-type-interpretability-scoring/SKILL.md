---
name: single-cell-cell-type-interpretability-scoring
description: Use when you have a trained Pollock VAE classification module (e.g. scRNAseq_brca_panimmune)
  plus raw-count .h5ad/.RDS cells and need the interpretability scores identifying
  which genes drive each predicted cell state.
when_to_use_negative:
- Input expression matrices are normalized rather than raw counts — explain mode requires
  unnormalized counts in .X or @assays$RNA@counts
- You need cell type prediction itself rather than explanation — use Pollock predict
  mode instead
- Very large numbers of cells to explain without downsampling — the README warns of
  excessive runtimes for large cell counts
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: Pollock
  role: Provides the explain mode CLI and VAE-plus-linear-classification-head model
    whose per-gene attributions are computed
  repo: https://github.com/ding-lab/pollock
- name: Captum
  role: Computes the Integrated Gradients feature attributions with respect to input
    genes against background samples
- name: PyTorch
  role: Underlying deep learning framework in the Pollock conda environment in which
    the VAE model and attributions run
- name: scanpy
  role: Reads and supplies the .h5ad AnnData objects used as explain and background
    inputs (from_scanpy source type)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:2439377d92dc847ff1c8b069d2d9098bfa5b6735a05c34ff218edc3237f46d65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Captum
  - Pollock
  - PyTorch
  - scanpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-cell-type-interpretability-scoring

## Summary

Ranks the genes that drive Pollock's cell state predictions (driver working genes, DWGs) by applying Captum Integrated Gradients feature attributions in Pollock's explain mode against a sampled background, yielding per-cell-state ranked gene tables used to confirm recovery of known markers.

## When to use

Use when you have a trained Pollock VAE classification module (e.g. scRNAseq_brca_panimmune) plus raw-count .h5ad/.RDS cells and need the interpretability scores identifying which genes drive each predicted cell state.

## When NOT to use

- Input expression matrices are normalized rather than raw counts — explain mode requires unnormalized counts in .X or @assays$RNA@counts
- You need cell type prediction itself rather than explanation — use Pollock predict mode instead
- Very large numbers of cells to explain without downsampling — the README warns of excessive runtimes for large cell counts

## Inputs

- Pretrained Pollock module directory (e.g. scRNAseq_brca_panimmune from pretrained_models.tar.gz, Zenodo record 5895221)
- Seurat .rds or scanpy .h5ad object with cells to explain, holding raw (unnormalized) counts in @assays$RNA@counts or anndata .X
- Seurat .rds or scanpy .h5ad object of cells to sample as background (raw counts)
- Predicted cell state labels for the cells to explain (--predicted-key)

## Outputs

- Tab-separated .txt table of per-cell-state ranked driver working genes (with --output-prefix and --txt-output)
- Per-cell Integrated Gradients gene attribution scores aggregated by predicted cell state

## How to apply

Run Pollock in explain mode (source_type from_seurat or from_scanpy) pointing --module-filepath at the pretrained module and --explain-filepath at held-out annotated cells whose expression matrix is raw, unnormalized counts. Supply --background-filepath with a cell object to be sampled as background; keep the default --background-sample-size of 100 cells, which is sufficient in most use cases (downsampling is recommended because runtimes scale with cell count). Pollock computes Captum Integrated Gradients attributions with respect to input genes; aggregate per-cell scores by predicted cell state, rank genes to obtain the top DWGs per state, and write them with --output-prefix and --txt-output as a tab-separated file. Validate by confirming recovery of canonical markers reported in Fig 4B, e.g. FOXP3 among top-ranked Treg DWGs and LAG3/PDCD1 among top-ranked exhausted CD8/CD4 T cell DWGs.

## Related tools

- **Pollock** (Provides the explain mode CLI and VAE-plus-linear-classification-head model whose per-gene attributions are computed) — https://github.com/ding-lab/pollock
- **Captum** (Computes the Integrated Gradients feature attributions with respect to input genes against background samples)
- **PyTorch** (Underlying deep learning framework in the Pollock conda environment in which the VAE model and attributions run)
- **scanpy** (Reads and supplies the .h5ad AnnData objects used as explain and background inputs (from_scanpy source type))

## Examples

```
pollock explain from_scanpy --module-filepath scRNAseq_brca_panimmune --scanpy-h5ad-filepath heldout_cells.h5ad --background-filepath background_cells.h5ad --background-sample-size 100 --output-prefix dwg_scores --txt-output
```

## Evaluation signals

- Canonical marker recovery as in Fig 4B: FOXP3 appears among top-ranked Treg DWGs
- LAG3 and PDCD1 appear among top-ranked DWGs for exhausted CD8 and CD4 T cells, respectively
- Per-cell-state ranked gene tables are produced for all predicted immune cell states (up to 500 held-out cells per state)
- Default --background-sample-size of 100 cells used; ~100 explain cells with ~100 background cells completes in approximately 15 minutes, so far longer runtimes suggest a misconfiguration
- Output file exists with the expected tab-separated format under the --output-prefix (only when --txt-output is set)

## Limitations

- Explain mode has excessive runtimes for very large numbers of cells; downsampling is recommended
- The default background sample size of 100 cells is stated as sufficient in most use cases, implying unusual datasets may need adjustment
- Explained cells must contain raw, unnormalized counts in the expected slots (@assays$RNA@counts or anndata .X)

## Evidence

- [abstract] abstract: "provides a set of pretrained human cancer reference models, and reports interpretability scores that identify the genes that drive cell type classifications"
- [methods] methods: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [readme] readme: "Filepath to seurat .rds object or scanpy .h5ad anndata object containing cells to be explained. Expression data must be raw counts (i.e. unnormalized)."
