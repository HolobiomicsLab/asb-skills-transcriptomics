---
name: voronoi-tessellation-nuclei-seeded-segmentation
description: Use when a quick, membrane-agnostic whole-cell segmentation baseline
  from nuclei seeds is needed on multiplexed immunofluorescence or MIBI tissue images,
  especially to benchmark against membrane-aware segmenters or to quantify undersegmentation
  in crowded epithelia.
when_to_use_negative:
- Tissue contains columnar epithelial cells with elongated, irregular morphologies
  — the article states dilation-from-nuclei and Voronoi approaches 'are often insufficient'
  and that 'Voronoi undersegments cells towards the lumen as it does not take into
  account cell membranes'
- Cell boundaries follow visible membrane staining and accurate per-cell borders are
  the analysis goal — a membrane-aware pipeline (MIRIAM) or deep-learning segmenter
  (Mesmer) is preferred
- The input is a per-cell quantification table rather than image data — segmentation
  is not applicable
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Repository hosting the benchmark pipeline in which Voronoi tessellation was
    run as a comparison segmenter on the annotated TMA cores
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Implementation language used to run the nuclei-seeded Voronoi tessellation
    and compute DSC/JSC evaluation metrics
- name: Matlab
  role: Alternative MIRIAM implementation environment; results with Matlab or Python
    implementations show largely similar results
  repo: https://github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:b931430edabd85200dde77e7fcb16ce4464b586e72527202e8f232657492027f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  repo_url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Matlab
  - Mesmer
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# voronoi-tessellation-nuclei-seeded-segmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A nuclei-seeded Voronoi tessellation baseline for whole-cell segmentation of multiplexed tissue images, where each cell's territory is assigned by partitioning space outward from detected nuclei seeds without using membrane information. The article uses it as a benchmark to show why membrane-aware methods like MIRIAM are needed for columnar epithelial cells.

## When to use

Use when a quick, membrane-agnostic whole-cell segmentation baseline from nuclei seeds is needed on multiplexed immunofluorescence or MIBI tissue images, especially to benchmark against membrane-aware segmenters or to quantify undersegmentation in crowded epithelia.

## When NOT to use

- Tissue contains columnar epithelial cells with elongated, irregular morphologies — the article states dilation-from-nuclei and Voronoi approaches 'are often insufficient' and that 'Voronoi undersegments cells towards the lumen as it does not take into account cell membranes'
- Cell boundaries follow visible membrane staining and accurate per-cell borders are the analysis goal — a membrane-aware pipeline (MIRIAM) or deep-learning segmenter (Mesmer) is preferred
- The input is a per-cell quantification table rather than image data — segmentation is not applicable

## Inputs

- Multiplexed immunofluorescence or MIBI tissue image stacks (e.g., colon adenoma TMA cores with DAPI and membrane markers such as NaKATPase)
- Nuclei segmentation masks or DAPI channel to derive nuclei seeds
- Optionally, manual cell-border ground-truth annotations for evaluation

## Outputs

- Whole-cell segmentation masks (Voronoi territory per nucleus seed)
- Per-sub-image and per-cell DSC/JSC metrics when evaluated against manual annotations (e.g., written to segmentation_evaluation_metrics.csv)

## How to apply

First obtain nuclei segmentation masks (e.g., from DAPI channels in the TMA core images) to use as seeds. Then run nuclei-based Voronoi tessellation so that every pixel is assigned to the nearest nucleus seed, implicitly dilating each cell territory outward to the midpoint between neighboring seeds. Tile larger specimens into 512x512 sub-images (the article evaluated 240 sub-images across three manually annotated colon adenoma TMA cores) and compute per-sub-image and per-cell Dice (DSC) and Jaccard (JSC) similarity against manual cell-border annotations. In this paper's benchmark, Voronoi yielded cell-level DSC 0.47 ± 0.21 and JSC 0.33 ± 0.18, markedly below MIRIAM (DSC 0.67 ± 0.14, JSC 0.52 ± 0.15) and Mesmer (DSC 0.61 ± 0.19, JSC 0.46 ± 0.18), because it does not account for cell membranes.

## Related tools

- **MIRIAM** (Repository hosting the benchmark pipeline in which Voronoi tessellation was run as a comparison segmenter on the annotated TMA cores) — https://github.com/Coffey-Lab/MIRIAM
- **Python** (Implementation language used to run the nuclei-seeded Voronoi tessellation and compute DSC/JSC evaluation metrics)
- **Matlab** (Alternative MIRIAM implementation environment; results with Matlab or Python implementations show largely similar results) — https://github.com/Coffey-Lab/MIRIAM

## Examples

```
python run_segmentation_benchmark.py --method voronoi --images tma_cores/ --tile 512 --dapi DAPI --membrane NaKATPase --ground-truth manual_annotations/ --output segmentation_evaluation_metrics.csv
```

## Evaluation signals

- Cell-level DSC against manual annotations should fall near 0.47 ± 0.21 (JSC near 0.33 ± 0.18) on comparable 512x512 sub-images of annotated TMA cores; large deviations suggest seed or tiling errors
- Image-level DSC should be lower than cell-level DSC (article reports 0.32 ± 0.11 image-level vs 0.47 ± 0.21 cell-level) with a consistent gap
- Voronoi DSC/JSC must be below both MIRIAM and Mesmer on the same sub-images; if Voronoi matches or exceeds them, the seeds or ground-truth alignment are suspect
- Inspect regions toward lumens: systematic undersegmentation (single merged territories spanning multiple epithelial cells) is the expected failure signature, not a pipeline bug
- Sub-image count should match the tiling scheme (240 sub-images across three manually annotated TMA cores)

## Limitations

- Voronoi undersegments cells towards the lumen as it does not take into account cell membranes
- Existing methods like dilation from nuclei and Voronoi tessellation are often insufficient for segmenting cells with irregular morphologies such as columnar epithelial cells
- Segmentation quality depends entirely on nuclei seed accuracy; the article notes deep learning methods for nuclei detection were not implemented and may help improve initial seeds

## Evidence
