---
name: doublet-rate-estimation-per-cell-count
description: Use when running Chord (or any simulated-doublet method) on droplet scRNA-seq
  data without experimental doublet labels (no demuxlet/cell hashing) and you must
  supply an expected doubletrate before calling doublets.
when_to_use_negative:
- Experimental doublet labels already exist (e.g., demuxlet called doublets from natural
  genetic variation, or cell hashing/MULTI-seq was performed) — use those directly
  instead of an estimated rate
- The dataset is not droplet-based microfluidic scRNA-seq, where the doublet-counted-as-one-cell
  artefact and the 10X rate rule do not apply
- The sample is a merged multi-sample object whose cell-type proportions are not representative
  of any single sample (cf. Scrublet best practice to run per sample)
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Chord
  role: Consumes the estimated doubletrate in chord() and overkillDB2() to size the
    overkill removal fraction and the artificial-doublet training set
  repo: https://github.com/13308204545/Chord
- name: DoubletFinder
  role: Built-in overkill method whose pN convention (0.25 of merged real-artificial
    data) is analogous to the doubletrate-driven artificial doublet proportion
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Provides bcds/cxds scores used in the overkill stage that removes doubletrate
    × overkillrate of most-likely doublets
  repo: https://github.com/kostkalab/scds
- name: demuxlet
  role: 'Contrast case: genetic-variation-based doublet identification that bypasses
    the need for rate estimation when genotype data are available'
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:c742881a57393515b2ab836eca29fd3c4a601b3e69f6ebefe387e1403da97024
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletFinder
  - R
  - adabag
  - creatSCE
  - demuxlet
  - gbm
  - overkillDB2
  - remotes
  - scds
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# doublet-rate-estimation-per-cell-count

## Summary

Estimates the expected doublet rate for a droplet scRNA-seq sample from its cell count using the 10X rule of thumb (~0.9% per 1,000 cells), supplying the doubletrate parameter that Chord, its overkill filtering stage, and simulated-doublet generation all depend on.

## When to use

Use when running Chord (or any simulated-doublet method) on droplet scRNA-seq data without experimental doublet labels (no demuxlet/cell hashing) and you must supply an expected doubletrate before calling doublets.

## When NOT to use

- Experimental doublet labels already exist (e.g., demuxlet called doublets from natural genetic variation, or cell hashing/MULTI-seq was performed) — use those directly instead of an estimated rate
- The dataset is not droplet-based microfluidic scRNA-seq, where the doublet-counted-as-one-cell artefact and the 10X rate rule do not apply
- The sample is a merged multi-sample object whose cell-type proportions are not representative of any single sample (cf. Scrublet best practice to run per sample)

## Inputs

- Seurat object (seu) for a single droplet scRNA-seq sample with per-cell barcodes
- Cell count of the sample (number of droplets/cells captured)
- Optional overkillrate (fraction of top-scoring cells removed per method in overkill)

## Outputs

- doubletrate value passed to chord()/overkillDB2()
- Predicted per-sample doublet proportions (e.g., outname_doublets.csv barcodes and per-sample rates)
- Merged real-artificial training set (overkilled.robj) containing artificial doublets at the expected proportion set by doubletrate

## How to apply

Because experimental doublet identification like demuxlet, cell hashing, or MULTI-seq is unavailable for existing data, estimate the doublet rate computationally from sample size: use the 10X guideline of ~0.9% per 1,000 cells, scaling linearly with the number of cells captured. Pass this value as the doubletrate argument to chord() (e.g., a 24,280-cell NSCLC sample yields roughly 2.2%, and Chord's estimates for the five lung-tumour samples landed at 3.81–4.74%). The same rate drives two downstream steps: the overkill stage removes the top doubletrate × overkillrate fraction of cells flagged by DoubletFinder, bcds and cxds, and overkillDB2 samples artificial doublet pairs at a ratio of doubletrate/(1−doubletrate) relative to real cells (analogous to DoubletFinder's pN convention, default 0.25 of the merged real-artificial data). The estimate is an expected rate, not a fitted one — Chord's GBM ensemble refines per-cell calls on top of it, so modest misestimation shifts the removed fraction rather than invalidating the pipeline.

## Related tools

- **Chord** (Consumes the estimated doubletrate in chord() and overkillDB2() to size the overkill removal fraction and the artificial-doublet training set) — https://github.com/13308204545/Chord
- **DoubletFinder** (Built-in overkill method whose pN convention (0.25 of merged real-artificial data) is analogous to the doubletrate-driven artificial doublet proportion) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **scds** (Provides bcds/cxds scores used in the overkill stage that removes doubletrate × overkillrate of most-likely doublets) — https://github.com/kostkalab/scds
- **demuxlet** (Contrast case: genetic-variation-based doublet identification that bypasses the need for rate estimation when genotype data are available)

## Examples

```
chord(seu = seu, doubletrate = 0.009 * ncol(seu) / 1000, overkill = TRUE, outname = "sample11")
```

## Evaluation signals

- Per-sample doublet rates fall in a plausible band around the expected-rate calculation (e.g., the five NSCLC samples yielded 3.81%, 4.68%, 4.74%, 4.20%, and 4.42%)
- The fraction of cells removed per method in overkill equals doubletrate × overkillrate
- The merged real-artificial training object contains artificial doublets at the proportion set by doubletrate (default pN = 0.25 convention)
- overkilled.robj loads as a Seurat object containing the retained cells, and <outname>_doublets.csv exists with the expected barcode column
- Per-cell-type doublet proportions are biologically ordered as expected (e.g., fibroblasts low at 1.69%, myeloid highest at 8.15% in the NSCLC application)

## Limitations

- The ~0.9% per 1,000 cells value is a 10X-specific rule of thumb; the correct rate 'depends on the number of cells in the sample' and capture chemistry
- Chord is not optimal in runtime because it must integrate results of multiple methods, and refactoring the integrated methods is proposed but not implemented
- Other pre-processing steps such as ambient mRNA removal for droplets are needed to improve downstream accuracy but are not incorporated
- The claim that the overkill step might also improve other methods' performance is unverified

## Evidence

- [readme] Rate estimation is sample-size dependent per the README: "Q:how to estimate doubletrate? A:It depends on the number of cells in the sample. 10X can be referred：doubletrate = ~0.9% per 1,000 cells."
- [methods] doubletrate is an argument of chord(): "chord（seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want"）"
- [discussion] doubletrate drives overkill removal and simulated-doublet training-set generation: "Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a"
- [intro] Droplet methods encounter doublets counted as single cells, motivating rate-based detection: [evidence span withheld — anchor status: not_found]
