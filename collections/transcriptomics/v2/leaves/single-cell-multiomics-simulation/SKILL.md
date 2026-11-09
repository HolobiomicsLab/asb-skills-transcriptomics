---
name: single-cell-multiomics-simulation
description: Use when benchmarking or validating single-cell multi-omics integration,
  clustering, or GRN-inference methods requires synthetic scRNA-seq + scATAC-seq data
  with known differential expression, co-expression patterns, and regulatory ground
  truth.
when_to_use_negative:
- The target omics layers are proteomics or metabolomics, or spatially resolved data
  — MOSim's single-cell simulation is restricted to scRNA-seq and scATAC-seq and cannot
  simulate GRNs with spatial resolution
- Only a feature/abundance table is needed without regulatory ground truth — simpler
  scRNA-seq-only simulators avoid the extra regulatory layers
- Bulk (not single-cell) multi-omics simulation is required — use MOSim's mosim function
  instead, which additionally supports miRNA-seq, ChIP-seq, and Methyl-seq layers
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: MOSim
  role: R package providing sc_mosim to simulate single-cell scRNA-seq counts with
    scATAC-seq and TF regulatory layers from a seed dataset
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Execution environment; MOSim is a Bioconductor package installed via BiocManager
    or from GitHub with devtools
- name: Seurat
  role: SeuratData R package supplies the pbmc.multiome 10x Genomics seed dataset
    used to parameterize the simulation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:5e5bcea22c1a830143155747fb02c05f1a72dca27774ecbdc85febb01fa9e58e
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
  - Seurat
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-multiomics-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate single-cell multi-omics ground-truth data (scRNA-seq with scATAC-seq and transcription factor regulatory layers) using MOSim's sc_mosim function, with customizable experimental designs (cell types, groups, replicates, DE proportions) for benchmarking integration and GRN inference methods.

## When to use

Use when benchmarking or validating single-cell multi-omics integration, clustering, or GRN-inference methods requires synthetic scRNA-seq + scATAC-seq data with known differential expression, co-expression patterns, and regulatory ground truth.

## When NOT to use

- The target omics layers are proteomics or metabolomics, or spatially resolved data — MOSim's single-cell simulation is restricted to scRNA-seq and scATAC-seq and cannot simulate GRNs with spatial resolution
- Only a feature/abundance table is needed without regulatory ground truth — simpler scRNA-seq-only simulators avoid the extra regulatory layers
- Bulk (not single-cell) multi-omics simulation is required — use MOSim's mosim function instead, which additionally supports miRNA-seq, ChIP-seq, and Methyl-seq layers

## Inputs

- Seed single-cell dataset (e.g. pbmc.multiome 10x Genomics dataset from the SeuratData R package)
- Experimental design parameters: number of cell types, groups, biological replicates, DEG percentages (up/down-regulated), activator/repressor percentages

## Outputs

- Simulated scRNA-seq count matrix per cell type/group/replicate
- Simulated scATAC-seq regulatory layer
- Simulated transcription factor (TF) regulatory layer
- Ground-truth labels of differentially abundant features and active regulatory relationships

## How to apply

Load a seed single-cell dataset (e.g. the pbmc.multiome 10x Genomics dataset from the SeuratData R package) to parameterize the simulation. Call sc_mosim from the MOSim R package specifying the cell types (e.g. six), experimental groups (e.g. two), biological replicates (e.g. three), and differential expression percentages (e.g. 30% up-regulated and 20% down-regulated DEGs) plus activator/repressor percentages, to generate scRNA-seq counts with scATAC-seq and TF regulatory layers. sc_mosim relies on SPARSim for scRNA-seq count simulation, so cite SPARSim when using this module. Optionally aggregate the simulated count matrix to average count per cell type and apply a log(x+1) transformation to assess whether simulated profiles recapitulate the imposed co-expression/DE structure (e.g. via Spearman distance, k-medoids clustering, and PCA). Validate by comparing cluster assignments against the true simulated cell-type/DE-group labels.

## Related tools

- **MOSim** (R package providing sc_mosim to simulate single-cell scRNA-seq counts with scATAC-seq and TF regulatory layers from a seed dataset) — github.com/ConesaLab/MOSim
- **R** (Execution environment; MOSim is a Bioconductor package installed via BiocManager or from GitHub with devtools)
- **Seurat** (SeuratData R package supplies the pbmc.multiome 10x Genomics seed dataset used to parameterize the simulation)

## Examples

```
BiocManager::install("MOSim"); sim <- sc_mosim(omics = list("scRNA-seq", "scATAC-seq", "TF"), cell_types = pbmc_types, groups = 2, replicates = 3, nDEGs = 0.3)
```

## Evaluation signals

- K-medoids clustering (Spearman correlation distance, k=10) of across-cell-type average profiles assigns ~96.7% of simulated features to the expected cluster, i.e. only ~3.3% incorrect assignments
- PCA on the log(x+1)-transformed profiles confirms clustering aligns with the simulation settings (cell-type/DE-group structure)
- Simulated differential features match the specified DEG proportions (e.g. 30% up-regulated, 20% down-regulated) and group/replicate design
- Active regulatory relationships (scATAC-seq, TFs) show higher correlation between gene and regulator than 20000 random feature pairs with no simulated regulatory effects (Kendall's Tb for single-cell data)
- Optimal cluster number (chosen by maximizing Silhouette coefficient and minimizing intra-cluster variability) matches the simulated co-expression pattern count

## Limitations

- Single-cell simulation is restricted to scRNA-seq and scATAC-seq only, since these are the only two techniques simultaneously performable on the same cell
- MOSim cannot simulate GRNs with spatial resolution (spatial multi-omics extension deferred to future work)
- Not applicable to omics layers like proteomics and metabolomics, which may influence gene regulation
- sc_mosim strongly relies on SPARSim functionality, which should also be cited when used

## Evidence

- [methods] Log transformation step: [evidence span withheld — anchor status: not_found]
- [readme] SPARSim dependency and citation requirement: [evidence span withheld — anchor status: external_doc]
