---
name: pcr-amplification-and-fragmentation-simulation
description: Use when you have a true counts matrix from SimulateTrueCounts and need
  observed counts that reflect a specific scRNA-seq protocol (non-UMI vs UMI), or
  when testing whether simulated data reproduce protocol-specific artifacts like gene
  length bias.
when_to_use_negative:
- You only need true (latent) transcript counts without technical noise — use SimulateTrueCounts
  alone
- You need to model batch effects — use DivideBatches on the output of True2ObservedCounts
  instead, as the library-preparation pipeline itself omits reverse transcription
  and library cleanup steps
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0203
tools:
- name: SymSim
  role: Provides True2ObservedCounts, gene_len_pool, and the Th17_data.RData reference
    used to simulate library preparation with gene length bias
  repo: https://github.com/YosefLab/SymSim
- name: ggplot2
  role: Plots gene length versus length-normalized expression for simulated vs real
    (Th17) data side by side, reproducing Fig. 4b
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:c55ed73e6cddc2b8f644fb368a128daa63da82207d945bccd327bfa2b0ee35f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - SymSim
  - ggplot2
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# pcr-amplification-and-fragmentation-simulation

## Summary

Use SymSim's True2ObservedCounts function to convert simulated true transcript counts into observed non-UMI read counts or UMI counts, explicitly modeling capture efficiency, PCR amplification (with gene length bias via lenslope and nbins), fragmentation, and sequencing depth. This reproduces protocol-specific technical artifacts such as the gene length bias seen in full-length scRNA-seq data.

## When to use

Use when you have a true counts matrix from SimulateTrueCounts and need observed counts that reflect a specific scRNA-seq protocol (non-UMI vs UMI), or when testing whether simulated data reproduce protocol-specific artifacts like gene length bias.

## When NOT to use

- You only need true (latent) transcript counts without technical noise — use SimulateTrueCounts alone
- You need to model batch effects — use DivideBatches on the output of True2ObservedCounts instead, as the library-preparation pipeline itself omits reverse transcription and library cleanup steps

## Inputs

- True transcript counts matrix and cell metadata from SimulateTrueCounts
- Gene lengths sampled from gene_len_pool (human transcript lengths shipped with SymSim)
- Protocol parameters (protocol='nonUMI' or 'UMI', alpha_mean, alpha_sd, depth_mean, depth_sd, lenslope, nbins)
- Real reference count matrix for comparison, e.g. ss2_130cells_counts from Th17_data.RData

## Outputs

- Observed read count matrix (non-UMI) with same gene x cell dimensions as the input true counts
- Observed UMI count matrix with same gene x cell dimensions as the input true counts
- Gene length vs length-normalized expression comparison plots (e.g. ggplot2, reproducing Fig. 4b)

## How to apply

Load the true counts matrix and cell metadata produced by SimulateTrueCounts, then assign each gene a length sampled from gene_len_pool (human transcript lengths shipped with SymSim). Run True2ObservedCounts with protocol='nonUMI', alpha_mean=0.1, alpha_sd=0.05, depth_mean=1e5, depth_sd=3e3, and gene length bias controlled via lenslope and nbins, to simulate capture efficiency, length-biased PCR amplification, fragmentation, and sequencing depth. For a UMI-protocol comparison, rerun True2ObservedCounts on the same true counts with protocol='UMI', alpha_mean=0.05, alpha_sd=0.02, depth_mean=5e4, depth_sd=3e3. Validate by checking the observed matrices retain the gene x cell dimensions of the true counts, and by comparing simulated non-UMI gene length vs length-normalized expression against a real non-UMI reference such as the Gaublomme Th17 dataset (ss2_130cells_counts from Th17_data.RData).

## Related tools

- **SymSim** (Provides True2ObservedCounts, gene_len_pool, and the Th17_data.RData reference used to simulate library preparation with gene length bias) — https://github.com/YosefLab/SymSim
- **ggplot2** (Plots gene length versus length-normalized expression for simulated vs real (Th17) data side by side, reproducing Fig. 4b)

## Examples

```
observed_nonUMI <- True2ObservedCounts(true_counts, meta_cell, protocol="nonUMI", alpha_mean=0.1, alpha_sd=0.05, gene_len=gene_len, depth_mean=1e5, depth_sd=3e3, lenslope=0.02, nbins=20)
```

## Evaluation signals

- Observed count matrices have the same gene x cell dimensions as the input true counts
- Simulated non-UMI data show gene-expression dependence on gene length similar to the experimental Gaublomme Th17 reference (positive length bias removed after length normalization)
- No gene length bias appears in the UMI-based simulated data (contrast condition)
- Length-normalized expression vs gene length pattern for simulated non-UMI counts visually matches the Th17 ss2_130cells_counts reference pattern when plotted side by side

## Limitations

- The simulated pipeline omits several steps, including reverse transcription and library cleaning up
- Details of calculating the fragmentation distributions are only in Supplementary Note 3 and not in the main text
- SymSim is designed for single-cell RNA-Seq simulation with interpretable process-level parameters; it does not capture every physical step of library preparation

## Evidence

- [methods] True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts: "True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts"
- [methods] Each genes needs to be assigned a gene length. We sample lengths from human transcript lengths.: "Each genes needs to be assigned a gene length. We sample lengths from human transcript lengths."
- [methods] load(system.file("extdata", "Th17_data.RData", package = "SymSim"))
counts <- ss2_130cells_counts: "load(system.file("extdata", "Th17_data.RData", package = "SymSim"))"
- [discussion] this pipeline omits several steps, including reverse transcription, and library cleaning up.: "this pipeline omits several steps, including reverse transcription, and library cleaning up."
- [readme] devtools::install_github('YosefLab/SymSim'): "devtools::install_github("YosefLab/SymSim")"
