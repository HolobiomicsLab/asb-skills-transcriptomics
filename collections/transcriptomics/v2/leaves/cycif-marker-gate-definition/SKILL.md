---
name: cycif-marker-gate-definition
description: Use when a single-cell marker-intensity table (e.g., MCMICRO output from
  a CyCIF slide) needs binary marker calls before phenotyping or proliferation-index
  computation, and gates must be set explicitly per marker.
when_to_use_negative:
- Input is still multi-channel whole-slide imagery that has not been converted to
  single-cell data — MCMICRO preprocessing must happen first, since 'the output of
  MCMICRO is the input of SpatialCells'
- The goal is spatial region assignment (e.g., assignPointsToRegions, getCommunities)
  rather than per-marker intensity gating
- Markers of interest are not present among the acquired CyCIF antibody channels,
  so no intensity threshold can be defined
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SpatialCells
  role: Provides prep.setGate/setGates, the functions used to define per-marker intensity
    gates on the single-cell table
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream conversion of multi-channel whole-slide CyCIF images into the single-cell
    data that the gates are applied to
- name: AnnData
  role: Container format for the single-cell marker-intensity table that setGate/setGates
    operate on
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:2ee444903ee8a83f06849f6c60c3739d53b882a78948963906d8a5b586287fc0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/SemenovLab/SpatialCells
  repo_url: https://github.com/SemenovLab/SpatialCells
  tools:
  - AnnData
  - MCMICRO
  - SpatialCells
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# CyCIF marker gate definition

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Defines user-specified intensity thresholds (gates) for individual markers in a CyCIF single-cell intensity table using SpatialCells' prep.setGate/setGates, converting raw per-cell marker intensities into binary positive/negative calls that downstream measurements such as getMPI depend on.

## When to use

Use when a single-cell marker-intensity table (e.g., MCMICRO output from a CyCIF slide) needs binary marker calls before phenotyping or proliferation-index computation, and gates must be set explicitly per marker.

## When NOT to use

- Input is still multi-channel whole-slide imagery that has not been converted to single-cell data — MCMICRO preprocessing must happen first, since 'the output of MCMICRO is the input of SpatialCells'
- The goal is spatial region assignment (e.g., assignPointsToRegions, getCommunities) rather than per-marker intensity gating
- Markers of interest are not present among the acquired CyCIF antibody channels, so no intensity threshold can be defined

## Inputs

- AnnData object holding a CyCIF single-cell marker-intensity table (e.g., MEL1: 1,110,585 cells, 30 antibody channels, preprocessed with MCMICRO)
- User-specified intensity threshold per marker (e.g., gates for PHH3, Ki67, PCNA, MCM2, p21, p27)

## Outputs

- Per-cell binary gate calls (positive/negative) for each gated marker, stored in the AnnData object
- Per-cell MPI labels in {-1, 0, 1} when gates are consumed by spatialcells.msmt.getMPI (exportable as e.g. mel1_per_cell_mpi.csv)

## How to apply

Load the MCMICRO-derived single-cell table (e.g., the MEL1 CyCIF sample: 1,110,585 cells across 30 antibody channels) into an AnnData object, then call spatialcells.prep.setGate or setGates for each marker with a user-chosen intensity threshold — e.g., proliferation markers PHH3 (Serine-10 phosphorylated histone H3), Ki67, PCNA and MCM2, and arrest markers p21 and p27. The gate turns each cell's continuous intensity into a positive/negative call; for getMPI this feeds the piecewise rule where MPI = 1 when at least one proliferation marker exceeds its gate and no arrest marker does, MPI = -1 when an arrest marker exceeds its gate, and MPI = 0 otherwise. Gates must be set before applying msmt.getMPI, so gate choice directly controls the resulting per-cell labels. Validate gate sanity downstream: for MEL1, the fraction of SOX10+ tumor cells with MPI = 1 should reproduce the reported 7.4%.

## Related tools

- **SpatialCells** (Provides prep.setGate/setGates, the functions used to define per-marker intensity gates on the single-cell table) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream conversion of multi-channel whole-slide CyCIF images into the single-cell data that the gates are applied to)
- **AnnData** (Container format for the single-cell marker-intensity table that setGate/setGates operate on)

## Examples

```
from spatialcells.prep import setGates; setGates(adata, markers=['Ki67','PHH3','PCNA','MCM2','p21','p27'], gates=[...])  # user-specified intensity thresholds
```

## Evaluation signals

- Every cell receives an MPI label in {-1, 0, 1} after gates are consumed by getMPI, with MPI = -1 taking precedence when any arrest marker (p21/p27) exceeds its gate
- Fraction of SOX10+ tumor cells with MPI = 1 reproduces the reported reference value of 7.4% for the MEL1 sample
- Cells with at least one proliferation marker above gate but no arrest marker above gate are exactly those labeled MPI = 1
- Gate thresholds are explicitly recorded per marker (user-defined parameters), not silently defaulted
- Exported per-cell label file (e.g., mel1_per_cell_mpi.csv) has one row per input cell with no dropped cells

## Limitations

- Gates are user-specified intensity thresholds rather than automatically learned, so gate quality depends on the analyst's threshold choices
- SpatialCells can currently only be installed from source, and no software version is stated, affecting reproducibility of gate settings
- Gates only apply to markers present in the preprocessed single-cell table; excluding markers from gating changes the piecewise MPI assignment

## Evidence
