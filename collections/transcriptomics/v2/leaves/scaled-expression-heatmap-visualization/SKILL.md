---
name: scaled-expression-heatmap-visualization
description: Use when you need to visually validate temporal expression patterns of
  differentially expressed genes and their linked regulators (activators vs repressors)
  across omics layers from simulated or real multi-omics count data.
when_to_use_negative:
- The goal is single-cell data visualization — MOSim's sc_mosim validation uses Kendall's
  Tb correlation and k-medoids/Silhouette clustering instead, not this bulk heatmap
  workflow
- Input data are raw counts that need simulation or differential analysis first rather
  than already-selected DEGs with linked regulators for visualization
- The omics layers of interest are proteomics or metabolomics — MOSim does not support
  these layers
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0204
tools:
- name: MOSim
  role: Generates the simulated bulk multi-omics count data (RNA-seq, miRNA-seq, TF
    regulatory layers) and DE/regulatory metadata that feed the heatmap
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment for mean computation, log(x+1) + z-score scaling,
    and heatmap rendering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:2677549b6712c51b833eb9595ac202585d45852f82b9000fd12f7d5168717454
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - ProjecTILs
  - R
  - SPARSim
  - Seurat
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- averaged-expression-heatmap-visualization
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Scaled-expression heatmap visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Renders heatmaps of log(x+1)-transformed, row-wise z-scored group-mean expression profiles for selected genes/regulators across multi-omic layers (RNA-seq, miRNA-seq, TF), enabling visual validation of simulated activator/repressor regulatory patterns in multilayer GRN simulations.

## When to use

Use when you need to visually validate temporal expression patterns of differentially expressed genes and their linked regulators (activators vs repressors) across omics layers from simulated or real multi-omics count data.

## When NOT to use

- The goal is single-cell data visualization — MOSim's sc_mosim validation uses Kendall's Tb correlation and k-medoids/Silhouette clustering instead, not this bulk heatmap workflow
- Input data are raw counts that need simulation or differential analysis first rather than already-selected DEGs with linked regulators for visualization
- The omics layers of interest are proteomics or metabolomics — MOSim does not support these layers

## Inputs

- MOSim simulation object containing simulated count matrices and DE/regulatory metadata for RNA-seq, miRNA-seq and TF layers
- List of first 100 DEGs and their linked regulators in miRNA-seq and TF layers
- Design metadata: two experimental groups, three biological replicates, six time points

## Outputs

- Heatmap of row-wise z-scored, log(x+1)-transformed per-group mean expression profiles ordered by activator/repressor and temporal pattern
- Pearson correlation values for DEG–regulator pairs vs 20,000 random non-regulatory feature pairs (supporting validation statistics)

## How to apply

From the MOSim object, extract simulated count matrices and differential-expression/regulatory metadata, selecting the first 100 DEGs and their linked regulators in the miRNA-seq and TF layers. Compute per-group mean expression across the three biological replicates for the selected genes and regulators. Apply a log(x+1) transformation followed by row-wise z-score scaling to the mean profiles. Render a heatmap of the scaled mean-expression profiles per group, ordered to display temporal expression patterns of activators and repressors across the RNA-seq, miRNA-seq and TF layers (two experimental groups, six time points, 30M-read design). Validate the visualized activator/repressor consistency quantitatively by computing Pearson correlations between DEG–regulator pairs and comparing them against 20,000 random feature pairs with no simulated regulatory effects.

## Related tools

- **MOSim** (Generates the simulated bulk multi-omics count data (RNA-seq, miRNA-seq, TF regulatory layers) and DE/regulatory metadata that feed the heatmap) — github.com/ConesaLab/MOSim
- **R** (Implementation environment for mean computation, log(x+1) + z-score scaling, and heatmap rendering)

## Examples

```
Rscript -e 'mosim_res <- mosim(omics="RNA-seq", depth=30e6, groups=2, replicates=3, times=6, RNAseq_config=list(degs=0.05, reg IRS=0.4)); heat_expression(mosim_res, first_degs=100, scale="zscore", transform="log(x+1)")'
```

## Evaluation signals

- Heatmap shows distinct feature profiles across RNA-seq, miRNA-seq and TF layers that reflect both activator and repressor regulatory effects
- Row-wise z-scores of the scaled matrix have mean ~0 and unit standard deviation per gene row; values are non-negative before log(x+1) transform
- Pearson correlations of true DEG–regulator pairs are visibly separated from the distribution of 20,000 random feature pairs with no simulated regulatory effects
- Temporal ordering in the heatmap matches the six-time-point, two-group design and k-means/PCA clustering of simulated profiles aligns with simulation settings

## Limitations

- Restricted to bulk layers in this workflow; single-cell counterpart uses Kendall's Tb correlation and k-medoids clustering rather than this heatmap validation
- MOSim cannot simulate GRNs with spatial resolution or omics layers like proteomics and metabolomics, so the heatmap cannot cover such data
- Regulatory-effect visual separation depends on the simulation configuration (e.g. 5% DEGs, 40% repressors); with fewer repressors the activator/repressor contrast may be weak

## Evidence

- [results] The simulated multilayer GRN heatmaps (first 100 DEGs) show distinct feature profiles across RNA-seq, miRNA-seq and TF layers that accurately reflect both activator and repressor regulatory effects.: [evidence span withheld — anchor status: not_found]
- [methods] log transformation method: [evidence span withheld — anchor status: not_found]
