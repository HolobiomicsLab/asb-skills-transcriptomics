---
name: ortholog-group-clustering
description: Use when building or extending a protein ortholog database for non-model
  eukaryotic species (e.g., species lacking a reference transcriptome) so reads can
  be quantified against ortholog groups instead of transcript IDs.
when_to_use_negative:
- The species already has a well-annotated reference genome/transcriptome and a standard
  gene-level quantification pipeline (e.g., Kallisto against GRCz11) is sufficient
  — KO/ortholog coverage is limited (only 76.16% of human and 61.40% of zebrafish
  protein-coding genes carry KOs), so ortholog-based analysis is mainly for non-model
  species.
- The input is already a count table or gene list keyed to known IDs — use the downstream
  statistical/enrichment modules instead of rebuilding ortholog groups.
- A taxonomically narrow sub-group database would be a mismatch for the sample (e.g.,
  mapping protist reads against the 'fishes' database), which risks missing key functional
  groups.
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0622
tools:
- name: OrthoFinder
  role: Performs the all-vs-all orthology inference that bins the 12,828,537 proteins
    into 666,067 ortholog groups (run with -t 56 and -a 25).
- name: KEGGREST
  role: Downloads all protein-coding sequences for the 687 eukaryotic KEGG organisms,
    saving one FASTA per organism.
- name: KEGG
  role: Source of the protein-coding gene sequences and organism set for the ortholog
    database build.
- name: EcoOmicsDB
  role: The resulting ortholog database (29 taxonomic sub-groups) that integrates
    ExpressAnalyst modules for species without a reference transcriptome.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:5bce0e02601a3252955b4eb092fc3b664e38b87c15e13e107dcda3a7e26762a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EcoOmicsDB
  - KEGG
  - KEGGREST
  - OrthoFinder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# ortholog-group-clustering

## Summary

Binning protein-coding sequences from hundreds of eukaryotic species into ortholog groups with OrthoFinder, as used to construct the EcoOmicsDB ortholog database (666,067 groups from 12,828,537 KEGG proteins across 687 species) that underpins KO-level RNA-seq analysis of non-model species.

## When to use

Use when building or extending a protein ortholog database for non-model eukaryotic species (e.g., species lacking a reference transcriptome) so reads can be quantified against ortholog groups instead of transcript IDs.

## When NOT to use

- The species already has a well-annotated reference genome/transcriptome and a standard gene-level quantification pipeline (e.g., Kallisto against GRCz11) is sufficient — KO/ortholog coverage is limited (only 76.16% of human and 61.40% of zebrafish protein-coding genes carry KOs), so ortholog-based analysis is mainly for non-model species.
- The input is already a count table or gene list keyed to known IDs — use the downstream statistical/enrichment modules instead of rebuilding ortholog groups.
- A taxonomically narrow sub-group database would be a mismatch for the sample (e.g., mapping protist reads against the 'fishes' database), which risks missing key functional groups.

## Inputs

- Protein-coding FASTA sequences per organism downloaded from KEGG (687 eukaryotic proteomes, one FASTA per organism, n = 12,828,537 total proteins)
- NCBI taxonomy system used to organize results into 29 taxonomic sub-group databases

## Outputs

- OrthoFinder orthogroups results binning every protein into an ortholog group (target: 666,067 groups)
- Per-species summary table with protein-coding gene counts and ortholog-group memberships for each of the 687 species (Table 1 structure)
- Power-law fit of the ortholog group-size distribution with fitted parameters
- EcoOmicsDB ortholog database split into 29 taxonomic sub-group databases

## How to apply

Download all protein-coding sequences for the target eukaryotic organisms from KEGG using KEGGREST, saving one FASTA per organism (the EcoOmicsDB build covered 687 organisms, n = 12,828,537 proteins). Concatenate the 687 proteomes into a single input directory and run OrthoFinder all-vs-all orthology inference with 56 threads for the main run (-t 56) and 25 threads for the sequence search stage (-a 25). Parse the OrthoFinder orthogroups results to bin every protein into an ortholog group and tally the total number of groups, targeting the published scale of 666,067 groups. Then build a per-species summary table reporting protein-coding gene counts and ortholog-group memberships for each species (matching the structure of Table 1), and fit the ortholog group-size distribution to a power-law model, recording the fitted parameters. The resulting ortholog database (EcoOmicsDB) is organized into 29 NCBI-taxonomy-based taxonomic sub-group databases so users can pick the sub-group closest to their study species.

## Related tools

- **OrthoFinder** (Performs the all-vs-all orthology inference that bins the 12,828,537 proteins into 666,067 ortholog groups (run with -t 56 and -a 25).)
- **KEGGREST** (Downloads all protein-coding sequences for the 687 eukaryotic KEGG organisms, saving one FASTA per organism.)
- **KEGG** (Source of the protein-coding gene sequences and organism set for the ortholog database build.)
- **EcoOmicsDB** (The resulting ortholog database (29 taxonomic sub-groups) that integrates ExpressAnalyst modules for species without a reference transcriptome.)

## Examples

```
orthofinder -f concatenated_proteomes/ -t 56 -a 25
```

## Evaluation signals

- Pipeline output reproduces the reported scale: 687 organisms, 12,828,537 protein-coding genes, and 666,067 ortholog groups.
- The ortholog group-size distribution follows a power-law distribution, consistent with the published Table 1 and its fitted parameters.
- The per-species summary table contains one row per species with protein-coding gene counts and ortholog-group memberships matching the structure of Table 1.
- Every one of the 12,828,537 downloaded proteins is binned into exactly one ortholog group (no unassigned proteins remain after parsing the orthogroups results).

## Limitations

- The group-size distribution only 'largely' follows a power law, so the fit should be reported as approximate, not exact.
- Reads from the same gene may map to similar but different orthologs across species, requiring care in interpretation for taxonomically diverse comparisons.
- There is an inherent tradeoff in clustering resolution (e.g., choice of k): higher resolution minimizes grouping of distinct functional proteins but makes cross-species grouping harder, and improper sub-group database selection may miss key functional groups.

## Evidence

- [results] The first round of OrthoFinder analysis binned 12,828,537 protein-coding sequences from 687 species into 666,067 ortholog groups, whose size distribution largely follows a power law distribution.: [evidence span withheld — anchor status: not_found]
- [abstract] EcoOmicsDB integration statement.: "All modules are integrated with EcoOmicsDB, an ortholog database that enables comprehensive analysis for species without a reference transcriptome."
- [discussion] KEGG download scale statement.: "All protein-coding genes (n = 12,828,537) from 687 organisms covering all major phyla of eukaryotes were downloaded from KEGG"
