---
name: doubletrate-estimation-for-droplet-scrnaseq
description: Use when running Chord's chord() call (or simulating artificial doublets
  generally) on 10X-style droplet scRNA-seq data and an input 'doubletrate' must be
  supplied before doublet detection can proceed.
when_to_use_negative:
- Input is plate-based scRNA-seq or an already doublet-filtered dataset where no new
  artificial doublets need to be simulated
- 'The rate should not be copied across samples: it depends on the number of cells
  in the sample, so recompute it per sample rather than reusing a previous value'
edam_operation: http://edamontology.org/operation_3802
edam_topics: []
tools:
- name: Chord
  role: Consumes the estimated doubletrate to control simulated doublet generation
    for its GBM/AdaBoost ensemble doublet calls
  repo: https://github.com/13308204545/Chord
- name: DoubletFinder
  role: Installed dependency of Chord whose pN artificial-doublet proportion interacts
    with the doublet-rate setting
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:912fee10bf77703592ebaae0daec1d754aa1e407dfdb09ad92f580a0037011dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletFinder
  - R
  - Scrublet
  - Solo
  - adabag
  - gbm
  - remotes
  edam_operation: http://edamontology.org/operation_3802
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# doubletrate-estimation-for-droplet-scrnaseq

## Summary

Estimates the expected doublet rate for a droplet-based scRNA-seq sample from its cell count and passes it to ensemble doublet callers like Chord, which uses the rate to control how many artificial doublets are simulated for training the GBM/AdaBoost classifier.

## When to use

Use when running Chord's chord() call (or simulating artificial doublets generally) on 10X-style droplet scRNA-seq data and an input 'doubletrate' must be supplied before doublet detection can proceed.

## When NOT to use

- Input is plate-based scRNA-seq or an already doublet-filtered dataset where no new artificial doublets need to be simulated
- The rate should not be copied across samples: it depends on the number of cells in the sample, so recompute it per sample rather than reusing a previous value

## Inputs

- Seurat object ('seu') of a droplet-based 10X scRNA-seq sample with known cell count
- SingleCellExperiment object ('sce', converted via creatSCE) for scds-based methods
- Estimated doublet rate derived from cell count (~0.9% per 1,000 cells for 10X data)

## Outputs

- Configured chord() run producing outname_doublets.csv with doublet barcodes
- Simulated doublet training set sized according to the doubletrate (adddoublt parameter)

## How to apply

Determine the number of cells in the sample and apply the 10X heuristic that the doublet rate is approximately 0.9% per 1,000 cells, scaling the percentage with cell count. Pass this value as the doubletrate parameter of chord(seu, doubletrate, overkill=TRUE, outname), where it governs the number of simulated doublets (via the adddoublt parameter) generated from remaining cells to build the training set after the 'overkill' step removes top-ranked doublet-like cells (overkillrate, 0–1). On the second modular-boost run, supply an updated rate ('estimated doubletrate 2') alongside addmethods1/addmethods2 score tables. The rate is an estimate, not a measurement, so it directly conditions the class balance of simulated versus real cells that the GBM ensemble learns from.

## Related tools

- **Chord** (Consumes the estimated doubletrate to control simulated doublet generation for its GBM/AdaBoost ensemble doublet calls) — https://github.com/13308204545/Chord
- **DoubletFinder** (Installed dependency of Chord whose pN artificial-doublet proportion interacts with the doublet-rate setting) — https://github.com/chris-mcginnis-ucsf/DoubletFinder

## Examples

```
chord(seu = seu, doubletrate = 0.009 * ncol(seu)/1000 * 100, overkill = TRUE, outname = "sample1")
```

## Evaluation signals

- The doubletrate value scales with cell count: a 10X sample of N cells yields approximately 0.0009*N (fraction) as the expected rate
- The chord() run completes and emits 'outname_doublets.csv' containing a set of doublet barcodes of plausible size relative to the supplied rate
- Simulated training-set size (adddoublt) is consistent with doubletrate; changing the rate should visibly change the number of simulated doublets
- Downstream metrics (AUC, pAUC800/900/950/975) computed against experimental annotations remain comparable to reference values (e.g. Chord average AUC 0.801214286, ChordP 0.8132)

## Limitations

- The 0.9%-per-1,000-cells figure is a heuristic specific to 10X droplet chemistry, not a measurement from the sample itself
- Chord is not optimal in runtime because it integrates results of multiple methods; the rate-dependent simulation step contributes to this cost
- Chord's authors note that 'overkill' may improve other methods' performance but this was not tested, so rate-tuned overkill settings lack independent validation

## Evidence

- [readme] It depends on the number of cells in the sample. 10X can be referred：doubletrate = ~0.9% per 1,000 cells.: "It depends on the number of cells in the sample. 10X can be referred：doubletrate = ~0.9% per 1,000 cells."
- [methods] chord call signature: "chord（seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want"）"
- [discussion] overkill removal and training-set generation: "Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a"
- [discussion] runtime limitation: "Chord requires the integration of the results of multiple methods, so it is not optimal in terms of time efficiency."
