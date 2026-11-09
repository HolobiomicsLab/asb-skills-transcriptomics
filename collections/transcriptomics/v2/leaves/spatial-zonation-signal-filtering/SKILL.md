---
name: spatial-zonation-signal-filtering
description: Use when single-cell liver data contains a spatial zonation signal that
  confounds reconstruction of a temporal (circadian/regeneration) trajectory, and
  prior knowledge of zonated genes or a cell-to-location spatial mapping is available
  to define the signal to remove.
when_to_use_negative:
- No prior knowledge of zonated genes and no spatial cell-to-location mapping exists,
  since SiFT strongly relies on the prior knowledge used to define the cell-cell similarity
  kernel
- The downstream task requires properly log-transformed expression, because SiFT-filtered
  data are no longer properly log-transformed
- The signal of interest is the spatial zonation itself rather than a hidden temporal
  trajectory — filtering would remove the target signal
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SiFT
  role: Kernel-based projection framework that projects expression onto the zonation
    signal and deducts it to yield filtered expression
  repo: https://github.com/nitzanlab/sift-sc
- name: sift.kernels.KnnKernel
  role: Cell-cell similarity kernel built from prior-knowledge zonated gene expression
    to define the zonation signal
  repo: https://github.com/nitzanlab/sift-sc
- name: sift.kernels.MappingKernel
  role: Kernel built from the novoSpaRc spatial cell-to-location mapping to define
    the zonation signal
  repo: https://github.com/nitzanlab/sift-sc
- name: pip
  role: 'Installation of the sift-sc package: pip install sift-sc'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:370fc7edd1f1312c9c9dab2e77f9146bd7377af1daaf8a5ce784fbfb754b33ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - SiFT
  - pip
  - sift.SiFT
  - sift.kernels.KnnKernel
  - sift.kernels.MappingKernel
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# spatial-zonation-signal-filtering

## Summary

Filter the spatial zonation signal from single-cell liver data with SiFT to enhance recovery of the underlying temporal circadian clock trajectory, using prior-knowledge zonated genes or a novoSpaRc spatial mapping to define the cell-cell similarity kernel.

## When to use

Use when single-cell liver data contains a spatial zonation signal that confounds reconstruction of a temporal (circadian/regeneration) trajectory, and prior knowledge of zonated genes or a cell-to-location spatial mapping is available to define the signal to remove.

## When NOT to use

- No prior knowledge of zonated genes and no spatial cell-to-location mapping exists, since SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel
- The downstream task requires properly log-transformed expression, because SiFT-filtered data are no longer properly log-transformed
- The signal of interest is the spatial zonation itself rather than a hidden temporal trajectory — filtering would remove the target signal

## Inputs

- GSE145197 liver single-cell expression matrix (scanpy AnnData)
- Cell metadata with spatial/sample annotations
- novoSpaRc cell-to-location spatial mapping matrix
- Prior-knowledge list of zonated genes used to build the KnnKernel
- Set of 0 or 10 reference temporal (circadian) marker genes, including hlf and elovl3
- Reference temporal circadian profile for correlation

## Outputs

- Zonation-filtered gene expression matrix (count matrix with spatial signal removed)
- Pearson correlation table of temporal reconstruction vs reference for four conditions (SiFT novoSpaRc 0/10 markers, SiFT zonated 10 markers, original), written to fig4d_correlations.csv
- Qualitative check of hlf and elovl3 temporal trends preserved in filtered data

## How to apply

Load the GSE145197 liver single-cell dataset (expression matrix, cell metadata, and novoSpaRc cell-to-location mapping) with scanpy and install SiFT via 'pip install sift-sc'. Build a cell-cell similarity kernel capturing similarity with respect to zonation — either sift.kernels.KnnKernel from expression of prior-knowledge zonated genes, or sift.kernels.MappingKernel from the novoSpaRc spatial mapping — then run sift.SiFT to project the expression count matrix onto the zonation signal and deduct the projection to obtain filtered expression. Reconstruct the temporal circadian signal from each filtered dataset (plus the unfiltered original), optionally with 0 or 10 reference temporal marker genes, and compute Pearson correlations against the reference temporal profile. Reproduced correlations should match Fig. 4d targets: SiFT novoSpaRc kernel with 0 markers = 0.81, with 10 markers = 0.96, SiFT zonated genes with 10 markers = 0.71, original unfiltered data = 0.44. Rationale: filtering the spatially variable zonation component removes the dominant unwanted signal so the weaker circadian program becomes recoverable.

## Related tools

- **SiFT** (Kernel-based projection framework that projects expression onto the zonation signal and deducts it to yield filtered expression) — https://github.com/nitzanlab/sift-sc
- **sift.kernels.KnnKernel** (Cell-cell similarity kernel built from prior-knowledge zonated gene expression to define the zonation signal) — https://github.com/nitzanlab/sift-sc
- **sift.kernels.MappingKernel** (Kernel built from the novoSpaRc spatial cell-to-location mapping to define the zonation signal) — https://github.com/nitzanlab/sift-sc
- **pip** (Installation of the sift-sc package: pip install sift-sc)

## Examples

```
pip install sift-sc; import sift; sft = sift.SiFT(kernel=sift.kernels.MappingKernel(adata=adata, mapping_key='novoSpaRc_mapping')); sft.filter()
```

## Evaluation signals

- Pearson correlation of temporal reconstruction matches Fig. 4d: 0.96 (SiFT novoSpaRc, 10 markers), 0.81 (SiFT novoSpaRc, 0 markers), 0.71 (SiFT zonated genes, 10 markers), 0.44 (original data)
- hlf and elovl3 temporal trends are qualitatively preserved after filtering
- Zonation-driven spatial variation is eliminated from the filtered expression
- Correlation table is written to fig4d_correlations.csv with the four condition rows

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal
- Filtering modifies gene expression so the data is no longer properly log-transformed, a property expected in certain downstream analysis tasks
- No ground truth reference exists to evaluate filtering results; evaluation relies on application-specific integration metrics such as the correlation targets

## Evidence

- [abstract] SiFT enhances the liver circadian signal by filtering spatial zonation: "SiFT enhances the liver circadian signal by filtering spatial zonation"
- [intro] Intro: filtering spatial signal enhances temporal circadian signal: "we use prior knowledge regarding liver zonation to filter the spatial signal from single-cell liver data, thereby enhancing the temporal circadian signal encoded by the cells"
- [methods] Methods: pip installation of SiFT: "Through **pip**::

    pip install sift-sc"
- [discussion] Discussion: reliance on prior knowledge of the kernel: [evidence span withheld — anchor status: not_found]
- [discussion] Discussion: log-transformation caveat: "it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks"
- [readme] README: SiFT purpose: "Signal FilTering is a tool for uncovering hidden biological processes in single-cell data."
