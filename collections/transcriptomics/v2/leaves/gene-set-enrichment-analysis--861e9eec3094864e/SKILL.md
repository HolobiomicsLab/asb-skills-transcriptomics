---
name: gene-set-enrichment-analysis
description: Use when a DMC (differentially methylated CpG) list from a 450K/EPIC
  Illumina array analysis needs to be translated into enriched biological pathways
  with per-database enrichment statistics.
when_to_use_negative:
- Input is raw IDAT methylation array data — this module operates downstream on an
  already-derived DMC list, not on raw files
- Input is a normalized beta value table (testDataFile2), which is intended for MDS/PCA
  modules rather than pathway enrichment
- The goal is genomic-feature or GO functional annotation rather than KEGG/Reactome/WikiPathways
  over-representation or GSEA
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3173
tools:
- name: methylR
  role: Hosts the pathway enrichment module in its shiny GUI; takes the DMC list as
    direct input and runs over-representation/GSEA against three databases
  repo: https://github.com/JD2112/methylr
- name: shiny
  role: R web framework providing the graphical interface through which the pathway
    analysis options (databases, P-value adjustment, number of pathways) are selected
  repo: https://github.com/JD2112/methylr
- name: Docker
  role: Container packaging methylR for local execution so the pathway module can
    be launched reproducibly (AMD64 architecture)
  repo: https://github.com/JD2112/methylr-devel
- name: Singularity
  role: Alternative container runtime to launch the methylR Docker image from the
    terminal, e.g. on an HPC cluster
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:2f417c25f7f4561a82fddfc1a01b24014ee9db1bbe268b535cd66ad4ae338ea9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - Docker
  - Singularity
  - methylR
  - shiny
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# gene-set-enrichment-analysis

## Summary

Perform pathway-level biological interpretation downstream of differential methylation by running over-representation and gene-set enrichment analysis against KEGG, Reactome, and WikiPathways databases within methylR's shiny GUI.

## When to use

Use when a DMC (differentially methylated CpG) list from a 450K/EPIC Illumina array analysis needs to be translated into enriched biological pathways with per-database enrichment statistics.

## When NOT to use

- Input is raw IDAT methylation array data — this module operates downstream on an already-derived DMC list, not on raw files
- Input is a normalized beta value table (testDataFile2), which is intended for MDS/PCA modules rather than pathway enrichment
- The goal is genomic-feature or GO functional annotation rather than KEGG/Reactome/WikiPathways over-representation or GSEA

## Inputs

- DMC list (differentially methylated CpG table) from upstream differential methylation analysis
- testDataFile1 (differentially methylated data file usable for pathway enrichment analysis)
- group data information if additional annotation context is required

## Outputs

- Pathway enrichment results table with per-database (KEGG, Reactome, WikiPathways) pathway entries and enrichment statistics (Figure 1e layout)

## How to apply

Take the DMC list produced by the upstream differential methylation step (e.g. from ChAMP or minfi, or the packaged testDataFile1) and load it into the pathway enrichment module of methylR's shiny interface. Map the CpG identifiers to genes, then run over-representation / gene-set enrichment against the three supported databases (KEGG, Reactome, WikiPathways) via the GUI options. Choose a P-value adjustment method from the options offered and set the number of pathways to report. Export the enrichment results table, which should match the Figure 1e layout of per-database pathway entries with enrichment statistics. The same module can also consume testDataFile1 from the methylr GitHub data directory, which is explicitly designated for pathway enrichment analysis.

## Related tools

- **methylR** (Hosts the pathway enrichment module in its shiny GUI; takes the DMC list as direct input and runs over-representation/GSEA against three databases) — https://github.com/JD2112/methylr
- **shiny** (R web framework providing the graphical interface through which the pathway analysis options (databases, P-value adjustment, number of pathways) are selected) — https://github.com/JD2112/methylr
- **Docker** (Container packaging methylR for local execution so the pathway module can be launched reproducibly (AMD64 architecture)) — https://github.com/JD2112/methylr-devel
- **Singularity** (Alternative container runtime to launch the methylR Docker image from the terminal, e.g. on an HPC cluster)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest  # then open http://localhost:3838 and load the DMC list into the pathway enrichment module
```

## Evaluation signals

- The exported enrichment results table exists and is a parseable tabular file
- The table matches the Figure 1e results-table layout: per-database pathway entries with enrichment statistics
- Entries appear for each of the three supported databases: KEGG, Reactome, and WikiPathways
- The chosen P-value adjustment method and number-of-pathways selection are reflected in the number of rows reported
- CpG identifiers in the input DMC list were successfully mapped to genes (non-empty gene-set input produced enrichment results)

## Limitations

- The methylR Docker container is built for AMD64 only; the ARM64 chipset architecture is not supported
- Only three pathway databases are supported (KEGG, Reactome, WikiPathways); the pipeline was tested on Linux and Mac-based systems
- Only one complete analysis should be run per browser session; close the browser afterwards to clear temporary memory

## Evidence

- [full_text] methylR's pathway analysis module takes the DMC list as direct input and performs over-representation and gene-set enrichment analysis against three databases (KEGG, Reactome, and WikiPathways): [evidence span withheld — anchor status: not_found]
- [readme] testDataFile1 - differentially methylated data file, can be used for gene features, Volcano plot, chromosome map, gene ontology and pathway enrichment analysis modules.: [evidence span withheld — anchor status: not_found]
- [intro] methylr: a single shiny solution from sequencer data to pathway analysis: "methylr: a single shiny solution from sequencer data to pathway analysis"
- [abstract] We implemented eight downstream analysis modules that can perform multidimensional analyses to pathway enrichment: "We implemented eight downstream analysis modules that can perform multidimensional analyses to pathway enrichment"
- [methods] NOTE: the ARM64 chipset architecture is actually not supported by methylR.: "NOTE: the ARM64 chipset architecture is actually not supported by methylR."
