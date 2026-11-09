---
name: expectation-maximization-parameter-estimation
description: Use when smoothed per-cell segment-level expression means from scRNA-seq
  data must be discretized into a fixed set of copy number states via unsupervised
  mixture-model fitting with EM.
when_to_use_negative:
- The tumor has minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas)
  so aneuploidy-based state discretization is unsuitable
- You need continuous copy number ratios rather than discretized states — use the
  CNA_mtx_relat ratios directly
- Segments have not yet been defined — EM requires the post-segmentation per-cell
  segment means, not raw gene-level counts
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: SCEVAN
  role: R package that computes per-cell segment means, fits the five-truncated-normal
    mixture via EM with fixed initial parameters, and aggregates calls by majority
    vote through pipelineCNA
  repo: github.com/AntonioDeFalco/SCEVAN
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:63c5beba1ff28a4dbd4aff04bf0ab1815e7de4556c3c11b7aa7641b2d69b73d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# expectation-maximization-parameter-estimation

## Summary

Per-cell genomic segment mean expression values are modeled as a mixture of five truncated normal distributions whose parameters are estimated by EM starting from empirically chosen fixed initial parameters; the resulting per-cell states are aggregated by majority vote across cells to assign each segment one of five copy number states (deletion, loss, neutral, gain, amplification).

## When to use

Use when smoothed per-cell segment-level expression means from scRNA-seq data must be discretized into a fixed set of copy number states via unsupervised mixture-model fitting with EM.

## When NOT to use

- The tumor has minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas) so aneuploidy-based state discretization is unsuitable
- You need continuous copy number ratios rather than discretized states — use the CNA_mtx_relat ratios directly
- Segments have not yet been defined — EM requires the post-segmentation per-cell segment means, not raw gene-level counts

## Inputs

- Per-cell segment mean expression matrix over joint segmentation breaks (CNA_mtx_relat from computeCNAmtx)
- Raw scRNA-seq count matrix with genes on rows and cells on columns (e.g. MGH106_data.RData from GSE131928)
- Genomic segment breakpoints from getBreaksVegaMC (Mumford–Shah joint segmentation)
- Fixed initial mixture parameters (Supplementary Table 1)

## Outputs

- Per-cell assignment of each genomic segment to one of five copy number states: deletion, loss, neutral, gain, amplification
- Consensus segment-level CNV call per segment via majority vote across cells
- Per-cell tumor/normal classification data frame from pipelineCNA

## How to apply

First compute per-cell segment mean expression over the genomic segments defined by SCEVAN's joint Mumford–Shah segmentation breaks (getBreaksVegaMC / computeCNAmtx) on the baseline-corrected count matrix, e.g. the MGH106 scRNA-seq counts from GSE131928. Model these per-cell segment means as a mixture of five truncated normal distributions corresponding to the states deletion, loss, neutral, gain, and amplification. Run EM starting from empirically chosen fixed initial parameters (Supplementary Table 1 of the article), since the states are predefined and truncated, rather than estimating the number of components. Assign each segment in each cell to exactly one of the five states, then aggregate per-cell calls by majority vote across cells to obtain a consensus segment-level CNV call. Validate that every segment in the output carries exactly one of the five state labels and that the calls are consistent with the per-cell tumor/normal classification returned by the pipelineCNA results data frame.

## Related tools

- **SCEVAN** (R package that computes per-cell segment means, fits the five-truncated-normal mixture via EM with fixed initial parameters, and aggregates calls by majority vote through pipelineCNA) — github.com/AntonioDeFalco/SCEVAN

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- The output table exists and every segment carries exactly one of the five state labels (deletion, loss, neutral, gain, amplification)
- Consensus segment calls are consistent with the per-cell tumor/normal classification in the pipelineCNA results data frame
- Mixture components correspond to the five predefined states in the expected order/ranges of relative expression, since components are truncated normals with empirically fixed initial parameters
- Subclone-specific CNV calls (SUBCLONES = TRUE) show coherent segment-level alterations shared across cells of the same clone, with segments <10 Mb apart representing the same CNA merged
- Majority-vote consensus is stable when re-run on the same input matrix with the same fixed initial parameters

## Limitations

- Initial mixture parameters are deferred to Supplementary Table 1 and were not verifiable in the provided excerpt
- The approach assumes aneuploidy and may not suit tumors with minimal genomic alterations
- A fixed five-state model is used; the number of components and states is not estimated from data

## Evidence

- [discussion] EM with fixed initialization: "starting from empirically chosen initial fixed parameters (Supplementary Table 1)."
- [methods] five truncated normal mixture: [evidence span withheld — anchor status: not_found]
- [results] five predefined copy number states by majority vote: [evidence span withheld — anchor status: not_found]
- [readme] SCEVAN package purpose: [evidence span withheld — anchor status: external_doc]
