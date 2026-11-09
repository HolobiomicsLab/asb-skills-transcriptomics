---
name: phase-portrait-co-expression-interpretation
description: 'Use when validating RNA velocity or TF-based gene dynamics fits: after
  running scVelo/TFvelo on scRNA-seq (e.g., E15.5 mouse pancreas), check phase portraits
  of -fitted genes for the clockwise TF–target traversal that evidences regulation.'
when_to_use_negative:
- Genes for which TFvelo cannot fit the dynamics well (Fig. S14) — the phase portrait
  will not show a coherent clockwise curve and offers no validation signal
- Datasets lacking identifiable candidate TFs in ENCODE/ChEA, where a TF representation
  axis cannot be constructed for the target gene
- Large-scale datasets where TFvelo's low time efficiency relative to scVelo makes
  full portrait-fitting runs impractical without parallelizing over more CPUs
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Fits gene dynamics based on TF expression levels and renders phase portrait
    fits, pseudotime, and streamplots via TFvelo_analysis_demo.py
  repo: github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Baseline dynamical-model RNA velocity whose estimates are compared against
    the TF-derived dynamics on phase portraits
  repo: theislab/scvelo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:58ccf37ff682cde8b153cf2d8068e46e5558285b6a86b9963caacfeb636a2540
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Phase portrait co-expression interpretation

## Summary

Qualitative validation of TF-based gene dynamics by inspecting phase portraits between TF representation (or unspliced/spliced counts) and the target gene, where a correctly modeled regulatory relationship manifests as a desired clockwise co-expression curve.

## When to use

Use when validating RNA velocity or TF-based gene dynamics fits: after running scVelo/TFvelo on scRNA-seq (e.g., E15.5 mouse pancreas), check phase portraits of best-fitted genes for the clockwise TF–target traversal that evidences regulation.

## When NOT to use

- Genes for which TFvelo cannot fit the dynamics well (Fig. S14) — the phase portrait will not show a coherent clockwise curve and offers no validation signal
- Datasets lacking identifiable candidate TFs in ENCODE/ChEA, where a TF representation axis cannot be constructed for the target gene
- Large-scale datasets where TFvelo's low time efficiency relative to scVelo makes full portrait-fitting runs impractical without parallelizing over more CPUs

## Inputs

- Preprocessed scRNA-seq AnnData object with spliced/unspliced counts and moments (pp.moments), e.g., E15.5 mouse pancreas dataset as prepared by the TFvelo pipeline
- Velocity estimates from scVelo (dynamical model, scvelo==0.2.4) or TFvelo (rc.h5ad / TFvelo.h5ad outputs)
- TF-target relationships from ENCODE and ChEA (ENCODE.zip decompressed) used to define candidate TFs for each target gene

## Outputs

- Phase portrait fitting figures of best-fitted genes saved to the 'figures' folder
- Pseudotime and streamplot visualizations on UMAP
- Annotated h5ad result file (e.g., 'TFvelo_pancreas_demo/TFvelo.h5ad') containing fitted dynamics and pseudo-time

## How to apply

Run TFvelo (or scVelo's dynamical model as the splicing-based baseline) on the preprocessed dataset to obtain velocity estimates and fitted gene dynamics, then execute TFvelo_analysis_demo.py to render phase portrait fits of the best-fitted genes alongside pseudotime and streamplot on UMAP. On each phase portrait, verify that the fitted trajectory traces a clockwise curve from the TF representation to the target gene's RNA abundance, reflecting the temporal phase delay between regulator and target. Use these curves as qualitative validation supporting the quantitative LASSO result (non-zero TF weights predicting scVelo velocity, ENCODE TF-target enrichment p = 6.66e-06): genes whose TFs receive non-zero LASSO weights should exhibit coherent clockwise phase portraits, while counterclockwise, noisy, or non-cyclical portraits indicate a poor regulatory fit for that gene. The phase delay between TFs and target is itself informative — it can provide temporal (pseudo-time) information, motivating the regulation-based velocity model.

## Related tools

- **TFvelo** (Fits gene dynamics based on TF expression levels and renders phase portrait fits, pseudotime, and streamplots via TFvelo_analysis_demo.py) — github.com/xiaoyeye/TFvelo
- **ScVelo** (Baseline dynamical-model RNA velocity whose estimates are compared against the TF-derived dynamics on phase portraits) — theislab/scvelo

## Examples

```
python TFvelo_analysis_demo.py --dataset_name pancreas
```

## Evaluation signals

- Phase portraits of best-fitted genes show the desired clockwise curve between the TFs representation and the target gene
- Analysis run writes the annotated result to 'TFvelo_pancreas_demo/TFvelo.h5ad' and saves figures to the 'figures' folder
- Genes whose TFs carry non-zero LASSO weights for predicting scVelo velocity (ENCODE enrichment p = 6.66e-06) display coherent clockwise phase portraits, linking qualitative and quantitative validation
- Fitted dynamics are consistent with TFvelo-inferred pseudo-time and streamplot direction on UMAP
- Prior steps completed successfully: TFvelo_pancreas/rc.h5ad exists from the run stage before visualization

## Limitations

- TFvelo still cannot fit the dynamics of some genes well (Fig. S14); improving the modeling for more genes remains a challenging task
- The phase portrait is a qualitative check — it complements, but does not replace, quantitative scoring such as held-out correlation and the one-sided t-test on ENCODE TF-target enrichment
- TFvelo's time efficiency is lower than scVelo, which could be a weakness when applied to large-scale datasets; running with more CPUs in parallel can reduce runtime

## Evidence

- [discussion] TFvelo can fit the gene dynamics well, showing a desired clockwise curve on the phase portrait between the TFs representation and the target: "TFvelo can fit the gene dynamics well, showing a desired clockwise curve on the phase portrait between the TFs representation and the target"
- [readme] This will show the pseudotime and streamplot on UMAP, and also the phase portrait fitting of best fitted genes.: "This will show the pseudotime and streamplot on UMAP, and also the phase portrait fitting of best fitted genes."
- [discussion] TFvelo models the dynamics of a target gene with the expression level of TFs and the target gene itself: "TFvelo models the dynamics of a target gene with the expression level of TFs and the target gene itself"
- [readme] The result will be stored in 'TFvelo_pancreas/rc.h5ad'.: "The result will be stored in 'TFvelo_pancreas/rc.h5ad'."
