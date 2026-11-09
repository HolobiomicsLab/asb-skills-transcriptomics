---
name: membrane-mask-overlap-metric-computation
description: Use when two segmentation algorithms or two implementations of the same
  pipeline have been run on identical multiplexed imaging data and you need a numeric
  overlap metric (DSC/JSC) to demonstrate cross-implementation reproducibility of
  membrane masks.
when_to_use_negative:
- You only have a single segmentation output and no paired masks from a second implementation
  or method to compare against.
- The comparison target is a manual ground-truth annotation for benchmarking segmentation
  accuracy (e.g., MIRIAM vs. hand-drawn membranes, DSC 0.65 ± 0.08), not cross-implementation
  consistency.
- The masks are nucleus or cytoplasm compartment masks rather than the membrane masks
  the pipeline is designed to resegment.
tools:
- name: MIRIAM
  role: Pipeline whose Matlab and Python implementations generate the paired membrane
    segmentation masks being compared
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: One of the two implementations used to produce reference membrane masks (run
    on manually annotated TMA core image stacks)
- name: Python
  role: The second implementation used to produce the comparison membrane masks on
    the identical image stacks
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:7cf674e4c56c958ad0b13c3d3cb7275bcb5717a1482c7150345f7c646f7196a4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Matlab
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# membrane-mask-overlap-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies agreement between membrane segmentation masks produced by two implementations (Matlab and Python) of the MIRIAM pipeline, using per-sub-image Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC) aggregated as mean ± standard deviation. It establishes that alternative software implementations yield largely similar cell segmentation on the same multiplexed immunofluorescence TMA cores.

## When to use

Use when two segmentation algorithms or two implementations of the same pipeline have been run on identical multiplexed imaging data and you need a numeric overlap metric (DSC/JSC) to demonstrate cross-implementation reproducibility of membrane masks.

## When NOT to use

- You only have a single segmentation output and no paired masks from a second implementation or method to compare against.
- The comparison target is a manual ground-truth annotation for benchmarking segmentation accuracy (e.g., MIRIAM vs. hand-drawn membranes, DSC 0.65 ± 0.08), not cross-implementation consistency.
- The masks are nucleus or cytoplasm compartment masks rather than the membrane masks the pipeline is designed to resegment.

## Inputs

- Membrane segmentation mask stack from the first implementation (e.g., Matlab MIRIAM) over TMA core sub-images
- Membrane segmentation mask stack from the second implementation (e.g., Python MIRIAM) on the identical sub-images
- Manually annotated TMA core image stacks (240 sub-images across three cores) as the shared input domain

## Outputs

- Per-sub-image agreement metrics: DSC, JSC (and optionally image correlation)
- Aggregated summary statistics: mean ± standard deviation DSC (0.84 ± 0.06) and JSC (0.74 ± 0.09)
- segmentation_agreement_metrics.csv containing per-image and summary agreement metrics

## How to apply

Run both implementations (e.g., MIRIAM in Matlab R2018b and in Python 3.7) on the identical manually annotated image stacks — in the reference study, 240 sub-images across three manually annotated TMA cores of multiplexed immunofluorescence data. For each sub-image, compute the Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC) between the membrane masks derived from each implementation, optionally alongside image-correlation. Aggregate per-image metrics into mean ± standard deviation values rather than relying on qualitative visual comparison. Judge success by whether the aggregated DSC and JSC reproduce the reported band (DSC 0.84 ± 0.06, JSC 0.74 ± 0.09), which supports the conclusion of 'largely similar results' between implementations.

## Related tools

- **MIRIAM** (Pipeline whose Matlab and Python implementations generate the paired membrane segmentation masks being compared) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (One of the two implementations used to produce reference membrane masks (run on manually annotated TMA core image stacks))
- **Python** (The second implementation used to produce the comparison membrane masks on the identical image stacks)

## Evaluation signals

- Mean membrane DSC across all sub-images should fall within the reported band 0.84 ± 0.06.
- Mean membrane JSC should fall within the reported band 0.74 ± 0.09.
- Metrics must be computed per sub-image across the full set (240 sub-images across three manually annotated TMA cores) before aggregating to mean ± standard deviation.
- Both implementations must be executed on the identical image stacks; masks must be spatially aligned to the same sub-image grid before overlap computation.
- Per-image and summary metrics should be reproducible when written to segmentation_agreement_metrics.csv.

## Limitations

- No software version numbers are stated for the Matlab or Python implementations in the article, so exact version reproduction may be ambiguous (the workflow card cites Matlab R2018b and Python 3.7).
- The pipeline requires Ilastik for some tasks, adding an external dependency that must be installed before masks can be generated.
- Example data in the repository omits the image stacks (too large); users must regenerate or download them separately, so the comparison cannot be run purely from the repo contents.
- The agreement result is demonstrated on membrane masks from multiplexed immunofluorescence TMA cores; transferability to other imaging modalities (e.g., MIBI) is asserted by the article but not quantified by this specific DSC/JSC comparison.

## Evidence
