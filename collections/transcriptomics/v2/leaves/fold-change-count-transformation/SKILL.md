---
name: fold-change-count-transformation
description: Use when simulating bulk multi-omics count data with MOSim and you need
  to impose temporal DEG fold-change profiles on negative-binomial counts and couple
  regulatory-layer features to gene expression patterns.
when_to_use_negative:
- Single-cell simulation — sc_mosim relies on SPARSim functionality and only supports
  scRNA-seq with scATAC-seq and TF regulatory layers, not the full bulk regulatory
  omics set
- You need regulatory omics layers beyond MOSim's supported set, e.g. proteomics or
  metabolomics, which the tool explicitly does not cover
- You only need to transform an existing observed count matrix — this skill generates
  synthetic counts, not a normalization of real data
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: R/Bioconductor package providing the mosim function that generates bulk RNA-seq
    plus regulatory omics layers (ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq, transcription
    factors)
  repo: github.com/ConesaLab/MOSim
- name: SPARSim
  role: Parameterization approach for negative-binomial count simulation of scRNA-seq-style
    count data relied on by MOSim
- name: R
  role: Implementation language; stats package random functions draw the negative-binomial
    counts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:0a8d2827758a296ba5050b84f08630da95ff0111933fb8b31bd3e72a61dd77d4
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
  - dplyr
  - purrr
  - stats
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fold-change-count-transformation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MOSim bulk simulation procedure in which negative-binomial RNA-seq counts are modulated by profile-specific fold changes (continuous or transitory induction/repression) and regulator (TF, ATAC-seq, ChIP-seq, miRNA-seq, Methyl-seq) profiles are derived as same-pattern (activator) or mirrored (repressor) relative to their target genes.

## When to use

Use when simulating bulk multi-omics count data with MOSim and you need to impose temporal DEG fold-change profiles on negative-binomial counts and couple regulatory-layer features to gene expression patterns.

## When NOT to use

- Single-cell simulation — sc_mosim relies on SPARSim functionality and only supports scRNA-seq with scATAC-seq and TF regulatory layers, not the full bulk regulatory omics set
- You need regulatory omics layers beyond MOSim's supported set, e.g. proteomics or metabolomics, which the tool explicitly does not cover
- You only need to transform an existing observed count matrix — this skill generates synthetic counts, not a normalization of real data

## Inputs

- Seed bulk RNA-seq count sample (used to estimate per-gene mean and dispersion parameters)
- Experimental design specification (conditions, time points, biological replicates, DEG subset size)
- Regulatory network/linkage definition assigning activator or repressor roles to TF, ATAC-seq, ChIP-seq, miRNA-seq, and Methyl-seq features

## Outputs

- Simulated bulk RNA-seq negative-binomial count matrix with profile-specific fold changes applied
- Regulator profile table encoding same-pattern (activator) or opposite-pattern (repressor) relationships to target genes

## How to apply

Load a seed bulk RNA-seq sample and estimate per-gene mean and dispersion parameters for negative-binomial (SPARSim-style) count generation. Randomly select a subset of features as DEGs and assign each a temporal label: continuous induction/repression (scaled fold change), transitory induction/repression, or flat profile, according to the supplied design parameters. Draw negative-binomial counts for every gene, time point, and replicate (R stats random functions), then apply the profile-specific fold changes to the base counts for flat and two-group designs. Derive regulator profiles by setting each activator's pattern identical to its target genes' pattern and each repressor's pattern to the mirrored/opposite pattern, then write the bulk count matrix and regulator profile table as outputs.

## Related tools

- **MOSim** (R/Bioconductor package providing the mosim function that generates bulk RNA-seq plus regulatory omics layers (ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq, transcription factors)) — github.com/ConesaLab/MOSim
- **SPARSim** (Parameterization approach for negative-binomial count simulation of scRNA-seq-style count data relied on by MOSim)
- **R** (Implementation language; stats package random functions draw the negative-binomial counts)

## Examples

```
BiocManager::install("MOSim")
```

## Evaluation signals

- Simulated counts are consistent with the seed sample's estimated per-gene mean and dispersion parameters
- Emitted activator regulator profiles correctly correspond (same temporal pattern) to their assigned target genes, and repressor profiles to the mirrored pattern (e.g. continuous induction vs continuous repression)
- Pearson correlation between gene-regulator pairs is computed for bulk data and compared against 20,000 random feature pairs with no simulated regulatory effects
- k-means clustering of simulated feature profiles recovers the assigned temporal expression patterns; PCA can confirm clustering aligns with the simulation settings

## Limitations

- The paper reports no quantitative evaluation of this activator/repressor coupling mechanism in isolation; acceptance is qualitative
- MOSim cannot simulate GRNs with spatial resolution and is not applicable to omics layers like proteomics and metabolomics
- MOSim does not report interaction among features or offer full customization compared to the gap identified for prior bulk simulators; single-cell simulation is restricted to scRNA-seq and scATAC-seq only

## Evidence
