---
name: simulated-scrnaseq-data-generation
description: Use when benchmarking source-to-target transfer clustering (e.g., NMF
  dictionary transfer + SC3) requires synthetic count matrices with exact ground-truth
  labels, or when comparing complete / incomplete / no-overlap source-target cell-type
  overlap settings.
when_to_use_negative:
- The analysis targets real published scRNA-Seq data (Tasic GSE71585, Usoskin GSE59739,
  Hockley GSE102962) — use the real-data wrappers (main_wrapper_tasic.py, main_wrapper_hockley.py)
  instead of simulated data generation.
- Ground-truth cell-type labels are already known for a real dataset and no synthetic
  benchmark is needed, since simulation is only required to recover supervised ARI
  values against known labels.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: scRNA (cmd_generate_data.py)
  role: Python framework whose command line script scRNA-generate-data.sh / cmd_generate_data.py
    produces the artificial scRNA-Seq count data, gene ids, and ground-truth labels
    used in the simulation study
  repo: https://github.com/nicococo/scRNA/blob/master/scRNA/cmd_generate_data.py
- name: SC3
  role: Downstream clustering method applied to the generated simulated datasets to
    benchmark the transfer learning approach; not used during generation itself
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:4fec31c2a1399683c858eb1dae4c297f42f2d1402993a83ce535d4e6fe9e6860
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SC3
  - scRNA (cmd_generate_data.py)
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# simulated-scrnaseq-data-generation

## Summary

Generate artificial count-level scRNA-Seq datasets with known ground-truth cell labels using the nicococo/scRNA package's scRNA-generate-data.sh / cmd_generate_data.py, in the paper's benchmark configuration (10,000 genes, 1,800 cells, 100 sets per overlap setting). These simulated datasets provide the controlled ground truth against which transfer-learning clustering and KTA/ARI agreement are evaluated.

## When to use

Use when benchmarking source-to-target transfer clustering (e.g., NMF dictionary transfer + SC3) requires synthetic count matrices with exact ground-truth labels, or when comparing complete / incomplete / no-overlap source-target cell-type overlap settings.

## When NOT to use

- The analysis targets real published scRNA-Seq data (Tasic GSE71585, Usoskin GSE59739, Hockley GSE102962) — use the real-data wrappers (main_wrapper_tasic.py, main_wrapper_hockley.py) instead of simulated data generation.
- Ground-truth cell-type labels are already known for a real dataset and no synthetic benchmark is needed, since simulation is only required to recover supervised ARI values against known labels.

## Inputs

- Generation parameters (number of genes = 10,000, number of cells = 1,800, cluster range, overlap setting) as command line arguments (see scRNA wiki for detailed argument list)
- Optional string cell-type labels to embed as ground truth (script supports string labels since 2017.02)

## Outputs

- Gene ids file
- Source- and target count data files
- Source- and target ground truth label files
- Per-setting simulation batches (100 sets per overlap setting) for downstream NMF dictionary transfer and SC3 clustering

## How to apply

Install the Python 3 framework with `pip install git+https://github.com/nicococo/scRNA.git`, which exposes the MacOS/Linux-only command line script scRNA-generate-data.sh; its underlying implementation is scRNA/cmd_generate_data.py. Run the script (defaults suffice per the README example) in an empty working directory, which emits gene ids, source- and target count data, and source- and target ground-truth labels. For the paper's simulation study, generate 100 sets of simulated data for each of the three settings, simulating expression levels of 10,000 genes in 1,800 cells, with 1,000 cells reserved as source and the target downsampled from 800 cells. Then split each simulated set according to the cell-type overlap scenario (complete, incomplete, or no overlap) so the source dictionary Hsrc can be NMF-learned from labeled source cells and transferred to the target. Parameter selections for the generated-data experiments are documented in Supplementary Section 2.1 and the reproducibility script scripts/experiments/main_wrapper_generated_data.py; turn off t-SNE plotting with --no-tsne to speed up large runs.

## Related tools

- **scRNA (cmd_generate_data.py)** (Python framework whose command line script scRNA-generate-data.sh / cmd_generate_data.py produces the artificial scRNA-Seq count data, gene ids, and ground-truth labels used in the simulation study) — https://github.com/nicococo/scRNA/blob/master/scRNA/cmd_generate_data.py
- **SC3** (Downstream clustering method applied to the generated simulated datasets to benchmark the transfer learning approach; not used during generation itself)

## Examples

```
pip install git+https://github.com/nicococo/scRNA.git; cd /foo; scRNA-generate-data.sh
```

## Evaluation signals

- Run of scRNA-generate-data.sh in an empty directory must produce the documented file set: gene ids, source- and target data, and source- and target ground truth labels.
- Generated count matrices must have the paper's simulated dimensions: 10,000 genes and 1,800 cells per dataset.
- Per overlap setting, 100 simulated sets must be generated before running the transfer-learning comparison.
- Ground-truth labels must be recoverable so supervised ARI against simulation truth can be computed for each candidate theta (per Supplementary Fig. S4 workflow).
- The simulation output must feed cleanly into scRNA-source.sh (NMF of source data) and scRNA-target.sh (target clustering with transfer) steps of the README workflow.

## Limitations

- The command line scripts are available for MacOS and Linux only.
- With the Anaconda platform, an 'Intel MKL FATAL ERROR: Cannot load libmkl_avx.so or libmkl_def.so.' may occur, resolved by running conda install mkl first.
- Depending on the data and cluster range, generation and subsequent clustering steps can take a long time; disable t-SNE plots with --no-tsne to speed up.
- Pre-processing parameters for the generated-data experiments are only fully documented in Supplementary Section 2.1 and the experiment wrapper scripts, not in the paper's main methods text.

## Evidence

- [methods] 100 sets of simulated data were generated for each of the three settings simulating the expression levels of 10 000 genes in 1800 cells: "100 sets of simulated data were generated for each of the three settings simulating the expression levels of 10 000 genes in 1800 cells"
- [supplementary] The command line script for simulating scRNA-Seq datasets is available at https://github.com/nicococo/scRNA/blob/master/scRNA/cmd_generate_data.py: "The command line script for simulating scRNA-Seq datasets is available at https://github.com/nicococo/scRNA/blob/master/scRNA/cmd_generate_data.py"
- [readme] This package contains methods for generating artificial data, clustering, and transfering knowledge from a source to a target dataset.: "This package contains methods for generating artificial data, clustering, and transfering knowledge from a source to a target dataset."
- [readme] Generate some artificial data by simply calling the _scRNA-generate-data.sh_ (using only default parameters).: "Generate some artificial data by simply calling the _scRNA-generate-data.sh_ (using only default parameters)."
- [readme] This will result in a number of files: * Gene ids * Source- and target data * Source- and target ground truth labels: "This will result in a number of files: * Gene ids * Source- and target data * Source- and target ground truth labels"
