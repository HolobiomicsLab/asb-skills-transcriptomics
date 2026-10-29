---
name: eukaryotic-protein-sequence-retrieval
description: Use when building or extending a protein ortholog database from KEGG
  eukaryotic proteomes — i.e., you need per-organism FASTA files for downstream all-vs-all
  orthology binning.
when_to_use_negative:
- Input is already a per-organism proteome FASTA set or a finished ortholog database
  — go straight to orthology binning or read-mapping instead
- You need taxon-specific reference genes absent from KEGG (e.g., vitellogenin, 'a
  precursor to egg yolk protein formation and a biomarker of interest, is not found
  in the KO system')
- Small, taxonomically narrow analyses — mapping against all 687 species risks mapping
  reads from one gene to similar but different orthologs across distant species
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_0157
tools:
- name: KEGGREST
  role: Programmatic retrieval of all protein-coding sequences for each KEGG eukaryotic
    organism, saved as one FASTA per species
- name: OrthoFinder
  role: Downstream all-vs-all orthology inference over the concatenated proteomes
    to bin proteins into ortholog groups
- name: ExpressAnalystR
  role: R package synchronized with the ExpressAnalyst web server, whose modules consume
    the EcoOmicsDB ortholog database built by this retrieval step
  repo: github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:1c52c0b1522927af6ee7ad4d07c217d7b911327b183477c5d0c13efcdf889998
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalystR
  - KEGGREST
  - OrthoFinder
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# eukaryotic-protein-sequence-retrieval

## Summary

Bulk-download all protein-coding sequences for eukaryotic organisms from KEGG using KEGGREST, saving one FASTA per species, to construct a protein/ortholog database (the EcoOmicsDB foundation, n = 12,828,537 proteins across 687 species).

## When to use

Use when building or extending a protein ortholog database from KEGG eukaryotic proteomes — i.e., you need per-organism FASTA files for downstream all-vs-all orthology binning.

## When NOT to use

- Input is already a per-organism proteome FASTA set or a finished ortholog database — go straight to orthology binning or read-mapping instead
- You need taxon-specific reference genes absent from KEGG (e.g., vitellogenin, 'a precursor to egg yolk protein formation and a biomarker of interest, is not found in the KO system')
- Small, taxonomically narrow analyses — mapping against all 687 species risks mapping reads from one gene to similar but different orthologs across distant species

## Inputs

- List of KEGG organism identifiers for target eukaryotic species (687 organisms in the reference build)
- KEGG database access via KEGGREST
- Per-organism protein-coding sequences exported as one FASTA per organism

## Outputs

- One FASTA file of protein-coding sequences per organism (687 files)
- Concatenated multi-organism proteome FASTA (12,828,537 proteins) for orthology inference
- Ortholog groups (target: 666,067 groups) with group-size distribution and fitted power-law parameters
- Per-species summary table: number of protein-coding genes and ortholog-group memberships per species

## How to apply

Use KEGGREST to programmatically fetch every protein-coding sequence for each of the target eukaryotic organisms in KEGG, writing one FASTA file per organism rather than a single merged file so per-species gene counts can be tallied. Concatenate the per-organism proteomes (target scale: 687 organisms, 12,828,537 proteins) and feed the merged FASTA to orthology inference (EcoOmicsDB used OrthoFinder with 56 threads for all-vs-all inference and 25 threads for the sequence search stage, -t 56 -a 25). Parse the orthogroups results to bin every protein into an ortholog group and record the total group count, plus a per-species summary table of protein-coding genes and ortholog-group memberships mirroring Table 1. Fit the ortholog group-size distribution to a power-law model as a sanity check — the published distribution follows a power law, which is a key validation signal that retrieval and binning were complete and correct.

## Related tools

- **KEGGREST** (Programmatic retrieval of all protein-coding sequences for each KEGG eukaryotic organism, saved as one FASTA per species)
- **OrthoFinder** (Downstream all-vs-all orthology inference over the concatenated proteomes to bin proteins into ortholog groups)
- **ExpressAnalystR** (R package synchronized with the ExpressAnalyst web server, whose modules consume the EcoOmicsDB ortholog database built by this retrieval step) — github.com/xia-lab/ExpressAnalystR

## Examples

```
library(KEGGREST); aaseq <- keggGet("gox/genes", "aaseq"); writeLines(aaseq, "gox.fa")  # repeat per organism: one FASTA per species
```

## Evaluation signals

- Retrieval scale matches the published build: 687 organisms and 12,828,537 protein-coding genes
- Orthology output matches 666,067 ortholog groups over the merged proteomes
- Per-organism FASTA files are valid, parseable FASTA and per-species gene counts reproduce the Table 1 structure
- Ortholog group-size distribution fits a power-law model, consistent with the published finding
- Per-species summary table of genes and ortholog-group memberships is complete for all 687 species

## Limitations

- Improper protein database selection can miss functional groups — 'if the protein database is not selected properly (e.g., reads from a protist sample are mapped to the "fishes" database), key functional groups may be missed'
- KO-system coverage is incomplete, so the retrieved/annotated space misses some genes of interest; only 14,964 of 19,648 human (76.16%) and 16,322 of 26,584 zebrafish (61.40%) protein-coding genes carry KOs
- The ortholog database resolution involves a tradeoff: 'There is an inherent tradeoff between these two objectives because the higher the resolution (increased value of k), the easier to minimize grouping of functional proteins but the harder to group'

## Evidence

- [discussion] EcoOmicsDB proteome scale from KEGG: "All protein-coding genes (n = 12,828,537) from 687 organisms covering all major phyla of eukaryotes were downloaded from KEGG"
- [discussion] EcoOmicsDB taxonomy structure: "EcoOmicsDB currently consists of 29 taxonomic sub-group databases based on the NCBI taxonomy system"
- [readme] Sequential function execution requirement: "Every function must be executed in sequence as it is shown on the R Command history, please do not skip any commands as this can result in errors downstream."
