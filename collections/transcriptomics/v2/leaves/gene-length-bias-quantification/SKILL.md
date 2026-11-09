---
name: gene-length-bias-quantification
description: Use when validating that a simulated non-UMI scRNA-seq dataset recapitulates
  gene length bias, or when contrasting length-biased full-length protocols against
  UMI-based protocols that lack it.
when_to_use_negative:
- The analysis targets UMI-based data where gene length bias is absent by design (e.g.
  the Zeisel cortex UMI dataset)
- Only true transcript counts are of interest and library-preparation/sequencing noise
  should not be simulated
- The task requires simulating batch effects rather than length bias — use DivideBatches
  instead
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: Provides True2ObservedCounts, gene_len_pool, and the Th17_data.RData reference
    to simulate non-UMI/UMI observed counts with gene length bias
  repo: https://github.com/YosefLab/SymSim
- name: ggplot2
  role: Plots gene length versus length-normalized expression for simulated and real
    data side by side
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:89b74a914e8f430fe4b80764c72590658ecc55de55c45cb52bd7a627ec42d7f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - SymSim
  - ggplot2
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Gene-length-bias quantification in scRNA-seq simulation

## Summary

Reproduces the gene-length dependence of expression seen in non-UMI (full-length) scRNA-seq by running SymSim's True2ObservedCounts with gene length bias (lenslope, nbins) on both non-UMI and UMI protocols, then comparing length-normalized expression against the Gaublomme Th17 reference.

## When to use

Use when validating that a simulated non-UMI scRNA-seq dataset recapitulates gene length bias, or when contrasting length-biased full-length protocols against UMI-based protocols that lack it.

## When NOT to use

- The analysis targets UMI-based data where gene length bias is absent by design (e.g. the Zeisel cortex UMI dataset)
- Only true transcript counts are of interest and library-preparation/sequencing noise should not be simulated
- The task requires simulating batch effects rather than length bias — use DivideBatches instead

## Inputs

- True counts matrix and cell metadata from SimulateTrueCounts (genes x cells)
- gene_len_pool of human transcript lengths (shipped with SymSim)
- Gaublomme Th17 reference counts (ss2_130cells_counts from Th17_data.RData in SymSim extdata)

## Outputs

- Observed non-UMI count matrix with gene length bias (same gene x cell dimensions as input true counts)
- Observed UMI count matrix on the same true counts
- ggplot2 side-by-side plot of gene length versus length-normalized expression (simulated vs real, Fig. 4b-style comparison)

## How to apply

Load the true counts matrix and cell metadata from SimulateTrueCounts (e.g. output of task_002) and assign each gene a length sampled from the gene_len_pool of human transcript lengths shipped with SymSim. Run True2ObservedCounts twice on the same true counts: once with protocol='nonUMI', alpha_mean=0.1, alpha_sd=0.05, depth_mean=1e5, depth_sd=3e3, with gene length bias controlled via lenslope and nbins to model length-biased PCR amplification and fragmentation; and once with protocol='UMI', alpha_mean=0.05, alpha_sd=0.02, depth_mean=5e4, depth_sd=3e3. Load the Gaublomme Th17 reference (ss2_130cells_counts from Th17_data.RData) and compute length-normalized expression per gene for both the simulated non-UMI counts and the reference counts. Plot gene length versus length-normalized expression side by side with ggplot2, reproducing the Fig. 4b comparison: the simulated non-UMI pattern should match the Th17 reference, while UMI-based simulated data show no length bias.

## Related tools

- **SymSim** (Provides True2ObservedCounts, gene_len_pool, and the Th17_data.RData reference to simulate non-UMI/UMI observed counts with gene length bias) — https://github.com/YosefLab/SymSim
- **ggplot2** (Plots gene length versus length-normalized expression for simulated and real data side by side)

## Examples

```
devtools::install_github('YosefLab/SymSim'); obs <- True2ObservedCounts(true_counts, meta_cell, protocol='nonUMI', alpha_mean=0.1, alpha_sd=0.05, gene_len=gene_len_pool, depth_mean=1e5, depth_sd=3e3, lenslope=0.02, nbins=20)
```

## Evaluation signals

- The gene length vs length-normalized expression relationship in simulated non-UMI data visually matches the Th17 (ss2_130cells_counts) reference pattern from Fig. 4b
- No gene length dependence appears in the UMI-based simulated data run on the same true counts
- Observed count matrices from True2ObservedCounts have the same gene x cell dimensions as the input true counts
- Gene lengths assigned to simulated genes are drawn from the gene_len_pool human transcript length distribution

## Limitations

- The SymSim pipeline omits several library-preparation steps, including reverse transcription and library cleaning up
- Details of the fragmentation distributions are deferred to Supplementary Note 3 and are not included in the main text
- The non-UMI Th17 reference contains only 130 IL17-expressing T helper cells, limiting the resolution of the length-bias comparison

## Evidence

- [methods] True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts: "True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts"
- [methods] Each genes needs to be assigned a gene length. We sample lengths from human transcript lengths.: "Each genes needs to be assigned a gene length. We sample lengths from human transcript lengths."
- [methods] load(system.file("extdata", "Th17_data.RData", package = "SymSim"))
counts <- ss2_130cells_counts: "load(system.file("extdata", "Th17_data.RData", package = "SymSim"))"
- [methods] devtools::install_github('YosefLab/SymSim'): "devtools::install_github('YosefLab/SymSim')"
- [intro] SymSim recapitulates properties of the data (e.g., high abundance of zeros or increased noise in non-UMI protocols) without the need to explicitly force them as factors in a distributional model: [evidence span withheld — anchor status: not_found]
