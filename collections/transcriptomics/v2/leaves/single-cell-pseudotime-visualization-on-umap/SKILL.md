---
name: single-cell-pseudotime-visualization-on-umap
description: Use when a TFvelo velocity result (e.g. rc.h5ad from a completed run
  on pancreas or another dataset) needs pseudotime ordering, streamplot arrows on
  UMAP, and phase-portrait fitting plots for validation and manuscript figures.
when_to_use_negative:
- When the velocity model has not yet been run — the visualization step depends on
  the rc.h5ad output of TFvelo_run_demo.py
- When the input lacks spliced/unspliced or TF-derived dynamics and only raw counts
  are available without preprocessing via TFvelo_run_demo.py
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Produces the velocity/pseudotime estimates that TFvelo_analysis_demo.py visualizes
    on UMAP; provides the demo scripts for running and analysis
  repo: github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Framework on which TFvelo is built (scanpy/scvelo conventions for h5ad AnnData
    handling and UMAP-based visualization of velocities)
  repo: theislab/scvelo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:23d08df86f86e86c4b5490da4e1232e43e3d5479b5420851f7262c82400baafb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  - UniTVelo
  - VeloAE
  - velocyto
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# single-cell-pseudotime-visualization-on-umap

## Summary

Visualizes the pseudotime and RNA-velocity streamplot inferred by TFvelo on the UMAP embedding of a scRNA-seq dataset, alongside phase-portrait fits of the best fitted genes, using TFvelo_analysis_demo.py. This is the standard results-inspection and figure-production step that follows a TFvelo velocity run.

## When to use

Use when a TFvelo velocity result (e.g. rc.h5ad from a completed run on pancreas or another dataset) needs pseudotime ordering, streamplot arrows on UMAP, and phase-portrait fitting plots for validation and manuscript figures.

## When NOT to use

- When the velocity model has not yet been run — the visualization step depends on the rc.h5ad output of TFvelo_run_demo.py
- When the input lacks spliced/unspliced or TF-derived dynamics and only raw counts are available without preprocessing via TFvelo_run_demo.py

## Inputs

- AnnData h5ad file with TFvelo-fitted dynamics (e.g. TFvelo_pancreas/rc.h5ad) containing UMAP embedding, velocity fields, and fitted parameters
- --dataset_name argument identifying the dataset (e.g. 'pancreas' or a custom name registered in the preprocess() function)

## Outputs

- Annotated h5ad with pseudotime/velocity annotations (e.g. TFvelo_pancreas_demo/TFvelo.h5ad)
- Figure files (pseudotime on UMAP, streamplot on UMAP, phase-portrait fits of best fitted genes) saved in the 'figures' folder

## How to apply

After running TFvelo_run_demo.py, invoke TFvelo_analysis_demo.py with the same --dataset_name; the script reads the fitted result (e.g. 'TFvelo_pancreas/rc.h5ad') and renders pseudotime and streamplot on the UMAP embedding plus the phase-portrait fitting of best fitted genes. It writes the annotated result to 'TFvelo_pancreas_demo/TFvelo.h5ad' and saves all figures into a 'figures' folder. The same procedure generalizes to custom datasets: add a branch in the preprocess() function that reads your h5ad file, run both scripts with your dataset name, and outputs land in 'TFvelo_'+dataset_name+'_demo'. Correct application is judged by whether the UMAP shows a coherent pseudotime gradient and directed streamplot consistent with the fitted gene dynamics, which can then be compared against the paper's pancreas results (Fig. 2 panels).

## Related tools

- **TFvelo** (Produces the velocity/pseudotime estimates that TFvelo_analysis_demo.py visualizes on UMAP; provides the demo scripts for running and analysis) — github.com/xiaoyeye/TFvelo
- **ScVelo** (Framework on which TFvelo is built (scanpy/scvelo conventions for h5ad AnnData handling and UMAP-based visualization of velocities)) — theislab/scvelo

## Examples

```
python TFvelo_analysis_demo.py --dataset_name pancreas
```

## Evaluation signals

- Output h5ad exists at the expected path 'TFvelo_<dataset_name>_demo/TFvelo.h5ad' after running the analysis script
- Figure files for pseudotime, streamplot, and phase-portrait fits are present in the 'figures' folder
- For pancreas, the streamplot and pseudotime on UMAP should match the trajectory reported in the paper and support the Fig. 2d-g / Fig. S5 comparison panels
- Phase portraits of best fitted genes should show the 'desired clockwise curve' between TF representation and target gene

## Limitations

- TFvelo is slower than scVelo baseline approaches, which could be a weakness for large-scale datasets; using more CPUs in parallel (--n_jobs) can reduce running time before visualization
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14), so some phase portraits in the visualization may show poor fits

## Evidence

- [methods] python TFvelo_analysis_demo.py --dataset_name pancreas: "python TFvelo_analysis_demo.py --dataset_name pancreas"
- [readme] pseudotime and streamplot on UMAP plus phase portraits: "This will show the pseudotime and streamplot on UMAP, and also the phase portrait fitting of best fitted genes."
- [discussion] clockwise curve on phase portrait: "TFvelo can fit the gene dynamics well, showing a desired clockwise curve on the phase portrait between the TFs representation and the target"
- [discussion] parallelization caveat: "Performing TFvelo with more CPUs in parallel could reduce the running time"
