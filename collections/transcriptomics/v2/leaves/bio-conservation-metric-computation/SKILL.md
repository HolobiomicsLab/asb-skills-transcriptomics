---
name: bio-conservation-metric-computation
description: Use when evaluating a signal-filtering or cell-cycle-removal method on
  annotated single-cell data and you need to quantify, alongside a removal score,
  whether the corrected expression matrix preserves the biology of interest (e.g.,
  subclone structure).
when_to_use_negative:
- The task is to remove the unwanted signal itself (cell cycle) rather than to score
  how well a corrected matrix preserved the biology of interest — use the filtering
  skill instead.
- No per-cell annotations exist for either the signal to filter or the biology to
  conserve, so neither the SiFT kernel nor the conservation metric can be defined.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SiFT (sift-sc)
  role: Produces the corrected expression matrices (one per cell-cell similarity kernel)
    on which the bio-conservation and cell-cycle removal scores are computed.
  repo: https://github.com/nitzanlab/sift-sc
- name: Cyclum
  role: Supplies Cyclum-binned phases and circular pseudotime used to build two of
    the SiFT kernels and serves as the upper-reference baseline in the score table.
  repo: https://github.com/KChen-lab/Cyclum
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:514ffc991c5c842d06cd82fc8418589f85e9ccd1589ab20f0a0e14760e983012
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# bio-conservation-metric-computation

## Summary

Computes biological conservation scores on corrected single-cell expression data so that signal-removal methods (here, six SiFT kernel variants) can be judged not only on how much unwanted variation they remove but on how much of the biological signal of interest (e.g., subclone separability) they retain.

## When to use

Use when evaluating a signal-filtering or cell-cycle-removal method on annotated single-cell data and you need to quantify, alongside a removal score, whether the corrected expression matrix preserves the biology of interest (e.g., subclone structure).

## When NOT to use

- The task is to remove the unwanted signal itself (cell cycle) rather than to score how well a corrected matrix preserved the biology of interest — use the filtering skill instead.
- No per-cell annotations exist for either the signal to filter or the biology to conserve, so neither the SiFT kernel nor the conservation metric can be defined.

## Inputs

- Corrected/filtered gene expression count matrix (e.g., SiFT-projected-and-deducted 279-cell mESC virtual tumor counts)
- Per-cell biological labels of the signal to be conserved (two-subclone annotations of the semi-synthetic tumor dataset)
- Per-cell cell-cycle annotations used to build the filtering kernels (ground-truth mapping, Cyclum bins/phases, Seurat cell-cycle labels)

## Outputs

- Bio-conservation score per method/kernel variant computed on the corrected data
- Paired cell-cycle removal score per method for joint comparison
- Assembled comparison table written to fig3d_scores.csv covering all six SiFT variants and all baselines

## How to apply

Run the filtering method under comparison — here sift.SiFT with each of the six cell-cell similarity kernels (MappingKernel on ground-truth cell-cycle labels, KnnKernel on cell-cycle gene scores, KnnKernel and MappingKernel on Cyclum-binned phases, a kernel on Cyclum distances, and MappingKernel on Seurat cell-cycle labels) plus baselines (Seurat regress-out, ccRemover, a generic regress_out covariate regression, and Cyclum as the upper reference) — then compute both a cell-cycle removal score and a bio-conservation score on each corrected expression matrix. The bio-conservation score is calculated with respect to known biology (the two-subclone labels of the 279-cell mESC virtual tumor dataset), mirroring the integration-metric strategy the paper uses because no direct ground truth for filtering results exists. Assemble the paired scores for all SiFT variants and all baselines into a single table (fig3d_scores.csv) so removal vs. conservation can be traded off across methods. Expect kernel quality to drive conservation: mapping kernels built from ground-truth or Cyclum-derived labels should score higher on bio-conservation than coarse-grained mappings.

## Related tools

- **SiFT (sift-sc)** (Produces the corrected expression matrices (one per cell-cell similarity kernel) on which the bio-conservation and cell-cycle removal scores are computed.) — https://github.com/nitzanlab/sift-sc
- **Cyclum** (Supplies Cyclum-binned phases and circular pseudotime used to build two of the SiFT kernels and serves as the upper-reference baseline in the score table.) — https://github.com/KChen-lab/Cyclum

## Examples

```
import sift; sft = sift.SiFT(kernel=sift.kernels.MappingKernel(mapping=cell_cycle_labels), data=X); sft.filter()  # then compute removal and bio-conservation scores on sft.filteraed_data
```

## Evaluation signals

- Each of the six SiFT kernel variants and every baseline (Seurat regress-out, ccRemover, regress_out covariate regression, Cyclum) has both a removal score and a bio-conservation score in the assembled table.
- SiFT variants using ground-truth or Cyclum-based mapping kernels show higher bio-conservation scores than coarse-grained mappings, as reported in the finding.
- The corrected matrices cover the same 279 cells as the input virtual tumor count matrix (no cells dropped during filtering).
- The final table is written to fig3d_scores.csv and ranks SiFT quantitatively against baselines on the two-subclone benchmark.

## Limitations

- There is no ground truth reference for filtering results; only application-specific integration metrics (removal and bio-conservation scores) can be used for evaluation.
- SiFT (and hence the scores measured on its output) strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal and depressed conservation scores.
- SiFT modifies gene expression so the corrected data are no longer properly log-transformed, which can affect downstream tasks that expect log-transformed input.

## Evidence

- [abstract] abstract: "SiFT performs the correction at the gene expression level, can scale to large datasets, and compares favorably to state-of-the-art methods"
- [discussion] discussion: "To mitigate this, we suggest using integration metrics (which in general may be application-specific), as has been used throughout this manuscript"
- [intro] intro: "we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter"
