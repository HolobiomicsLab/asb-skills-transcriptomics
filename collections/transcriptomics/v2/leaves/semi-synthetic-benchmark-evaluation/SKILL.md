---
name: semi-synthetic-benchmark-evaluation
description: Use when comparing signal-removal methods on a semi-synthetic single-cell
  dataset where a known nuisance signal (e.g., cell cycle) and a ground-truth biological
  attribute (e.g., subclone labels) are both annotated, so removal and conservation
  can be scored quantitatively.
when_to_use_negative:
- Data lacks any annotation of the nuisance signal (cell-cycle labels, scores, or
  Cyclum output), so no similarity kernel or baseline correction can be defined —
  SiFT 'strongly relies on the prior knowledge used to define the cell–cell similarity
  kernel'.
- The goal is batch-effect integration at the latent-representation level rather than
  gene-expression-level filtering of a biological nuisance signal.
- No quantitative metric or downstream separability signal exists to judge removal
  versus conservation, since evaluation must rely on integration metrics that 'may
  be application-specific'.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SiFT (sift-sc)
  role: 'Core filtering method: builds cell-cell similarity kernels (MappingKernel,
    KnnKernel) and projects/deducts the cell-cycle signal from the count matrix across
    six variants'
  repo: https://github.com/nitzanlab/sift-sc
- name: Cyclum
  role: Cell-cycle inference baseline and upper reference; provides phase bins and
    distances used as SiFT kernel inputs
  repo: https://github.com/KChen-lab/Cyclum
- name: pip
  role: Installation channel for the sift-sc package (pip install sift-sc)
- name: Python
  role: Implementation language in which kernels, SiFT runs, baselines, and score
    aggregation are scripted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:d3ce8b99f61ffe77e105ea4c520f6179325392870f1e17189df51020e79ffb0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Cyclum
  - Python
  - SiFT
  - SiFT (sift-sc)
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# semi-synthetic-benchmark-evaluation

## Summary

Quantitative benchmarking of signal-filtering methods (SiFT vs. cell-cycle removal baselines) on a semi-synthetic two-subclone mESC dataset (279 cells), scoring cell-cycle removal and biological conservation across six SiFT kernel variants and competing methods, with results assembled into fig3d_scores.csv.

## When to use

Use when comparing signal-removal methods on a semi-synthetic single-cell dataset where a known nuisance signal (e.g., cell cycle) and a ground-truth biological attribute (e.g., subclone labels) are both annotated, so removal and conservation can be scored quantitatively.

## When NOT to use

- Data lacks any annotation of the nuisance signal (cell-cycle labels, scores, or Cyclum output), so no similarity kernel or baseline correction can be defined — SiFT 'strongly relies on the prior knowledge used to define the cell–cell similarity kernel'.
- The goal is batch-effect integration at the latent-representation level rather than gene-expression-level filtering of a biological nuisance signal.
- No quantitative metric or downstream separability signal exists to judge removal versus conservation, since evaluation must rely on integration metrics that 'may be application-specific'.

## Inputs

- 279-cell mESC virtual tumor count matrix (counts, semi-synthetic two-subclone dataset)
- Per-cell cell-cycle annotations (ground truth mapping)
- Cell-cycle gene scores (e.g., Seurat-style scores) per cell
- Cyclum outputs: binned phases, phase distances, and Cyclum-corrected data (upper reference)

## Outputs

- Six SiFT-filtered expression matrices (one per kernel variant)
- Per-method cell-cycle removal score
- Per-method bio-conservation score
- Consolidated scores table: fig3d_scores.csv

## How to apply

Load the 279-cell mESC virtual tumor count matrix with per-cell cell-cycle annotations and construct multiple cell-cell similarity kernels capturing the nuisance signal: a sift.kernels.MappingKernel on ground-truth cell-cycle mapping, sift.kernels.KnnKernel on cell-cycle gene scores, KnnKernel and MappingKernel variants on Cyclum-binned phases/distances, and a MappingKernel on Seurat cell-cycle labels. For each kernel, run sift.SiFT to project the count matrix onto the cell-cycle signal and deduct the projection, yielding six filtered expression matrices. Run competing baselines on the same data — Seurat regress-out of cell-cycle scores, ccRemover, a regress_out-style covariate regression, and Cyclum as the upper reference. For every method compute a cell-cycle removal score and a bio-conservation score (used here in lieu of ground truth corrected data, per the article's noted evaluation challenge), then assemble all scores into a single table written to fig3d_scores.csv. Decision point: SiFT variants with ground-truth or Cyclum-based mappings achieved higher bio-conservation, so report per-kernel results rather than a single aggregate.

## Related tools

- **SiFT (sift-sc)** (Core filtering method: builds cell-cell similarity kernels (MappingKernel, KnnKernel) and projects/deducts the cell-cycle signal from the count matrix across six variants) — https://github.com/nitzanlab/sift-sc
- **Cyclum** (Cell-cycle inference baseline and upper reference; provides phase bins and distances used as SiFT kernel inputs) — https://github.com/KChen-lab/Cyclum
- **pip** (Installation channel for the sift-sc package (pip install sift-sc))
- **Python** (Implementation language in which kernels, SiFT runs, baselines, and score aggregation are scripted)

## Examples

```
pip install sift-sc; import sift; sft = sift.SiFT(kernel=sift.kernels.MappingKernel(mapping=cc_labels)); sft.filter(X)
```

## Evaluation signals

- fig3d_scores.csv contains one row per method (six SiFT variants plus Seurat regress-out, ccRemover, regress_out-style regression, and Cyclum reference) with both cell-cycle removal and bio-conservation scores.
- SiFT variants should attain quantitatively higher cell-cycle removal scores than the baseline methods on this benchmark.
- SiFT variants using ground-truth or Cyclum-based mappings for filtering should show higher bio-conservation scores than other variants.
- Running Cyclum as an upper reference should bound the achievable cell-cycle removal score; corrected methods at or above baselines but below the reference are consistent with the reported finding.
- All six filtered matrices must retain the 279-cell x gene dimensions of the original count matrix after projection and deduction.

## Limitations

- No ground truth reference exists to directly evaluate computational filtering results; evaluation must rely on integration metrics that may be application-specific.
- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal (relevant when using Seurat labels or binned Cyclum phases as kernels).
- Filtering modifies gene expression so the corrected data is no longer properly log-transformed, which matters for downstream tasks expecting log-transformed input.

## Evidence

- [discussion] When focusing on removing cell cycle effects, a major source of bias in single-cell data, in a semi-simulated setting: "When focusing on removing cell cycle effects, a major source of bias in single-cell data, in a semi-simulated setting"
- [discussion] To mitigate this, we suggest using integration metrics (which in general may be application-specific): "To mitigate this, we suggest using integration metrics (which in general may be application-specific)"
- [abstract] SiFT performs the correction at the gene expression level, can scale to large datasets, and compares favorably to state-of-the-art methods: "SiFT performs the correction at the gene expression level, can scale to large datasets, and compares favorably to state-of-the-art methods"
- [discussion] This kernel is then used to obtain a projection of gene expression onto the signal, which is then removed from the original expression.: "This kernel is then used to obtain a projection of gene expression onto the signal, which is then removed from the original expression."
- [readme] Install SiFT via PyPI by running:: pip install sift-sc: "Install SiFT via PyPI by running::

    pip install sift-sc"
