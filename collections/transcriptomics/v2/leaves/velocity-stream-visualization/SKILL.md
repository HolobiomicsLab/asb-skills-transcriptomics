---
name: velocity-stream-visualization
description: Use when an AnnData object contains estimated velocities and cell-type
  labels and you need to visually verify that an inferred trajectory (e.g., OPC→OL
  maturation, retinal branches PR/RGC/AC-HC, Fev+→Delta transitions) is recovered.
when_to_use_negative:
- The AnnData lacks a velocity layer because utv.run_model or scVelo has not yet been
  fit — run the model first
- Quantitative comparison across methods is the goal alone — use utv.evaluate CBDir
  scores rather than relying only on stream plots
- Only per-gene phase-portrait dynamics are needed — use utv.pl.plot_range(gene_name,
  adata, velo, ...) instead of embedding streams
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scVelo
  role: Provides the plotting API (scv.pl.velocity_embedding_stream, scv.pl.scatter)
    used to render velocity streams and latent time on UniTVelo-processed AnnData;
    also serves as the baseline method whose streams are compared against UniTVelo
- name: UniTVelo
  role: Estimates gene velocities and unified/gene-specific latent time that populate
    the AnnData fields visualized in the stream plots; supplies utv.evaluate for CBDir
    metrics
  repo: github.com/StatBiomed/UniTVelo
- name: Python 3
  role: Runtime environment for UniTVelo, scVelo, and the plotting calls
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:5e9a81c4897fc5b9988284d88bb3bc98bf572ec48854459569a0aac7610f29c4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellRank
  - Python 3
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# velocity-stream-visualization

## Summary

Render RNA velocity fields as streamline embedding plots (and latent-time scatter plots) using scVelo's plotting API on AnnData objects processed by UniTVelo or scVelo, to visually assess recovered differentiation trajectories and branch structure.

## When to use

Use when an AnnData object contains estimated velocities and cell-type labels and you need to visually verify that an inferred trajectory (e.g., OPC→OL maturation, retinal branches PR/RGC/AC-HC, Fev+→Delta transitions) is recovered.

## When NOT to use

- The AnnData lacks a velocity layer because utv.run_model or scVelo has not yet been fit — run the model first
- Quantitative comparison across methods is the goal alone — use utv.evaluate CBDir scores rather than relying only on stream plots
- Only per-gene phase-portrait dynamics are needed — use utv.pl.plot_range(gene_name, adata, velo, ...) instead of embedding streams

## Inputs

- AnnData (.h5ad) with spliced/unspliced count matrices and velocity fields computed by UniTVelo (utv.run_model) or scVelo
- Cell-type label column name in adata.obs
- Latent time values stored in adata (color='latent_time') for time scatter plots

## Outputs

- Velocity embedding stream plot (PNG, dpi=100) colored by cell-type label
- Latent time scatter plot (color_map='gnuplot', size=20)
- Qualitative trajectory assessment against expected branch/transition structure (e.g., Supplementary Fig. S6)

## How to apply

After fitting UniTVelo via utv.run_model (or a scVelo baseline) on spliced/unspliced AnnData, call scv.pl.velocity_embedding_stream(adata, color=label, dpi=100, title='') where label is the column name in adata.obs specifying cell types. Render latent time with scv.pl.scatter(adata, color='latent_time', color_map='gnuplot', size=20). Judge the output qualitatively against expected biological orderings: e.g., the dentate gyrus stream should show a strengthened OPC-to-OL sub-lineage, the E15.5 retina stream should show cycling progenitors feeding three branches (PR, RGC, AC/HC) with a connected RGC terminal population (as in Supplementary Fig. S6), and the P20 pons stream should show the continuous OPC→COP→NFOL→MFOL path where scVelo's stream breaks down. For pancreas data, compare runs with and without cell-cycle regression for the subtle Fev+ pre-endocrine to Delta transition. Complement visual inspection with quantitative Cross-Boundary Direction Correctness via utv.evaluate(adata, cluster_edges, label, 'velocity').

## Related tools

- **scVelo** (Provides the plotting API (scv.pl.velocity_embedding_stream, scv.pl.scatter) used to render velocity streams and latent time on UniTVelo-processed AnnData; also serves as the baseline method whose streams are compared against UniTVelo)
- **UniTVelo** (Estimates gene velocities and unified/gene-specific latent time that populate the AnnData fields visualized in the stream plots; supplies utv.evaluate for CBDir metrics) — github.com/StatBiomed/UniTVelo
- **Python 3** (Runtime environment for UniTVelo, scVelo, and the plotting calls)

## Examples

```
import unitvelo as utv; import scvelo as scv; adata = utv.run_model(path_to_adata, label, config_file=velo); scv.pl.velocity_embedding_stream(adata, color=label, dpi=100, title=''); scv.pl.scatter(adata, color='latent_time', color_map='gnuplot', size=20)
```

## Evaluation signals

- Dentate gyrus (P12/P35) stream shows the expected OPC-to-OL ordering with strengthened sub-lineage signal matching Supplementary Fig. S6
- E15.5 retina stream shows cycling progenitor transitions feeding all three branches (PR, RGC, AC/HC) with the RGC terminal population connected, unlike scVelo's disconnected RGC path
- P20 pons stream shows the continuous OPC→COP→NFOL→MFOL maturation path where scVelo (stochastic/dynamical) does not; CBDir from utv.evaluate consistent with paper Table 1 (UniTVelo 0.609 vs scVelo 0.347 and 0.074)
- Pancreas stream (matching Supplementary Fig. S7d,e setting) shows a subtle Fev+ pre-endocrine to Delta transition with positive direction-correctness at that cluster edge
- Output files exist and CSV evaluation outputs conform to the expected format

## Limitations

- Stream plots are a qualitative acceptance check; the article notes existing velocity methods can return erroneous or disconnected trajectories, so visual inspection must be paired with metrics like CBDir
- No unsupervised method exists for selecting dynamically informed genes for projecting cell transitions, which can affect how the velocity vector is projected onto embeddings
- Possible over-counting in unspliced reads may affect cell transition metrics derived from the velocity vector
- Embedding plots depend on prior preprocessing (scVelo gene filtering and moment computation) and on configuration choices such as FIT_OPTION '1' vs '2' and R2_ADJUST

## Evidence

- [methods] Velocity embedding stream visualization: "scv.pl.velocity_embedding_stream(adata, color=label, dpi=100, title='')"
- [methods] Latent time estimation: "scv.pl.scatter(adata, color='latent_time', color_map='gnuplot', size=20)"
- [readme] README run model and plotting: "adata = utv.run_model(path_to_adata, label, config_file=velo)
scv.pl.velocity_embedding_stream(adata, color=label, dpi=100, title='')"
- [results] scVelo disconnected RGC path vs UniTVelo: [evidence span withheld — anchor status: not_found]
- [results] Pons CBDir quantitative check: [evidence span withheld — anchor status: not_found]
