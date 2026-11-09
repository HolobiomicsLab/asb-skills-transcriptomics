---
name: phase-portrait-fitting-visualization
description: Use when you need to validate that a gene-regulation-based RNA velocity
  method fits gene dynamics correctly by inspecting phase portraits (TF representation
  vs. target) and UMAP pseudotime/streamplots, especially on multi-omics or scRNA-seq
  datasets.
when_to_use_negative:
- Your dataset lacks well-defined transcription-factor-gene regulatory structure,
  so phase portraits between TF representation and target will not be meaningful
- You only need velocity vector fields without validating gene-level dynamic fits
  — a plain scVelo streamplot suffices
- You are working with a very large-scale dataset where TFvelo's runtime is prohibitive
  compared to scVelo
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: TFvelo
  role: Produces the fitted dynamics and pseudotime that TFvelo_analysis_demo.py visualizes
    as phase portraits and UMAP streamplots
  repo: https://github.com/xiaoyeye/TFvelo
- name: scVelo
  role: Framework on which TFvelo is built; supplies the dynamical-modeling infrastructure
    and neighbor/UMAP utilities used in visualization
  repo: https://github.com/theislab/scvelo
- name: MultiVelo
  role: Baseline whose pseudotime, differentiation directions, and gene phase portraits
    (e.g., the erroneous cyclic AHI1 fit) are compared against TFvelo's visualizations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:e163e3aad8e848d5aeb114e9d7de50bc1a47b9c50e9368b4a406239c3fdf6ec4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaoyeye/TFvelo
  tools:
  - MultiVelo
  - TFvelo
  - scVelo
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# phase-portrait-fitting-visualization

## Summary

Visualize TFvelo's inferred pseudotime, streamplot on UMAP, and phase-portrait fits of target genes (e.g., AHI1, NTRK2, GRIN2B) to verify that TF-to-target dynamics follow the expected clockwise curve, and compare them against baselines like MultiVelo.

## When to use

Use when you need to validate that a gene-regulation-based RNA velocity method fits gene dynamics correctly by inspecting phase portraits (TF representation vs. target) and UMAP pseudotime/streamplots, especially on multi-omics or scRNA-seq datasets.

## When NOT to use

- Your dataset lacks well-defined transcription-factor-gene regulatory structure, so phase portraits between TF representation and target will not be meaningful
- You only need velocity vector fields without validating gene-level dynamic fits — a plain scVelo streamplot suffices
- You are working with a very large-scale dataset where TFvelo's runtime is prohibitive compared to scVelo

## Inputs

- AnnData (.h5ad) file with scRNA-seq counts, preprocessed by TFvelo's preprocess() function
- TFvelo result file rc.h5ad produced by TFvelo_run_demo.py
- Optional MultiVelo baseline output on the same dataset (10x multi-omics embryonic mouse brain)

## Outputs

- Annotated AnnData with inferred pseudotime (TFvelo.h5ad) in the '<save_name>_demo' folder
- UMAP pseudotime and streamplot figures saved in the 'figures' folder
- Phase-portrait fitting plots of target genes showing TF-to-target curves

## How to apply

After running TFvelo (TFvelo_run_demo.py with TF_databases ENCODE and ChEA, WX_method lsq_linear, init_weight_method correlation), execute TFvelo_analysis_demo.py to generate pseudotime and streamplot visualizations on UMAP plus phase-portrait fits of best-fitted genes; results are stored as TFvelo.h5ad in the '<save_name>_demo' folder and figures in the 'figures' folder. For new datasets, first add a load branch for your h5ad file into the preprocess() function of TFvelo_run_demo.py. For benchmarking against MultiVelo, run the baseline from the package's 'baselines' folder on the same AnnData object and plot phase portraits of target genes (e.g., AHI1, NTRK2, GRIN2B) under both methods. The key acceptance criterion is that a well-fitted gene shows a clockwise curve on the phase portrait between the TFs representation and the target; disagreement (e.g., MultiVelo's erroneous cyclic fit for AHI1) indicates a mis-specified dynamic.

## Related tools

- **TFvelo** (Produces the fitted dynamics and pseudotime that TFvelo_analysis_demo.py visualizes as phase portraits and UMAP streamplots) — https://github.com/xiaoyeye/TFvelo
- **scVelo** (Framework on which TFvelo is built; supplies the dynamical-modeling infrastructure and neighbor/UMAP utilities used in visualization) — https://github.com/theislab/scvelo
- **MultiVelo** (Baseline whose pseudotime, differentiation directions, and gene phase portraits (e.g., the erroneous cyclic AHI1 fit) are compared against TFvelo's visualizations)

## Examples

```
python TFvelo_analysis_demo.py --dataset_name your_dataset_name
```

## Evaluation signals

- A correctly fitted gene shows a clockwise curve on the phase portrait between the TFs representation and the target
- Inferred pseudotime ordering and stream direction on UMAP agree with the differentiation directions reported in Fig. 5a-c of the manuscript
- Result file TFvelo.h5ad is written to the '<save_name>_demo' folder and figures appear in the 'figures' folder
- On the 10x multi-omics mouse brain data, TFvelo's gene fits match or exceed MultiVelo's (e.g., no spurious cyclic dynamic for AHI1)
- Generated h5ad outputs for the chosen dataset_name are placed consistently in 'TFvelo_' + dataset_name + '_demo'

## Limitations

- TFvelo still cannot fit the dynamics of some genes well (Fig. S14), so some phase portraits will show poor fits
- TFvelo is slower than scVelo, which could be a weakness when being applied to large-scale datasets
- Reproducibility gap: the 10x multi-omics dataset is referenced without an accession identifier, so the exact input h5ad may need to be sourced independently

## Evidence

- [discussion] Phase portrait acceptance criterion: "TFvelo can fit the gene dynamics well, showing a desired clockwise curve on the phase portrait between the TFs representation and the target"
- [readme] Visualization command and outputs: "This will show the pseudotime and streamplot on UMAP, and also the phase portrait fitting of best fitted genes."
- [readme] Results storage locations: "As a result, all generated h5ad files will be puted in the folder named: "TFvelo_"+your_data_name+"_demo". And figures will be saved in the folder "figures"."
- [results] AHI1 comparison against MultiVelo: [evidence span withheld — anchor status: not_found]
- [readme] Customizing preprocess for new datasets: "you can define a personalized name for the dataset, and simply add the following codes into the preprocess() function in TFvelo_run_demo.py"
