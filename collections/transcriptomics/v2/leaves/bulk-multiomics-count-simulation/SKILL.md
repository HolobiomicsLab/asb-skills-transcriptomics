---
name: bulk-multiomics-count-simulation
description: Use when you need synthetic bulk RNA-seq count matrices with coupled
  regulatory omics layers and ground-truth regulatory interactions to benchmark GRN
  inference, multi-omics integration pipelines, or DE/clustering methods.
when_to_use_negative:
- When single-cell multi-omics simulation is needed beyond scRNA-seq and scATAC-seq
  — sc_mosim is restricted to those two layers and does not cover ChIP-seq, Methyl-seq
  or miRNA-seq
- When spatial resolution of the GRN is required, or when simulating proteomics/metabolomics
  layers, which MOSim does not support
- When the input is already a feature/count table for direct analysis rather than
  a need for synthetic ground-truth data generation
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: 'Core simulator: mosim() generates bulk RNA-seq plus ATAC-seq, miRNA-seq,
    ChIP-seq, Methyl-seq and TF regulatory layers with configurable design, DEGs and
    active regulators'
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Runtime environment; MOSim is a Bioconductor R package installed via BiocManager
    and driven from R scripts
- name: SPARSim
  role: Count-data simulation functionality underlying sc_mosim single-cell simulation
    (cite when the sc module is used)
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:da6e48c2359662f0ffd6fb75c0c635dada08b60804a3d954880e83e789792a85
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
  - R
  - SPARSim
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bulk-multiomics-count-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate bulk multi-omics count data (RNA-seq with ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq and TF regulatory layers) seeded from real multi-omics data such as the STATegra mouse dataset, using the MOSim R package's mosim() function to generate ground-truth data with known DEGs, temporal co-expression patterns, and active gene–regulator relationships.

## When to use

Use when you need synthetic bulk RNA-seq count matrices with coupled regulatory omics layers and ground-truth regulatory interactions to benchmark GRN inference, multi-omics integration pipelines, or DE/clustering methods.

## When NOT to use

- When single-cell multi-omics simulation is needed beyond scRNA-seq and scATAC-seq — sc_mosim is restricted to those two layers and does not cover ChIP-seq, Methyl-seq or miRNA-seq
- When spatial resolution of the GRN is required, or when simulating proteomics/metabolomics layers, which MOSim does not support
- When the input is already a feature/count table for direct analysis rather than a need for synthetic ground-truth data generation

## Inputs

- Seed multi-omics dataset (e.g. STATegra mouse multi-omics data)
- Experimental design specification: number of reads (e.g. 30M), groups, biological replicates, time points
- DEG fraction and repressor/significant-regulation percentages
- Optional temporal co-expression profile definitions

## Outputs

- Simulated bulk RNA-seq count matrix
- Simulated regulatory-layer count matrices (ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq, TF)
- Differential-expression metadata (DEG lists) per omics layer
- Ground-truth gene–regulator regulatory relationships (activators and repressors)
- Validation artifacts such as cluster_assessment.csv or more_evaluation_metrics.csv

## How to apply

Install MOSim from Bioconductor (BiocManager::install("MOSim")) and call mosim() seeded with STATegra-derived mouse multi-omics data, configuring the experimental design — e.g. 30M reads, two experimental groups, three biological replicates, five or six time points, 5–15% differentially expressed genes, and 40–60% repressors/significant regulations depending on the scenario. Include the RNA-seq layer plus desired regulatory layers (ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq, TF) and, optionally, predefined temporal co-expression profiles. Extract the simulated count matrices and differential-expression/regulatory metadata from the returned MOSim object. Validate the simulation by filtering lowly expressed genes (CPM > 1), applying a log(x+1) transformation, and running k-means clustering (k chosen by maximizing the Silhouette coefficient) to confirm simulated temporal patterns are recovered; validate regulatory layers by computing Pearson correlations for DEG–regulator pairs and comparing against 20,000 random feature pairs with no simulated regulatory effects.

## Related tools

- **MOSim** (Core simulator: mosim() generates bulk RNA-seq plus ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq and TF regulatory layers with configurable design, DEGs and active regulators) — github.com/ConesaLab/MOSim
- **R** (Runtime environment; MOSim is a Bioconductor R package installed via BiocManager and driven from R scripts)
- **SPARSim** (Count-data simulation functionality underlying sc_mosim single-cell simulation (cite when the sc module is used))

## Examples

```
BiocManager::install("MOSim"); result <- mosim(omics = c("RNA-seq", "miRNA-seq", "TF"), times = 6, groups = 2, reps = 3, depth = 30000000, degenes = 0.05)
```

## Evaluation signals

- K-means clustering of simulated gene profiles with optimal k (e.g. k = 7 by Silhouette maximization) reproduces one cluster per simulated temporal pattern, with an incorrect-assignment rate below 0.5%
- PCA on log(x+1)-transformed, CPM > 1-filtered simulated data aligns with the simulation settings
- Pearson correlations of DEG–regulator pairs in bulk data are clearly separated from the distribution of 20,000 random feature pairs with no simulated regulatory effect
- Downstream GRN inference on the simulated data achieves expected benchmark performance (e.g. MORE: sensitivity 85.5%, F1 62.9%)
- Expected counts of DEGs, fitted models, and significant regulations are reproduced (e.g. 10,593 DEGs; 370,566 potential interactions)

## Limitations

- Single-cell simulation is restricted to scRNA-seq and scATAC-seq only, as these are the only simultaneously performable techniques
- MOSim cannot simulate GRNs with spatial resolution
- Not applicable to omics layers like proteomics and metabolomics, which may influence gene regulation in more complex or uncertain ways
- Benchmark configurations vary across tasks (e.g. six time points/three replicates vs. 20 time points/one replicate), so parameter choices must match the intended validation scenario

## Evidence

- [abstract] The mosim function generates bulk transcriptomics data (RNA-seq) and additional regulatory omics layers (ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq, and transcription factors): "The mosim function generates bulk transcriptomics data (RNA-seq) and additional regulatory omics layers (ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq, and transcription factors)"
- [methods] tested with mouse multi-omics data from the STATegra project: "tested with mouse multi-omics data from the STATegra project [12]"
- [methods] a log transformation (log(x+1)) was applied to the data: [evidence span withheld — anchor status: not_found]
- [methods] k-means clustering [39] was applied to the simulated feature profiles to assess the correct simulation of temporal expression patterns: "k-means clustering [39] was applied to the simulated feature profiles to assess the correct simulation of temporal expression patterns"
- [readme] MOSim is a Bioconductor R package, and we strongly recommend that it is installed from the Bioconductor repository.: "MOSim is a Bioconductor R package, and we strongly recommend that it is installed from the Bioconductor repository."
- [discussion] Currently, single-cell simulation is restricted to scRNA-seq and scATAC-seq: "Currently, single-cell simulation is restricted to scRNA-seq and scATAC-seq, as these are presently the only two commercially available sequencing techniques that can be simultaneously performed on"
