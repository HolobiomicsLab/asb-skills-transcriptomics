---
name: temporal-reconstruction-correlation-analysis
description: Use when a known confounding spatial signal must be filtered from single-cell
  expression data and recovery of a hidden temporal trajectory is quantified by Pearson
  correlation against reference temporal marker profiles.
when_to_use_negative:
- No prior knowledge of the signal to filter is available — SiFT strongly relies on
  the prior knowledge used to define the cell-cell similarity kernel
- The mapping (e.g. spatial cell-to-location assignment) is inaccurate or coarse-grained,
  which may lead to a loss of signal
- The goal is generic batch-effect integration in a joint latent representation (e.g.
  Harmony, scVI, bbknn) rather than filtering a specific biological signal at the
  gene-expression level
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SiFT (sift-sc)
  role: Filter the zonation signal via sift.SiFT with sift.kernels.KnnKernel (zonated
    genes) and sift.kernels.MappingKernel (novoSpaRc mapping)
  repo: https://github.com/nitzanlab/sift-sc
- name: scanpy
  role: Load the GSE145197 liver single-cell expression matrix, cell metadata, and
    spatial mapping
- name: novoSpaRc
  role: Provides the spatial cell-to-location mapping used to build the mapping kernel
    for zonation filtering
- name: pip
  role: Install SiFT via pip install sift-sc
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:4db2c34ce6aeba3b44175d464913140efe7b2f95b5d049c2b40aafe1430f36d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - SiFT
  - SiFT (sift-sc)
  - novoSpaRc
  - pip
  - scanpy
  - sift.SiFT
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# temporal-reconstruction-correlation-analysis

## Summary

Quantifies whether filtering a confounding spatial signal (liver zonation) from single-cell data with SiFT improves recovery of an underlying temporal circadian trajectory, measured by Pearson correlations of the reconstructed temporal signal against a reference profile across four filtering conditions.

## When to use

Use when a known confounding spatial signal must be filtered from single-cell expression data and recovery of a hidden temporal trajectory is quantified by Pearson correlation against reference temporal marker profiles.

## When NOT to use

- No prior knowledge of the signal to filter is available — SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel
- The mapping (e.g. spatial cell-to-location assignment) is inaccurate or coarse-grained, which may lead to a loss of signal
- The goal is generic batch-effect integration in a joint latent representation (e.g. Harmony, scVI, bbknn) rather than filtering a specific biological signal at the gene-expression level

## Inputs

- GSE145197 liver single-cell expression matrix with cell metadata, loadable with scanpy
- novoSpaRc spatial cell-to-location mapping used to build the mapping kernel
- Prior-knowledge list of zonated genes defining the spatial signal to filter
- Reference temporal profile and set of temporal marker genes (0 or 10 markers; e.g. hlf, elovl3)

## Outputs

- Pearson correlation table for the four conditions (SiFT-novoSpaRc 0 and 10 markers, zonated-genes-only, original data) written to fig4d_correlations.csv
- Qualitative gene-trend check results confirming hlf and elovl3 temporal trends in the filtered data

## How to apply

Load the GSE145197 liver single-cell dataset (expression matrix, cell metadata, and novoSpaRc spatial cell-to-location mapping) with scanpy. Build a cell-cell similarity knn kernel from prior-knowledge zonated genes using sift.kernels.KnnKernel and run sift.SiFT to project the expression matrix onto the zonation signal and deduct that projection; in parallel, repeat the filtering using sift.kernels.MappingKernel built from the novoSpaRc spatial mapping. Reconstruct the temporal (circadian/regeneration) signal from each filtered matrix plus the unfiltered original, using 0 and 10 reference temporal markers, and compute Pearson correlations of each reconstruction against the reference temporal profile. Judge success by matching the Fig. 4d targets (0.81 for SiFT-novoSpaRc with 0 markers, 0.96 with 10 markers, 0.71 for zonated-genes-only filtering with 10 markers, 0.44 for the original data) and by confirming that hlf and elovl3 temporal trends are qualitatively preserved while zonation-driven spatial variation is eliminated; write the correlation table and gene-trend checks to fig4d_correlations.csv.

## Related tools

- **SiFT (sift-sc)** (Filter the zonation signal via sift.SiFT with sift.kernels.KnnKernel (zonated genes) and sift.kernels.MappingKernel (novoSpaRc mapping)) — https://github.com/nitzanlab/sift-sc
- **scanpy** (Load the GSE145197 liver single-cell expression matrix, cell metadata, and spatial mapping)
- **novoSpaRc** (Provides the spatial cell-to-location mapping used to build the mapping kernel for zonation filtering)
- **pip** (Install SiFT via pip install sift-sc)

## Examples

```
pip install sift-sc
```

## Evaluation signals

- SiFT with novoSpaRc mapping and 0 temporal markers reproduces Pearson correlation 0.81; with 10 markers, 0.96
- Zonated-genes-only filtering with 10 markers yields Pearson correlation 0.71 and the unfiltered original data yields 0.44
- hlf and elovl3 temporal trends are qualitatively preserved in the filtered data
- Zonation-driven spatial variation is eliminated from the filtered expression matrix
- fig4d_correlations.csv contains the four-condition correlation table matching the reported values

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal
- No ground truth reference is available to evaluate computational filtering results; integration metrics are application-specific
- SiFT modifies gene expression so the filtered data are no longer properly log-transformed, which matters for downstream tasks expecting log-transformed data

## Evidence

- [abstract] SiFT enhances the liver circadian signal by filtering spatial zonation: "SiFT enhances the liver circadian signal by filtering spatial zonation"
- [intro] Prior-knowledge zonation filtering enhances the temporal circadian signal: "we use prior knowledge regarding liver zonation to filter the spatial signal from single-cell liver data, thereby enhancing the temporal circadian signal encoded by the cells"
- [discussion] SiFT depends on the prior knowledge defining the kernel: "it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel."
- [discussion] Coarse-grained mapping risks signal loss: "As we showcase in various examples, an inaccurate or coarse-grained mapping may lead to a loss of signal."
- [readme] SiFT is an open-source package for uncovering hidden biological processes: "Signal FilTering is a tool for uncovering hidden biological processes in single-cell data."
