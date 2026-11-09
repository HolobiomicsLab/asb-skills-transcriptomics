---
name: cross-species-orthology-inference
description: Use when building or reproducing an ortholog database (e.g. EcoOmicsDB)
  from many species' proteomes, or when non-model-species RNA-seq reads must be quantified
  against cross-species ortholog groups rather than a reference transcriptome.
when_to_use_negative:
- The species has a well-annotated reference transcriptome/genome with standard gene
  IDs — direct annotation and genome alignment is preferable, since many uncharacterized
  IDs remain even in published reference genomes but mapping through orthologs is
  unnecessary overhead
- Reads can be mapped directly to KEGG ortholog (KO) databases with Seq2Fun's translated
  search — no custom ortholog group construction is needed
- A single-species analysis where cross-species orthology introduces ambiguity (reads
  from the same gene may map to similar but different orthologs across species)
edam_topics:
- http://edamontology.org/topic_0114
tools:
- name: OrthoFinder
  role: Performs all-vs-all orthology inference over the concatenated 687-species
    proteome to bin 12,828,537 proteins into 666,067 ortholog groups; run with -t
    56 and -a 25
- name: KEGGREST
  role: Downloads all protein-coding sequences for the 687 eukaryotic KEGG organisms,
    saving one FASTA per organism
- name: KEGG
  role: Source of the protein-coding gene sequences for all 687 organisms covering
    the major eukaryotic phyla
- name: EcoOmicsDB
  role: Downstream ortholog database built from the inferred ortholog groups and organized
    into 29 taxonomic sub-groups; integrated with ExpressAnalyst for analysis of species
    without a reference transcriptome
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:f2890bc7cb9473e7df8a296e1e6932ac76fb8383a2693194c3f359c2a758c729
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
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cross-species-orthology-inference

## Summary

Bins protein-coding sequences from hundreds of eukaryotic species into ortholog groups (as done to build EcoOmicsDB's ortholog database backing ExpressAnalyst), so that RNA-seq reads from species without a reference transcriptome can be mapped to shared functional units.

## When to use

Use when building or reproducing an ortholog database (e.g. EcoOmicsDB) from many species' proteomes, or when non-model-species RNA-seq reads must be quantified against cross-species ortholog groups rather than a reference transcriptome.

## When NOT to use

- The species has a well-annotated reference transcriptome/genome with standard gene IDs — direct annotation and genome alignment is preferable, since many uncharacterized IDs remain even in published reference genomes but mapping through orthologs is unnecessary overhead
- Reads can be mapped directly to KEGG ortholog (KO) databases with Seq2Fun's translated search — no custom ortholog group construction is needed
- A single-species analysis where cross-species orthology introduces ambiguity (reads from the same gene may map to similar but different orthologs across species)

## Inputs

- Per-organism protein FASTA files for eukaryotic species (one FASTA per organism, downloaded from KEGG via KEGGREST)
- Concatenated multi-species proteome FASTA for all-vs-all orthology search
- OrthoFinder orthogroups results directory (orthogroup-to-protein assignments)

## Outputs

- 666,067 ortholog groups covering 12,828,537 proteins from 687 species
- Per-species summary table: number of protein-coding genes and ortholog-group memberships per species (Table 1 structure)
- Power-law fit parameters for the ortholog group-size distribution
- Taxonomic sub-group ortholog databases (29 NCBI-taxonomy-based sub-groups in EcoOmicsDB)

## How to apply

Download all protein-coding sequences for the target eukaryotic organisms from KEGG using KEGGREST, saving one FASTA per organism (EcoOmicsDB uses 687 species, n = 12,828,537 proteins). Concatenate the 687 proteomes and run OrthoFinder for all-vs-all orthology inference (EcoOmicsDB used 56 threads, -t 56, and 25 threads for the sequence search stage, -a 25). Parse the orthogroups results to bin every protein into an ortholog group and tally the total number of groups (666,067 in EcoOmicsDB), then build a per-species summary table of protein-coding gene counts and ortholog-group memberships. Fit the ortholog group-size distribution to a power-law model to confirm the expected large-scale behavior; the resulting groups become the protein ortholog database (optionally organized into taxonomic sub-groups, e.g. EcoOmicsDB's 29 NCBI-taxonomy-based sub-group databases) into which translated RNA-seq reads are aligned for annotation transfer.

## Related tools

- **OrthoFinder** (Performs all-vs-all orthology inference over the concatenated 687-species proteome to bin 12,828,537 proteins into 666,067 ortholog groups; run with -t 56 and -a 25)
- **KEGGREST** (Downloads all protein-coding sequences for the 687 eukaryotic KEGG organisms, saving one FASTA per organism)
- **KEGG** (Source of the protein-coding gene sequences for all 687 organisms covering the major eukaryotic phyla)
- **EcoOmicsDB** (Downstream ortholog database built from the inferred ortholog groups and organized into 29 taxonomic sub-groups; integrated with ExpressAnalyst for analysis of species without a reference transcriptome)

## Examples

```
orthofinder -f concatenated_proteomes/ -t 56 -a 25
```

## Evaluation signals

- Pipeline recovers exactly 687 organisms and 12,828,537 protein-coding sequences after the KEGG download
- OrthoFinder binning yields the reported scale of 666,067 ortholog groups (within tolerance for tool/version drift)
- Per-species summary table matches the structure and counts of published Table 1
- Ortholog group-size distribution follows a power law with parameters consistent with the published fit
- Sub-group databases partition consistently under the NCBI taxonomy (29 sub-groups in EcoOmicsDB)

## Limitations

- Reads from the same gene may be mapped to similar but different orthologs for different species, requiring care in interpretation for taxonomically diverse comparisons
- Improper sub-group database selection (e.g. mapping protist reads to the 'fishes' database) can miss key functional groups such as vitellogenin, which is not in the KO system
- KO-based ortholog annotation has limited transcriptome coverage — only 76.16% of human and 61.40% of zebrafish protein-coding genes carry KOs — so group sizes and coverage are bounded by KO annotation completeness
- Choosing the clustering resolution k involves an inherent tradeoff between grouping distinct functional proteins and grouping across species

## Evidence

- [discussion] All protein-coding genes (n = 12,828,537) from 687 organisms covering all major phyla of eukaryotes were downloaded from KEGG: "All protein-coding genes (n = 12,828,537) from 687 organisms covering all major phyla of eukaryotes were downloaded from KEGG"
- [abstract] EcoOmicsDB abstract integration: "All modules are integrated with EcoOmicsDB, an ortholog database that enables comprehensive analysis for species without a reference transcriptome."
- [discussion] EcoOmicsDB sub-groups: "EcoOmicsDB currently consists of 29 taxonomic sub-group databases based on the NCBI taxonomy system"
- [intro] Ortholog database replaces assembly/annotation: "the typical steps of transcript assembly and annotation are replaced with the alignment of individual translated reads directly to a comprehensive, high-resolution protein ortholog database"
- [results] Power-law size distribution finding: [evidence span withheld — anchor status: not_found]
