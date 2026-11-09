---
name: f1-score-computation-across-dropout-levels
description: Use when benchmarking a pretrained Pollock module's tolerance to artificial
  sequencing dropout — e.g., you have .h5ad/.rds validation data with ground-truth
  'cell_type' annotations and need an F1-versus-dropout robustness curve.
when_to_use_negative:
- Input expression matrices are normalized (e.g. normalize_total to 1e4 + log1p) rather
  than raw counts — Pollock predict mode requires raw expression counts in the anndata
  .X or Seurat @assays$RNA@counts slot
- The analysis goal is robustness to batch effects, platform shift, or snATAC-seq
  gene-activity input rather than artificially added dropout
- You need to train a new module from annotated data rather than evaluate an existing
  pretrained module's prediction robustness
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Pollock
  role: Pretrained VAE-with-linear-classification-head modules run in predict mode
    ('pollock predict') on each masked dataset to produce cell type predictions
  repo: https://github.com/ding-lab/pollock
- name: scanpy
  role: Loads validation count matrices (.h5ad) and provides the anndata objects into
    which artificial dropout masking is applied before prediction
- name: PyTorch
  role: Backend dependency of the Pollock conda environment used to execute model
    predictions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:d2860ead515d5122934a7078dff3687d16df90fddd44c2fe1822c1da3ac14f6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Pollock
  - PyTorch
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# f1-score-computation-across-dropout-levels

## Summary

A robustness-evaluation procedure for Pollock cell type classification: scRNA-seq validation matrices are masked with six uniform-random dropout levels (0–90%) and macro-averaged F1 against annotated cell type labels is collated to characterize where prediction performance degrades.

## When to use

Use when benchmarking a pretrained Pollock module's tolerance to artificial sequencing dropout — e.g., you have .h5ad/.rds validation data with ground-truth 'cell_type' annotations and need an F1-versus-dropout robustness curve.

## When NOT to use

- Input expression matrices are normalized (e.g. normalize_total to 1e4 + log1p) rather than raw counts — Pollock predict mode requires raw expression counts in the anndata .X or Seurat @assays$RNA@counts slot
- The analysis goal is robustness to batch effects, platform shift, or snATAC-seq gene-activity input rather than artificially added dropout
- You need to train a new module from annotated data rather than evaluate an existing pretrained module's prediction robustness

## Inputs

- Raw (unnormalized) scRNA-seq count matrices from validation datasets in benchmarking_datasets.tar.gz (Zenodo 5895221), e.g. .h5ad with counts in anndata .X
- Pretrained Pollock modules from pretrained_models.tar.gz (Zenodo 5895221), e.g. scRNAseq_generalized
- Annotated ground-truth cell type labels (e.g. 'cell_type' in .obs or Seurat metadata)

## Outputs

- Cell type predictions from 'pollock predict' for each masked dataset
- dropout_f1_scores.csv collating macro-averaged F1 versus dropout fraction (0%, 20%, 40%, 60%, 80%, 90%)

## How to apply

Download pretrained_models.tar.gz and benchmarking_datasets.tar.gz from Zenodo record 5895221, and load each raw (unnormalized) count matrix for the validation datasets with scanpy. Generate six masked copies per dataset in which 0%, 20%, 40%, 60%, 80%, and 90% of matrix positions are set to zero (uniform random dropout simulation), keeping raw counts so Pollock's predict mode receives unnormalized input. Run 'pollock predict from_scanpy' (or from_seurat/from_10x) with the pretrained module on each masked copy, then compute macro-averaged F1 of predicted versus annotated cell type labels at each level. Collate F1 versus dropout fraction into dropout_f1_scores.csv; the expected pattern is F1 stable up to 60% additional dropout and a steadier decline beyond 60%.

## Related tools

- **Pollock** (Pretrained VAE-with-linear-classification-head modules run in predict mode ('pollock predict') on each masked dataset to produce cell type predictions) — https://github.com/ding-lab/pollock
- **scanpy** (Loads validation count matrices (.h5ad) and provides the anndata objects into which artificial dropout masking is applied before prediction)
- **PyTorch** (Backend dependency of the Pollock conda environment used to execute model predictions)

## Examples

```
pollock predict from_scanpy --module-filepath <path_to_module_directory> --scanpy-h5ad-filepath <masked_h5ad> --output-prefix output --txt-output
```

## Evaluation signals

- F1 remains stable (does not markedly decline) up to 60% additional dropout, then declines more steadily beyond 60% — this is the paper's validation criterion
- dropout_f1_scores.csv contains exactly six dropout levels (0%, 20%, 40%, 60%, 80%, 90%) per validation dataset
- F1 is computed as macro-average across cell type classes against the annotated labels, not micro- or per-class averages
- Masked matrices retain raw-count values with only zeroed positions (uniform random masking), and prediction is run on raw counts per the README requirement

## Limitations

- Dropout is simulated computationally by zeroing matrix positions (uniform random), not empirically measured degradation from sequencing depth, so the curve reflects artificial masking behavior only
- Results are established for the pretrained scRNA-seq modules and validation datasets in the paper's benchmarking set; other modules or tissues may show different breakpoints
- Pollock requires raw unnormalized counts as input (anndata .X or Seurat @assays$RNA@counts), so masked copies must preserve raw count semantics

## Evidence

- [methods] uniform-random dropout masking levels: [evidence span withheld — anchor status: not_found]
- [methods] predict mode execution: [evidence span withheld — anchor status: not_found]
- [results] robustness threshold finding: [evidence span withheld — anchor status: not_found]
- [readme] raw-count input requirement: "A saved .h5ad file to use as input. scanpy expression matrix (.X attribute in the anndata object) must be raw expression counts (i.e. not normalized)"
