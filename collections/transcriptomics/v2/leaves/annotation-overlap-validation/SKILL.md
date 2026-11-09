---
name: annotation-overlap-validation
description: Use when automated region/community annotations (e.g. germinal centers
  from getCommunities/getBoundary) must be checked against an existing manual annotation
  to confirm boundaries and label agreement.
when_to_use_negative:
- No manual/reference annotation exists to compare against — the overlap assessment
  cannot be performed and the output is an unvalidated prediction
- Input is raw (non-deconvoluted) Visium counts — getCommunities clusters on deconvolved
  cell-type composition, so spot-type composition must first be produced with the
  standard Cell2location protocol
- Goal is TME analysis on multiplexed single-cell imaging data (e.g. MCMICRO-preprocessed
  MEL1) rather than validating region annotations on spot-level Visium data
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SpatialCells
  role: Provides getCommunities, getBoundary/bufferBoundary, assignPointsToRegions,
    and utils.exportRegion to generate and export the spot-level region annotations
    being validated
  repo: https://github.com/SemenovLab/SpatialCells
- name: Cell2location
  role: Upstream deconvolution of Visium data into spot-level cell-type composition,
    the input basis for community clustering and germinal-center labeling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:b939b7799f80b7ab2af134a3429af61756b061468cb43ebe7346243897747534
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
  - Cell2location
  - SpatialCells
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# annotation-overlap-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates automated spatial region annotations (e.g. germinal centers identified by SpatialCells on Cell2location-deconvoluted Visium data) by exporting spot-level region labels and assessing their overlap against published manual annotations.

## When to use

Use when automated region/community annotations (e.g. germinal centers from getCommunities/getBoundary) must be checked against an existing manual annotation to confirm boundaries and label agreement.

## When NOT to use

- No manual/reference annotation exists to compare against — the overlap assessment cannot be performed and the output is an unvalidated prediction
- Input is raw (non-deconvoluted) Visium counts — getCommunities clusters on deconvolved cell-type composition, so spot-type composition must first be produced with the standard Cell2location protocol
- Goal is TME analysis on multiplexed single-cell imaging data (e.g. MCMICRO-preprocessed MEL1) rather than validating region annotations on spot-level Visium data

## Inputs

- Cell2location-deconvoluted public Visium human lymph node spatial transcriptomics data loaded as an AnnData object
- Manual (author) spot-level germinal-center annotation used as the reference label set

## Outputs

- gc_annotations.csv — spot-level region annotation export with one annotation entry per spot
- Predicted germinal-center labels per spot for comparison against the manual annotation

## How to apply

Load the Cell2location-deconvoluted public Visium human lymph node data into an AnnData object and run spatialcells.spatial.getCommunities to cluster spots by deconvolved cell-type composition. Delineate community outlines with spatialcells.spatial.getBoundary (applying bufferBoundary where boundaries need adjustment), then assign every spot to a region with spatialcells.spatial.assignPointsToRegions, labeling the germinal-center communities. Export spot-level labels via spatialcells.utils.exportRegion to gc_annotations.csv. Correct application is judged by the CSV containing one annotation entry per spot and by qualitative consistency between predicted and manual germinal-center labels, mirroring the agreement the authors report for the Visium lymph node analysis.

## Related tools

- **SpatialCells** (Provides getCommunities, getBoundary/bufferBoundary, assignPointsToRegions, and utils.exportRegion to generate and export the spot-level region annotations being validated) — https://github.com/SemenovLab/SpatialCells
- **Cell2location** (Upstream deconvolution of Visium data into spot-level cell-type composition, the input basis for community clustering and germinal-center labeling)

## Examples

```
spatialcells.utils.exportRegion(adata, regions, 'region', 'gc_annotations.csv')
```

## Evaluation signals

- gc_annotations.csv exists after the export step
- gc_annotations.csv contains one annotation entry per Visium spot (no missing or duplicated spot annotations)
- All germinal centers in the dataset are identified and their region boundaries are delineated by getBoundary
- Overlap of predicted spot-level germinal-center labels with the manual annotation is qualitatively consistent with the agreement reported for the Visium lymph node analysis

## Limitations

- SpatialCells can currently only be installed from source (pip install -r requirements.txt; pip install ., or conda env create --file=conda.yaml)
- No software version is stated for SpatialCells in the article, complicating reproducibility of the validation
- The article reports the Visium lymph node comparison only as qualitative agreement with the authors' manual annotation rather than a quantitative agreement metric

## Evidence

- [results] Visium lymph node data deconvoluted with Cell2location before SpatialCells analysis: "The data were deconvoluted following the standard Cell2location protocol and analyzed with SpatialCells [31]."
- [results] Public Visium human lymph node dataset used for germinal-center analysis: "we further conducted experiments on a public Visium dataset of a human lymph node sample [31]."
- [other] Task workflow: export spot-level annotations to gc_annotations.csv for overlap comparison against the manual annotation: [evidence span withheld — anchor status: not_found]
- [readme] README: SpatialCells documentation and tutorials location and install procedure: [evidence span withheld — anchor status: not_found]
