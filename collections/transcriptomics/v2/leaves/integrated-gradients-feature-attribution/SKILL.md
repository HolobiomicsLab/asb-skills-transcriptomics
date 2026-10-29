---
name: integrated-gradients-feature-attribution
description: Use when a trained Pollock cell-type/state model needs interpretable
  gene-level attributions — e.g. to identify driving genes (DWGs) for predicted immune
  cell states on held-out annotated cells.
when_to_use_negative:
- Input expression matrices are normalized rather than raw counts — explain mode requires
  unnormalized counts in .X or @assays$RNA@counts
- The goal is cell type prediction only (use Pollock predict mode; explanation is
  not needed and adds substantial runtime)
- Very large numbers of cells without downsampling — explain mode has excessive runtimes
  and cells should be downsampled (e.g. ~100 cells ≈ 15 minutes)
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Pollock
  role: Provides the explain mode CLI/API that computes feature interpretability scores
    from its VAE with linear classification head
  repo: https://github.com/ding-lab/pollock
- name: Captum
  role: Backend used to compute Integrated Gradients feature attributions with respect
    to input genes
- name: PyTorch
  role: Deep learning framework underlying the Pollock VAE model through which attributions
    are computed
- name: scanpy
  role: Loads .h5ad AnnData inputs (raw counts in .X) for from_scanpy explain runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:4765bbe82f359b8f6d80c6cbdcb6a31d714ce8395e75ef464a7004b0ea234e76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Captum
  - Pollock
  - PyTorch
  - scanpy
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# integrated-gradients-feature-attribution

## Summary

Computes Captum Integrated Gradients feature attributions through Pollock's VAE-classifier to rank genes that drive cell type/state predictions, producing per-state DWG (differentially weight gene) rankings that recover known markers such as FOXP3 in Tregs and LAG3/PDCD1 in exhausted T cells.

## When to use

Use when a trained Pollock cell-type/state model needs interpretable gene-level attributions — e.g. to identify driving genes (DWGs) for predicted immune cell states on held-out annotated cells.

## When NOT to use

- Input expression matrices are normalized rather than raw counts — explain mode requires unnormalized counts in .X or @assays$RNA@counts
- The goal is cell type prediction only (use Pollock predict mode; explanation is not needed and adds substantial runtime)
- Very large numbers of cells without downsampling — explain mode has excessive runtimes and cells should be downsampled (e.g. ~100 cells ≈ 15 minutes)

## Inputs

- Pretrained Pollock module directory (e.g. scRNAseq_brca_panimmune from pretrained_models.tar.gz, Zenodo record 5895221)
- Held-out annotated cells to explain (Seurat .rds or scanpy .h5ad; .X must be raw unnormalized counts; up to 500 cells per immune cell state)
- Background cells for attribution sampling (Seurat .rds or scanpy .h5ad, raw counts; sampled to --background-sample-size, default 100)

## Outputs

- Ranked table of top DWGs (driving genes) per predicted cell state (tab-separated .txt when --txt-output is set)
- Per-cell Integrated Gradients feature attribution scores aggregated by predicted cell state

## How to apply

Download the pretrained pan-immune module (scRNAseq_brca_panimmune) and the annotated benchmarking dataset from Zenodo record 5895221 (pretrained_models.tar.gz and benchmarking_datasets.tar.gz). Run Pollock in explain mode (from_seurat or from_scanpy) with --module-filepath pointing at the module, --explain-filepath pointing at held-out annotated cells (up to 500 per immune cell state) whose expression matrix is raw (unnormalized) counts, and --background-filepath pointing at cells to sample as background. Captum Integrated Gradients attributions are computed with respect to input genes against a background sample size of 100 cells (the default, sufficient in most use cases). Aggregate per-cell scores by predicted cell state, rank genes to obtain the top DWGs per state, and write the ranked table with --output-prefix and --txt-output (tab-separated text). Validate by confirming recovery of canonical markers as in Fig 4B: FOXP3 among top-ranked Treg DWGs and LAG3/PDCD1 among top-ranked exhausted CD8/CD4 T cell DWGs. Note runtimes scale with cell count — ~100 cells with a background of ~100 takes ~15 minutes, so downsample.

## Related tools

- **Pollock** (Provides the explain mode CLI/API that computes feature interpretability scores from its VAE with linear classification head) — https://github.com/ding-lab/pollock
- **Captum** (Backend used to compute Integrated Gradients feature attributions with respect to input genes)
- **PyTorch** (Deep learning framework underlying the Pollock VAE model through which attributions are computed)
- **scanpy** (Loads .h5ad AnnData inputs (raw counts in .X) for from_scanpy explain runs)

## Examples

```
pollock explain from_scanpy --module-filepath modules/scRNAseq_brca_panimmune --explain-filepath heldout_cells.h5ad --background-filepath background_cells.h5ad --output-prefix dwg_results --txt-output
```

## Evaluation signals

- FOXP3 appears among top-ranked DWGs for the Treg cell state (Fig 4B recovery)
- LAG3 appears among top-ranked DWGs for exhausted CD8 T cells and PDCD1 for exhausted CD4 T cells (Fig 4B recovery)
- 12679 genes overlap with the model after filtering (genes missing from the dataset are expected and reported, e.g. 4859 missing)
- Ranked DWG table written as tab-separated text when --txt-output is used, with one ranked gene list per predicted cell state
- Runtime stays tractable: ~100 explained cells with background sample size of ~100 completes in ~15 minutes; larger sets require downsampling

## Limitations

- Explain mode can have excessive runtimes for very large numbers of cells; downsample cells to explain
- Expression data in both --explain-filepath and --background-filepath must be raw (unnormalized) counts
- Genes missing from the input dataset (e.g. 4859 of 17538 model genes in the pbmc3k example) cannot receive attribution scores
- Background sample size default of 100 is stated as sufficient in most use cases; unusual cases may need tuning

## Evidence

- [abstract] explain mode rationale: "provides a set of pretrained human cancer reference models, and reports interpretability scores that identify the genes that drive cell type classifications"
- [readme] explain mode usage: "Number of cells to sample as background samples. The default of 100 cells is sufficient in most use cases."
- [readme] raw counts requirement: "A saved .h5ad file to use as input. scanpy expression matrix (.X attribute in the anndata object) must be raw expression counts (i.e. not normalized)"
- [readme] runtime caveat: "Note: explain mode can have excessive runtimes for very large numbers of cells, so we recommend downsampling"
- [readme] runtime reference: "For reference, running ~100 cells with a background sample size of ~100 cells results in a runtime of approximately 15 minutes."
- [full_text] marker recovery finding: [evidence span withheld — anchor status: not_found]
