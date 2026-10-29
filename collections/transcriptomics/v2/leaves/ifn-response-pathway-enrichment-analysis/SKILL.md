---
name: ifn-response-pathway-enrichment-analysis
description: Use when scRNA-seq case-control data has been signal-filtered (e.g.,
  SiFT healthy-trajectory removal) and you need a gene-set readout to confirm which
  cell types carry the recovered disease signal.
when_to_use_negative:
- The data are raw unfiltered counts where the healthy signal has not been removed
  — the paper reports the same analysis on the original data failed to expose the
  relevant cell types.
- The biological question targets a pathway other than interferon response, with no
  prior knowledge linking it to the filtered signal.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SiFT (sift-sc)
  role: Produces the healthy-signal-filtered expression matrix on which IFN-response
    enrichment is computed
  repo: https://github.com/nitzanlab/sift-sc
- name: sift.kernels.KnnKernel
  role: Builds the cell-cell similarity kernel mapping each disease cell to its nearest
    healthy reference cells for the filtering step preceding enrichment
  repo: https://github.com/nitzanlab/sift-sc
- name: Python
  role: Implementation environment for the SiFT filtering and downstream gene-set
    enrichment analysis
- name: pip
  role: Installs the sift-sc package (pip install sift-sc)
  repo: https://github.com/nitzanlab/sift-sc
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:c210333ed2446fb5098a1f25461ef39df08ccc63f94e2f46adbec5bfc7e08d38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - pip
  - sift.kernels.KnnKernel
  - sift.sifter
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# IFN-response pathway enrichment analysis

## Summary

Scores interferon (IFN)-response gene-set enrichment in SiFT-filtered COVID-19 PBMC scRNA-seq data to verify that filtering the healthy reference signal exposes disease-related dynamics in innate immune cell types (monocytes, DCs, HSPC).

## When to use

Use when scRNA-seq case-control data has been signal-filtered (e.g., SiFT healthy-trajectory removal) and you need a gene-set readout to confirm which cell types carry the recovered disease signal.

## When NOT to use

- The data are raw unfiltered counts where the healthy signal has not been removed — the paper reports the same analysis on the original data failed to expose the relevant cell types.
- The biological question targets a pathway other than interferon response, with no prior knowledge linking it to the filtered signal.

## Inputs

- SiFT-filtered gene expression matrix for COVID-19 patient PBMC cells (healthy signal deducted)
- Original (unfiltered) disease-cell expression matrix for the control comparison
- IFN-response gene set
- Cell type annotations (e.g., CD14+ monocytes, CD16+ monocytes, DCs, HSPC, Plasmablasts, Platelets) from the Stephenson et al. PBMC dataset

## Outputs

- Per-cell-type IFN-response enrichment scores on filtered data
- Comparison of enrichment (filtered vs original) demonstrating recovery of innate immune cell types
- List of disease-informative cell types showing enhanced interferon response (monocytes, DCs, HSPC)

## How to apply

After SiFT projects disease-cell expression onto a KnnKernel built against healthy control PBMC reference cells and deducts the projection, run IFN-response gene-set scoring on the filtered expression matrix. Quantify enrichment per annotated cell type, expecting enhanced interferon-response signal in monocytes, dendritic cells, and HSPC populations — the pattern the original (unfiltered) Stephenson et al. data failed to expose. Compare enrichment between filtered and original data as the validation contrast. Note that SiFT modifies gene expression so the data are no longer properly log-transformed, a property expected in certain downstream tasks; account for this when applying standard enrichment tools.

## Related tools

- **SiFT (sift-sc)** (Produces the healthy-signal-filtered expression matrix on which IFN-response enrichment is computed) — https://github.com/nitzanlab/sift-sc
- **sift.kernels.KnnKernel** (Builds the cell-cell similarity kernel mapping each disease cell to its nearest healthy reference cells for the filtering step preceding enrichment) — https://github.com/nitzanlab/sift-sc
- **Python** (Implementation environment for the SiFT filtering and downstream gene-set enrichment analysis)
- **pip** (Installs the sift-sc package (pip install sift-sc)) — https://github.com/nitzanlab/sift-sc

## Examples

```
pip install sift-sc; from sift.kernels import KnnKernel; sft = sift.SiFT(kernel=KnnKernel(n_neighbors=30)); sft.fit(data=disease_cells, ref=healthy_cells); sft.transform(); # then score IFN gene set on filtered matrix
```

## Evaluation signals

- Enriched IFN response is recovered in monocytes, DCs, and HSPC on the filtered data, whereas the same analysis on the original data fails to expose these cell types.
- The qualitative pattern matches the paper's Fig. 6-7 summary generated from the healthy-trajectory-filtered data.
- Per-cell-type enrichment scores on filtered data should be highest in innate immune populations rather than uniformly distributed.
- The filtered expression matrix no longer resembles the joint PCA representation shared with healthy cells (batch-corrected, e.g., bbknn-style harmonized space).

## Limitations

- SiFT modifies the gene expression so that data are no longer properly log-transformed, a property expected in certain downstream analysis tasks; statistical implications of applying common analysis tools to corrected gene expression are unexplored.
- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal.
- Lacks a ground truth reference to evaluate the filtering results; integration metrics used for evaluation may be application-specific.

## Evidence

- [other] After SiFT filtering of the healthy signal, the representation recovered innate immune cell types (monocytes, DCs, and HSPC) via the interferon response (IFN) score, whereas the same analysis on the original data failed to expose the relevant cell types.: [evidence span withheld — anchor status: not_found]
- [discussion] SiFT modifies the gene expression so that data is no longer properly log-transformed, a property expected in certain downstream analysis tasks.: "it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks"
- [discussion] SiFT exposed disease-related signals and single-cell dynamics by filtering a corresponding healthy trajectory obtained by mapping to reference healthy samples.: "SiFT exposed disease-related signals and single-cell dynamics by filtering a corresponding healthy trajectory obtained by mapping to reference healthy samples"
- [readme] Install SiFT via PyPI by running: pip install sift-sc: "Install SiFT via PyPI by running::

    pip install sift-sc"
