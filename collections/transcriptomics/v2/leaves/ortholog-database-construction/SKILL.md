---
name: ortholog-database-construction
description: Use when building a KO/protein ortholog reference from hundreds of eukaryote
  proteomes to support read-to-ortholog mapping for non-model species RNA-seq, instead
  of per-species transcript assembly and annotation transfer.
when_to_use_negative:
- When the species of interest already has a well-annotated reference transcriptome/genome
  — direct gene-level quantification (e.g., Kallisto against GRCz11) is preferable
- 'When taxonomically diverse comparisons require gene-level identity: reads from
  the same gene may map to similar but different orthologs across species'
- When the target function is absent from the KO system (e.g., vitellogenin) — ortholog-group-based
  mapping cannot recover it
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: OrthoFinder
  role: All-vs-all orthology inference across the 687 concatenated proteomes to produce
    ortholog groups
- name: KEGGREST
  role: Programmatic download of protein-coding sequences for the 687 KEGG organisms,
    one FASTA per organism
- name: ExpressAnalystR
  role: R package synchronized with the ExpressAnalyst web server that consumes the
    ortholog database for downstream statistical and enrichment analysis
  repo: github.com/xia-lab/ExpressAnalystR
- name: Seq2Fun
  role: Translates cleaned RNA-seq reads and searches them against the ortholog (KO)
    protein database using a seed of seven amino acids with BLOSUM62-scored extension
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:a9887ecb980141494f05abea209e12e3834246b0ca30c990aeb1f27486d9fd99
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalystR
  - KEGGREST
  - OrthoFinder
  - Seq2Fun
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ortholog-database-construction

## Summary

Constructs a high-resolution protein ortholog database (EcoOmicsDB) by downloading all protein-coding sequences from KEGG for 687 eukaryotic organisms and binning ~12.8 million proteins into ortholog groups with OrthoFinder, enabling annotation of RNA-seq data from species without a reference transcriptome.

## When to use

Use when building a KO/protein ortholog reference from hundreds of eukaryote proteomes to support read-to-ortholog mapping for non-model species RNA-seq, instead of per-species transcript assembly and annotation transfer.

## When NOT to use

- When the species of interest already has a well-annotated reference transcriptome/genome — direct gene-level quantification (e.g., Kallisto against GRCz11) is preferable
- When taxonomically diverse comparisons require gene-level identity: reads from the same gene may map to similar but different orthologs across species
- When the target function is absent from the KO system (e.g., vitellogenin) — ortholog-group-based mapping cannot recover it

## Inputs

- KEGG protein-coding sequences (FASTA) for 687 eukaryotic organisms, retrieved via KEGGREST (n = 12,828,537 proteins)
- NCBI taxonomy mapping for partitioning ortholog groups into taxonomic sub-group databases

## Outputs

- Ortholog group assignments for ~12.8 million proteins (666,067 ortholog groups) in EcoOmicsDB format
- Per-species summary table of protein-coding gene counts and ortholog-group memberships (687 species, matching Table 1)
- Power-law fit parameters for the ortholog group-size distribution
- 29 taxonomic sub-group protein databases (e.g., 'Fishes') for Seq2Fun translated read search

## How to apply

Download all protein-coding sequences for the 687 eukaryotic organisms in KEGG using KEGGREST, saving one FASTA per organism (n = 12,828,537 proteins). Concatenate the 687 proteomes and run OrthoFinder for all-vs-all orthology inference with 56 threads (-t 56) and 25 threads for the sequence search stage (-a 25). Parse the OrthoFinder orthogroups output to bin every protein into an ortholog group and tally the total (target: 666,067 groups). Fit the ortholog group-size distribution to a power-law model to confirm the database's structural coherence, and organize groups into taxonomic sub-group databases (EcoOmicsDB currently has 29 sub-groups based on the NCBI taxonomy system) so that read mapping can be restricted to a taxonomically appropriate subset. Choose the clustering resolution k carefully: higher k makes it easier to avoid grouping functionally distinct proteins but harder to group across species.

## Related tools

- **OrthoFinder** (All-vs-all orthology inference across the 687 concatenated proteomes to produce ortholog groups)
- **KEGGREST** (Programmatic download of protein-coding sequences for the 687 KEGG organisms, one FASTA per organism)
- **ExpressAnalystR** (R package synchronized with the ExpressAnalyst web server that consumes the ortholog database for downstream statistical and enrichment analysis) — github.com/xia-lab/ExpressAnalystR
- **Seq2Fun** (Translates cleaned RNA-seq reads and searches them against the ortholog (KO) protein database using a seed of seven amino acids with BLOSUM62-scored extension)

## Examples

```
orthofinder -f concat_proteomes/ -t 56 -a 25
```

## Evaluation signals

- Reproduce the reported scale: 687 organisms, 12,828,537 protein-coding genes downloaded from KEGG, and 666,067 ortholog groups
- Ortholog group-size distribution follows a power law; record fitted parameters and compare against published Table 1
- Per-species summary table covers all 687 species with gene counts and ortholog-group memberships matching Table 1 structure
- Sub-group database partitioning yields 29 taxonomic sub-groups consistent with the NCBI taxonomy system
- Spliced reads from a known organism (e.g., zebrafish mapped to a 'Fishes' database with zebrafish removed) cross-map plausibly, confirming functional coherence of groups

## Limitations

- Choosing the clustering resolution k involves an inherent tradeoff: higher k minimizes grouping of distinct functional proteins but makes cross-species grouping harder
- Improper sub-group database selection can miss key functional groups (e.g., protist reads mapped to the 'fishes' database)
- Reads from the same gene may map to similar but different orthologs in different species, requiring care in taxonomically diverse comparisons

## Evidence

- [intro] High-resolution ortholog database rationale: "the typical steps of transcript assembly and annotation are replaced with the alignment of individual translated reads directly to a comprehensive, high-resolution protein ortholog database"
- [discussion] Database scale and source: "All protein-coding genes (n = 12,828,537) from 687 organisms covering all major phyla of eukaryotes were downloaded from KEGG"
- [discussion] Sub-group organization: "EcoOmicsDB currently consists of 29 taxonomic sub-group databases based on the NCBI taxonomy system"
- [discussion] Clustering resolution tradeoff: "There is an inherent tradeoff between these two objectives because the higher the resolution (increased value of k), the easier to minimize grouping of functional proteins but the harder to group"
