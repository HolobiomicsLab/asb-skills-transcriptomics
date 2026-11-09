---
name: technical-variation-modeling
description: Use when you have a simulated or real true-count matrix (genes x cells)
  and need to generate realistic observed scRNA-seq read/UMI counts reflecting capture
  efficiency, PCR amplification bias, gene length bias, and sequencing depth.
when_to_use_negative:
- You need to simulate intrinsic transcriptional noise or extrinsic cell-state variability
  rather than technical/sequencing effects — use SimulateTrueCounts instead
- The input is already observed read/UMI data; True2ObservedCounts expects true transcript
  counts as input
- You need batch effects — use DivideBatches on the output of True2ObservedCounts
  instead
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SymSim
  role: Provides True2ObservedCounts, gene_len_pool, and Th17_data.RData reference
    data for simulating library preparation, sequencing, and gene length bias
  repo: https://github.com/YosefLab/SymSim
- name: ggplot2
  role: Plots gene length versus length-normalized expression for simulated and real
    (Th17) data side by side
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:bb17f86c4a2d09ac93e0fbbb5b0ac58fa3972dbd07a285c62316daa816d9d47d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - SymSim
  - ggplot2
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# technical-variation-modeling

## Summary

Simulates library preparation and sequencing (capture efficiency, amplification bias, fragmentation, sequencing depth) on top of true transcript counts using SymSim's True2ObservedCounts, including gene length bias specific to non-UMI protocols.

## When to use

Use when you have a simulated or real true-count matrix (genes x cells) and need to generate realistic observed scRNA-seq read/UMI counts reflecting capture efficiency, PCR amplification bias, gene length bias, and sequencing depth.

## When NOT to use

- You need to simulate intrinsic transcriptional noise or extrinsic cell-state variability rather than technical/sequencing effects — use SimulateTrueCounts instead
- The input is already observed read/UMI data; True2ObservedCounts expects true transcript counts as input
- You need batch effects — use DivideBatches on the output of True2ObservedCounts instead

## Inputs

- True transcript counts matrix (genes x cells), e.g. from SymSim SimulateTrueCounts or task_002 output
- Cell metadata from the true counts simulation
- gene_len_pool of human transcript lengths (data object shipped with SymSim)
- Reference experimental non-UMI counts for validation, e.g. ss2_130cells_counts from Th17_data.RData (system.file('extdata', 'Th17_data.RData', package='SymSim'))

## Outputs

- Observed non-UMI read counts matrix (same gene x cell dimensions as input true counts)
- Observed UMI counts matrix (same gene x cell dimensions as input true counts)
- Plots of gene length versus length-normalized expression for simulated non-UMI and real Th17 data (Fig. 4b-style comparison)

## How to apply

Start from a true counts matrix (e.g., from SimulateTrueCounts) and assign each gene a length sampled from the gene_len_pool of human transcript lengths shipped with SymSim. Call True2ObservedCounts twice on the same true counts: once with protocol='nonUMI' (alpha_mean=0.1, alpha_sd=0.05, depth_mean=1e5, depth_sd=3e3, plus lenslope and nbins controlling gene length bias) to simulate capture efficiency, length-biased PCR amplification, fragmentation, and depth; once with protocol='UMI' (alpha_mean=0.05, alpha_sd=0.02, depth_mean=5e4, depth_sd=3e3). The rationale is that non-UMI protocols introduce gene length dependence on expression, whereas UMI-based data do not, as shown in the paper's Fig. 4b comparison against the Gaublomme Th17 dataset. Validate by checking that observed count matrices retain the gene x cell dimensions of the input true counts and by plotting gene length versus length-normalized expression for simulated non-UMI and real counts (ss2_130cells_counts from Th17_data.RData) with ggplot2.

## Related tools

- **SymSim** (Provides True2ObservedCounts, gene_len_pool, and Th17_data.RData reference data for simulating library preparation, sequencing, and gene length bias) — https://github.com/YosefLab/SymSim
- **ggplot2** (Plots gene length versus length-normalized expression for simulated and real (Th17) data side by side)

## Examples

```
devtools::install_github('YosefLab/SymSim'); obs <- True2ObservedCounts(true_counts, meta_cell, protocol="nonUMI", alpha_mean=0.1, alpha_sd=0.05, gene_len=gene_len, depth_mean=1e5, depth_sd=3e3, lenslope=1, nbins=20)
```

## Evaluation signals

- Observed count matrices (non-UMI and UMI) must have the same gene x cell dimensions as the input true counts
- Simulated non-UMI gene length vs length-normalized expression relationship should visually match the Th17 (ss2_130cells_counts) reference pattern
- No gene length bias should appear in the UMI-based simulated counts
- Positive length bias should be removed after length normalization of the non-UMI data
- Depth distributions should match specified depth_mean/depth_sd (e.g. 1e5 +/- 3e3 for nonUMI, 5e4 +/- 3e3 for UMI)

## Limitations

- The simulation pipeline omits several steps, including reverse transcription and library cleaning up
- Details of the fragmentation distributions are deferred to Supplementary Note 3 and not included in the main text
- scVI is only applicable to large datasets, so non-UMI data in the paper were imputed with MAGIC instead

## Evidence

- [methods] True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts: "True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts"
- [methods] Each genes needs to be assigned a gene length. We sample lengths from human transcript lengths.: "Each genes needs to be assigned a gene length. We sample lengths from human transcript lengths."
- [discussion] The lengths of the simulated transcripts are obtained from the human reference genome: "The lengths of the simulated transcripts are obtained from the human reference genome"
- [methods] devtools::install_github('YosefLab/SymSim'): "devtools::install_github('YosefLab/SymSim')"
- [methods] load(system.file("extdata", "Th17_data.RData", package = "SymSim"))
counts <- ss2_130cells_counts: "load(system.file("extdata", "Th17_data.RData", package = "SymSim"))
counts <- ss2_130cells_counts"
