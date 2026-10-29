---
name: differential-expression-feature-extraction
description: Use when a simulated bulk multi-omics dataset (e.g. MOSim RNA-seq + miRNA-seq
  + TF) must yield a defined DEG/regulator feature set with ground-truth regulatory
  signs (activator vs repressor) for downstream validation.
when_to_use_negative:
- Input is real (non-simulated) experimental data without ground-truth regulatory
  signs — this skill's validation relies on MOSim-simulated DE and known activator/repressor
  effects
- Simulating single-cell omics layers other than scRNA-seq, scATAC-seq and TFs — MOSim's
  sc_mosim does not support other single-cell modalities
- The analysis requires spatially resolved GRNs or proteomics/metabolomics regulatory
  layers — MOSim explicitly does not cover these
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: Generates the simulated bulk multi-omics data (mosim function) with RNA-seq
    as the central layer, plus miRNA-seq and TF regulatory layers, providing the DEG
    and active-regulator ground truth that this skill extracts
  repo: github.com/ConesaLab/MOSim
- name: SPARSim
  role: Underlying count simulator relied on by MOSim's single-cell (sc_mosim) simulation
    module
- name: R
  role: Environment in which MOSim is installed via BiocManager and where count extraction,
    log/z-score transformation, correlation testing and heatmap rendering are performed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:d75d3f3523397b08e1bfcdc0c7df1313ce5137c655de8441c1b1529fc43b640f
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Differential-expression feature extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract differentially expressed features (DEGs) and their linked regulators from a simulated multi-omics MOSim object, then transform and validate their group-wise expression profiles against a no-regulation null. Provides ground-truth positive/negative feature sets for benchmarking GRN and multi-omics integration methods.

## When to use

Use when a simulated bulk multi-omics dataset (e.g. MOSim RNA-seq + miRNA-seq + TF) must yield a defined DEG/regulator feature set with ground-truth regulatory signs (activator vs repressor) for downstream validation.

## When NOT to use

- Input is real (non-simulated) experimental data without ground-truth regulatory signs — this skill's validation relies on MOSim-simulated DE and known activator/repressor effects
- Simulating single-cell omics layers other than scRNA-seq, scATAC-seq and TFs — MOSim's sc_mosim does not support other single-cell modalities
- The analysis requires spatially resolved GRNs or proteomics/metabolomics regulatory layers — MOSim explicitly does not cover these

## Inputs

- MOSim simulation object returned by mosim() (bulk RNA-seq plus miRNA-seq, TF, and optionally ATAC-seq/ChIP-seq/Methyl-seq layers)
- Simulated count matrices per omic layer (bulk RNA-seq counts, miRNA-seq counts, TF regulatory profiles)
- Differential-expression and active-regulator metadata with simulated regulatory effects (activator/repressor signs)
- Experimental design parameters: read depth (e.g. 30M), groups, replicates, time points, DE fraction, repressor fraction

## Outputs

- Selected DEG and linked-regulator feature list (e.g. first 100 DEGs) per omic layer
- Per-group mean count matrices over replicates for genes and regulators
- Log(x+1)-transformed, row-wise z-scored mean-expression profiles
- Group-wise DEG/regulator expression heatmap across RNA-seq, miRNA-seq and TF layers
- Pearson correlation values for DEG–regulator pairs versus 20,000 random no-regulation feature pairs

## How to apply

Run MOSim's mosim() with an explicit design (e.g. STATegra seed data, 30M reads, two groups, three biological replicates, six time points, 5% DEGs, 40% repressors) so differential expression is simulated under known conditions. Extract the simulated count matrices and the differential-expression/regulatory metadata from the returned MOSim object, selecting e.g. the first 100 DEGs and their linked miRNA-seq and TF regulators. Compute per-group mean expression across replicates, apply the log(x+1) transformation and row-wise z-score scaling, then render heatmaps ordered to show temporal activator/repressor patterns across layers. Validate by computing Pearson correlations for bulk DEG–regulator pairs (Kendall's Tb for single-cell) and comparing them against 20,000 random feature pairs with no simulated regulatory effects; regulator-DEG consistency confirms the ground truth is usable for benchmarking.

## Related tools

- **MOSim** (Generates the simulated bulk multi-omics data (mosim function) with RNA-seq as the central layer, plus miRNA-seq and TF regulatory layers, providing the DEG and active-regulator ground truth that this skill extracts) — github.com/ConesaLab/MOSim
- **SPARSim** (Underlying count simulator relied on by MOSim's single-cell (sc_mosim) simulation module)
- **R** (Environment in which MOSim is installed via BiocManager and where count extraction, log/z-score transformation, correlation testing and heatmap rendering are performed)

## Examples

```
sim <- mosim(omics = c("RNA-seq", "miRNA-seq", "TF")); degs <- sim[["RNA-seq"]][["DE_omics"]]
```

## Evaluation signals

- Simulated feature profiles recapitulate the specified design: correct number of groups, replicates and time points, with ~5% DEGs and ~40% repressors as set in mosim()
- DEG–regulator Pearson correlations (bulk) are clearly separated from the distribution of 20,000 random no-regulation feature pairs
- Activator and repressor pairs show sign-consistent correlations: positive for activators, negative for repressors
- k-means clustering of simulated profiles (with PCA confirmation) aligns with the simulated temporal expression settings
- Heatmap of row-wise z-scored log(x+1) mean expressions shows distinct, layer-consistent feature profiles across RNA-seq, miRNA-seq and TF layers

## Limitations

- Single-cell simulation via sc_mosim is restricted to scRNA-seq and scATAC-seq (plus TFs), so single-cell DEG–regulator validation uses Kendall's Tb rather than the full multi-layer bulk panel
- MOSim cannot simulate GRNs with spatial resolution, and is not applicable to proteomics or metabolomics regulatory layers
- Ground truth for DEG–regulator relationships must be checked against random-pair nulls (20,000 pairs), since simulated regulatory effects are only expected to hold for the specified DEG–regulator links

## Evidence

- [methods] log transformation step: [evidence span withheld — anchor status: not_found]
