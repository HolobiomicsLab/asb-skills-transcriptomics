---
name: immune-cell-state-marker-recovery-validation
description: Use when a trained Pollock cell-type classification model must be validated
  biologically by checking whether its per-state gene attribution (DWG) rankings recover
  literature-known markers such as FOXP3, LAG3, or PDCD1.
when_to_use_negative:
- Input expression matrices are normalized (e.g., log1p/scaled) — explain mode requires
  raw counts in .X or @assays$RNA@counts
- The goal is only cell type labeling without gene-level drivers — use predict mode
  instead of explain mode
- Very large numbers of cells to explain without downsampling — the README warns of
  excessive runtimes in explain mode
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Pollock
  role: Provides explain mode and the pretrained pan-immune VAE classifier whose feature
    interpretability scores are ranked into per-state DWGs
  repo: https://github.com/ding-lab/pollock
- name: Captum
  role: Computes Integrated Gradients feature attributions against background cell
    samples
- name: PyTorch
  role: Backend framework for the Pollock VAE model used in explain mode
- name: scanpy
  role: Reads and supplies .h5ad anndata inputs (raw counts) for explain mode
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:5fd84f37f0f6e7402adb29647620976de17f5647412944a1dec766ded9554679
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Captum
  - Pollock
  - PyTorch
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# immune-cell-state-marker-recovery-validation

## Summary

Validates that Pollock's deep-learning feature interpretability scores (Captum Integrated Gradients DWGs) recover known immune cell state markers — e.g., FOXP3 in Tregs and LAG3/PDCD1 in exhausted T cells — by running the pretrained pan-immune module (scRNAseq_brca_panimmune) in explain mode on held-out annotated cells.

## When to use

Use when a trained Pollock cell-type classification model must be validated biologically by checking whether its per-state gene attribution (DWG) rankings recover literature-known markers such as FOXP3, LAG3, or PDCD1.

## When NOT to use

- Input expression matrices are normalized (e.g., log1p/scaled) — explain mode requires raw counts in .X or @assays$RNA@counts
- The goal is only cell type labeling without gene-level drivers — use predict mode instead of explain mode
- Very large numbers of cells to explain without downsampling — the README warns of excessive runtimes in explain mode

## Inputs

- Pretrained Pollock module directory (e.g., scRNAseq_brca_panimmune from pretrained_models.tar.gz, Zenodo record 5895221)
- Held-out annotated cells in a Seurat .rds or scanpy .h5ad object (raw, unnormalized counts) with predicted cell type labels, up to 500 cells per immune cell state
- Background cells in a Seurat .rds or scanpy .h5ad object (raw counts) to be sampled to --background-sample-size (default 100)

## Outputs

- Ranked tab-separated .txt table of top driver genes (DWGs) per predicted cell state, written via --output-prefix with --txt-output
- Per-cell Integrated Gradients feature attribution scores over input genes

## How to apply

Download the pretrained pan-immune module (scRNAseq_brca_panimmune) and annotated benchmarking data from Zenodo record 5895221 (pretrained_models.tar.gz and benchmarking_datasets.tar.gz). Run Pollock in explain mode (from_seurat or from_scanpy) with --module-filepath pointing at the module, --explain-filepath pointing at held-out annotated cells (up to 500 cells per immune cell state), and --background-filepath pointing at cells sampled as background. Captum Integrated Gradients attributions are computed with respect to input genes against a --background-sample-size of 100 (the default, 'sufficient in most use cases'). Aggregate per-cell scores by predicted cell state, rank genes to get the top DWGs per state, and write the ranked table with --output-prefix and --txt-output (tab-separated text). Validate by confirming recovery of canonical markers as in Fig 4B: FOXP3 among top-ranked Treg DWGs, LAG3 among exhausted CD8 T cell DWGs, and PDCD1 among exhausted CD4 T cell DWGs.

## Related tools

- **Pollock** (Provides explain mode and the pretrained pan-immune VAE classifier whose feature interpretability scores are ranked into per-state DWGs) — https://github.com/ding-lab/pollock
- **Captum** (Computes Integrated Gradients feature attributions against background cell samples)
- **PyTorch** (Backend framework for the Pollock VAE model used in explain mode)
- **scanpy** (Reads and supplies .h5ad anndata inputs (raw counts) for explain mode)

## Examples

```
pollock explain from_scanpy --module-filepath models/scRNAseq_brca_panimmune --scanpy-h5ad-filepath heldout_immune_cells.h5ad --explain-filepath heldout_immune_cells.h5ad --background-filepath background_cells.h5ad --output-prefix dwg_scores --txt-output
```

## Evaluation signals

- FOXP3 appears among the top-ranked Treg driver genes, and LAG3 and PDCD1 among the top-ranked exhausted CD8/CD4 T cell DWGs, as reported in Fig 4B
- The ranked DWG table is written as tab-separated text under the --output-prefix when --txt-output is set
- Background sampling resolves to --background-sample-size (default 100) cells; ~100 explained cells with ~100 background cells complete in roughly 15 minutes
- Held-out input is limited to at most 500 cells per immune cell state
- All model-overlapping genes are scored; genes missing from the dataset after filtering are excluded from the ranking

## Limitations

- Explain mode has excessive runtimes for very large numbers of cells; the README recommends downsampling before running
- Explain mode inputs must be raw, unnormalized counts in the Seurat or scanpy objects
- The default background sample size of 100 cells is described as sufficient in most use cases, but background choice affects attribution estimates

## Evidence

- [abstract] reports interpretability scores that identify the genes that drive cell type classifications: "provides a set of pretrained human cancer reference models, and reports interpretability scores that identify the genes that drive cell type classifications"
- [methods] explain mode background sampling: [evidence span withheld — anchor status: not_found]
- [readme] background sample size default: "Number of cells to sample as background samples. The default of 100 cells is sufficient in most use cases."
- [readme] explain input must be raw counts: "Filepath to seurat .rds object or scanpy .h5ad anndata object containing cells to be explained. Expression data must be raw counts (i.e. unnormalized)."
- [methods] pretrained models location: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
