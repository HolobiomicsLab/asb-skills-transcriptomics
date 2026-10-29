---
name: scrnaseq-count-simulation
description: Use when an evaluation needs synthetic scRNA-Seq count data with known
  ground-truth cell-cluster labels — e.g. benchmarking clustering or transfer-learning
  pipelines where ARI can only be computed if true labels exist.
when_to_use_negative:
- Real scRNA-Seq datasets (e.g. Hockley, Usoskin, Tasic GEO data) are the analysis
  input — simulation parameters are for synthetic benchmarks, not for modifying real
  counts
- You need to transfer knowledge into an existing target dataset (NMF dictionary transfer,
  W'trg binarization, theta mixing) rather than generate synthetic data from scratch
- Running on Windows — the scRNA command-line scripts are available for MacOS and
  Linux only
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: scRNA (cmd_generate_data.py)
  role: Simulates count-level scRNA-Seq data with hierarchical cluster structure and
    ground-truth labels; invoked through the scRNA-generate-data.sh command-line wrapper
  repo: https://github.com/nicococo/scRNA
- name: SC3
  role: Downstream consumer of the simulated datasets — clustering of modified vs.
    unmodified target data is scored against the simulated ground truth via ARI
  repo: https://github.com/nicococo/scRNA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:25a681dbecd30c517d68e48ed1690b28c1e9139cffc46fec203a7a2baf354e71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - SC3
  - scRNA (cmd_generate_data.py)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# scrnaseq-count-simulation

## Summary

Generates synthetic scRNA-Seq count-level datasets with known ground-truth cluster labels using the scRNA package (cmd_generate_data.py / scRNA-generate-data.sh), producing hierarchical cluster structure (e.g. 8 sub-clusters from 5 top-level clusters) over ~10,000 genes in ~1800 cells for benchmarking clustering and transfer-learning methods.

## When to use

Use when an evaluation needs synthetic scRNA-Seq count data with known ground-truth cell-cluster labels — e.g. benchmarking clustering or transfer-learning pipelines where ARI can only be computed if true labels exist.

## When NOT to use

- Real scRNA-Seq datasets (e.g. Hockley, Usoskin, Tasic GEO data) are the analysis input — simulation parameters are for synthetic benchmarks, not for modifying real counts
- You need to transfer knowledge into an existing target dataset (NMF dictionary transfer, W'trg binarization, theta mixing) rather than generate synthetic data from scratch
- Running on Windows — the scRNA command-line scripts are available for MacOS and Linux only

## Inputs

- Simulation parameters (number of genes, cells, cluster hierarchy, DE fraction, fold-change distribution, NB dispersion) — defaults available via scRNA-generate-data.sh
- Target output directory for generated files

## Outputs

- Simulated scRNA-Seq count matrices for source and target datasets (~10,000 genes, 1800 cells per replicate)
- Gene ids (.tsv)
- Ground-truth source and target cluster labels (.tsv)
- Generated-data files used by scRNA-source.sh / scRNA-target.sh pipelines

## How to apply

Install the scRNA Python framework via pip (Python 3 only) and run the provided command-line script scRNA-generate-data.sh, which wraps scRNA/cmd_generate_data.py, in a working directory; the script writes gene ids, source/target count matrices, and ground-truth source/target labels. Control the simulated biology through the generation parameters used in the paper: a hierarchical structure of 8 sub-clusters derived from 5 top-level clusters, gamma-distributed background expression (shape 2, rate 0.1), 10-40% differentially expressed genes per cluster, log2 fold changes drawn from a normal distribution with mean 1 and sd 0.5, negative binomial sampling with dispersion 0.1, and ~10,000 genes. For the paper's simulation study, 100 replicate sets were generated per setting, each simulating 10,000 genes in 1800 cells, and the resulting data were split into source (fewer shared genes) and target (with additional target-only genes) sets before the standard pre-processing of cell filter, gene filter, and log-transformation after adding a pseudo-count of 1. The default cluster range used downstream by the framework is 6,7,8, matching the simulated sub-cluster counts.

## Related tools

- **scRNA (cmd_generate_data.py)** (Simulates count-level scRNA-Seq data with hierarchical cluster structure and ground-truth labels; invoked through the scRNA-generate-data.sh command-line wrapper) — https://github.com/nicococo/scRNA
- **SC3** (Downstream consumer of the simulated datasets — clustering of modified vs. unmodified target data is scored against the simulated ground truth via ARI) — https://github.com/nicococo/scRNA

## Examples

```
pip install git+https://github.com/nicococo/scRNA.git; scRNA-generate-data.sh
```

## Evaluation signals

- Generated output files exist as documented in the README: gene ids, source- and target data, and source- and target ground-truth labels
- Matrix dimensions match the paper's simulation settings (~10,000 genes, 1800 cells; 100 sets per setting when replicating the study)
- Ground-truth label structure reflects the specified hierarchy (8 sub-clusters derived from 5 top-level clusters), within the framework's default cluster range 6,7,8
- Counts are count-level (negative binomial, dispersion 0.1) and can be pre-processed by the standard pipeline: cell filter, gene filter, log-transform after pseudo-count of 1
- Downstream ARI comparison (TransferCluster vs. TargetCluster, overlapping 95% CIs as success criterion) is computable only if ground-truth labels were generated and retained

## Limitations

- The command-line scripts (including scRNA-generate-data.sh) are available for MacOS and Linux only; scRNA no longer supports Python 2
- Generation can be slow depending on data and cluster range; the README suggests disabling t-SNE plots (--no-tsne) to speed up related pipeline steps
- Anaconda users may hit the 'Intel MKL FATAL ERROR: Cannot load libmkl_avx.so or libmkl_def.so.' issue; fixed by running 'conda install mkl' first
- Simulated ground truth reflects the paper's assumed generative model (gamma background shape 2/rate 0.1, log2 FC ~ N(1, 0.5), NB dispersion 0.1); conclusions may not transfer to real-data characteristics

## Evidence

- [supplementary] Simulation script location: "The command line script for simulating scRNA-Seq datasets is available at https://github.com/nicococo/scRNA/blob/master/scRNA/cmd_generate_data.py"
- [methods] Simulation scale used in the study: "100 sets of simulated data were generated for each of the three settings simulating the expression levels of 10 000 genes in 1800 cells"
- [readme] Generated output artifacts: "This will result in a number of files: Gene ids, Source- and target data, Source- and target ground truth labels"
- [readme] Installation and default invocation: [evidence span withheld — anchor status: not_found]
- [readme] Platform constraint: "After successful completion, three command line arguments will be available for **MacOS and Linux only**"
