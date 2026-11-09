---
name: power-law-distribution-fitting
description: Use when a large-scale biological grouping result (e.g., ortholog group
  sizes from OrthoFinder over hundreds of species) needs its size distribution characterized
  and validated against a power-law expectation.
when_to_use_negative:
- When the input is a small or single-species orthology result where the group-size
  tail is too sparsely sampled to support a meaningful power-law fit
- When the grouping result itself cannot be reproduced at the reported scale (666,067
  groups from 12,828,537 proteins) — fitting a distribution on a failed binning step
  would not validate the pipeline
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: OrthoFinder
  role: performs the all-vs-all orthology inference whose orthogroup sizes are then
    fit to a power-law distribution
- name: KEGGREST
  role: downloads all protein-coding sequences for the 687 eukaryotic KEGG organisms
    (one FASTA per organism) that form the input proteomes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:146825da78f7df5afe75db2c67fc45102f33ea4854990dc40ea6ad3e9a219b2e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGGREST
  - OrthoFinder
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# power-law-distribution-fitting

## Summary

Fits a power-law model to the size distribution of ortholog groups produced by large-scale OrthoFinder binning, using the fitted parameters to validate that the grouping (666,067 groups from 12,828,537 proteins across 687 eukaryotic species) reproduces the reported EcoOmicsDB structure.

## When to use

Use when a large-scale biological grouping result (e.g., ortholog group sizes from OrthoFinder over hundreds of species) needs its size distribution characterized and validated against a power-law expectation.

## When NOT to use

- When the input is a small or single-species orthology result where the group-size tail is too sparsely sampled to support a meaningful power-law fit
- When the grouping result itself cannot be reproduced at the reported scale (666,067 groups from 12,828,537 proteins) — fitting a distribution on a failed binning step would not validate the pipeline

## Inputs

- OrthoFinder orthogroups results (protein-to-ortholog-group assignments) for 12,828,537 proteins from 687 species
- Concatenated KEGG protein FASTA files (one FASTA per organism, 687 organisms total)

## Outputs

- Fitted power-law parameters for the ortholog group-size distribution
- Per-species summary table reporting protein-coding gene counts and ortholog-group memberships for each of the 687 species (matching Table 1)
- Total ortholog-group count (666,067) and group-size tallies

## How to apply

First obtain ortholog groups by downloading all protein-coding sequences for the 687 eukaryotic organisms in KEGG (n = 12,828,537 proteins, one FASTA per organism via KEGGREST), concatenating the 687 proteomes, and running OrthoFinder all-vs-all orthology inference with 56 threads (-t 56) and 25 threads for the sequence search stage (-a 25). Parse the OrthoFinder orthogroups results to bin every protein into an ortholog group and tally the total (target: 666,067 groups). Compute the per-group size (number of member proteins), then fit the resulting group-size distribution to a power-law model and record the fitted parameters. Validation requires that the fitted distribution 'largely follows a power law distribution' and that the pipeline scale (687 organisms, 12,828,537 genes, 666,067 ortholog groups) and the per-species summary table (protein-coding genes and ortholog-group memberships per species) match the published Table 1.

## Related tools

- **OrthoFinder** (performs the all-vs-all orthology inference whose orthogroup sizes are then fit to a power-law distribution)
- **KEGGREST** (downloads all protein-coding sequences for the 687 eukaryotic KEGG organisms (one FASTA per organism) that form the input proteomes)

## Evaluation signals

- Pipeline reproduces the reported scale: 687 organisms, 12,828,537 protein-coding genes, and 666,067 ortholog groups
- The ortholog group-size distribution fits a power-law model, with fitted parameters recorded alongside the summary table
- The per-species summary table reports both protein-coding gene counts and ortholog-group memberships for all 687 species, matching the structure of Table 1
- Every protein from the concatenated proteomes is assigned to exactly one ortholog group when parsing the OrthoFinder results

## Limitations

- The published result states the size distribution 'largely follows' a power law, implying deviations from a perfect power-law fit rather than an exact match
- The fit is only as valid as the upstream binning; errors in the KEGG download or OrthoFinder run (e.g., thread configuration -t 56 / -a 25) propagate into the distribution
- The source card does not specify the fitting library, xmin choice, or goodness-of-fit test used, so the exact fitted parameters cannot be independently pinned down

## Evidence

- [results] OrthoFinder power-law result: [evidence span withheld — anchor status: not_found]
- [discussion] KEGG proteome download: "All protein-coding genes (n = 12,828,537) from 687 organisms covering all major phyla of eukaryotes were downloaded from KEGG"
- [abstract] EcoOmicsDB scope: "All modules are integrated with EcoOmicsDB, an ortholog database that enables comprehensive analysis for species without a reference transcriptome."
