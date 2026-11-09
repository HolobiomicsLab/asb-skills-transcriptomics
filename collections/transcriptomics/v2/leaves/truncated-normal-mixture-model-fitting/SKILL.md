---
name: truncated-normal-mixture-model-fitting
description: Use when a per-cell segment-level smoothed expression matrix (e.g. CNAmtx
  from SCEVAN's joint segmentation) must be discretized into the five CNV states via
  an EM-fitted truncated-normal mixture rather than ad hoc thresholding.
when_to_use_negative:
- The tumor has minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas),
  where the aneuploidy-based mixture assumption does not hold
- Input is still a raw scRNA-seq count matrix without prior baseline correction and
  joint segmentation — the mixture must be fit to smoothed segment means, not raw
  counts
- Only malignant/non-malignant classification is needed with no segment-level CNV
  state assignment (run pipelineCNA without SUBCLONES instead)
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCEVAN
  role: Implements the five-state truncated normal mixture model with EM inside its
    CNV calling pipeline (pipelineCNA / classifyTumorCells); consensus calls come
    from majority vote across cells
  repo: github.com/AntonioDeFalco/SCEVAN
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:e3ea6d50004c6165afb9c40b72d2115b5ef53e415ab20cc132f1247a2feb7dc7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# truncated-normal-mixture-model-fitting

## Summary

Fits a mixture of five truncated normal distributions to per-cell segment mean expression profiles and assigns each genomic segment to one of five copy number states (deletion, loss, neutral, gain, amplification), as done inside SCEVAN's CNV-calling workflow, with a final consensus call by majority vote across cells.

## When to use

Use when a per-cell segment-level smoothed expression matrix (e.g. CNAmtx from SCEVAN's joint segmentation) must be discretized into the five CNV states via an EM-fitted truncated-normal mixture rather than ad hoc thresholding.

## When NOT to use

- The tumor has minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas), where the aneuploidy-based mixture assumption does not hold
- Input is still a raw scRNA-seq count matrix without prior baseline correction and joint segmentation — the mixture must be fit to smoothed segment means, not raw counts
- Only malignant/non-malignant classification is needed with no segment-level CNV state assignment (run pipelineCNA without SUBCLONES instead)

## Inputs

- Per-cell segment mean expression matrix over genomic segments defined by joint segmentation breaks (CNAmtx output of computeCNAmtx / getBreaksVegaMC)
- Relative expression matrix after subtraction of the copy number baseline (median of confident normal cells)
- Initial mixture parameters from Supplementary Table 1 (empirically fixed starting values for the EM)

## Outputs

- Per-cell classification of each segment into one of five states: deletion, loss, neutral, gain, amplification
- Consensus per-segment CNV call table obtained by majority vote across cells
- Consistency with the per-cell tumor/normal classification returned by the pipelineCNA results data frame

## How to apply

After SCEVAN's preprocessing (filter <200 detected genes, REACTOME cell-cycle gene removal, baseline subtraction of confident normal cells, Mumford–Shah segmentation via getBreaksVegaMC/computeCNAmtx), take the per-cell segment mean values and model them as a mixture of five truncated normal distributions corresponding to deletion, loss, neutral, gain, and amplification states. Estimate the mixture parameters by expectation–maximization (EM) starting from the empirically chosen fixed initial parameters listed in Supplementary Table 1 of the paper. Assign each cell/segment to exactly one of the five states from the fitted responsibilities, then aggregate per-cell calls by majority vote across cells to obtain the consensus segment-level CNV call. The rationale is that smoothed expression levels for each state are approximately normal but bounded, so truncated normals avoid assigning probability mass outside physically meaningful log-ratios.

## Related tools

- **SCEVAN** (Implements the five-state truncated normal mixture model with EM inside its CNV calling pipeline (pipelineCNA / classifyTumorCells); consensus calls come from majority vote across cells) — github.com/AntonioDeFalco/SCEVAN

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Output table exists and every segment carries exactly one of the five state labels (deletion, loss, neutral, gain, amplification) — no segment unlabeled or multiply assigned
- Per-segment consensus calls are consistent with the per-cell tumor/normal classification in the pipelineCNA results data frame
- EM converges from the fixed initial parameters of Supplementary Table 1 (parameters are reproducible across runs because initialization is fixed)
- Majority-vote aggregation yields a single consensus state per segment across all classified cells
- For MGH106 (GSE131928), the resulting segment calls should reproduce the documented workflow output on the MGH106_data.RData count matrix

## Limitations

- The initial mixture parameters are only in Supplementary Table 1 and are not included in the main text, so reproduction requires access to the supplement
- Aneuploidy-based assumption is unsuitable for tumors with minimal genomic alterations such as leukemia, pediatric cancers, and Ependymomas
- Cell-cycle-driven expression artifacts can create artificial segments unless REACTOME cell-cycle genes are filtered first

## Evidence

- [methods] five truncated normal mixture states via EM from fixed init: [evidence span withheld — anchor status: not_found]
- [discussion] initial parameters deferred to supplement: "starting from empirically chosen initial fixed parameters (Supplementary Table 1)."
- [discussion] aneuploidy assumption limitation: "Thus, our approach (and similar) may not be suited in this case."
- [readme] concrete SCEVAN invocation on MGH106: [evidence span withheld — anchor status: external_doc]
