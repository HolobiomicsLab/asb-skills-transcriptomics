---
name: pathway-over-representation-analysis
description: Use when a DMC list from Illumina 450K/EPIC differential methylation
  analysis must be mapped to genes and tested for enriched biological pathways across
  KEGG, Reactome, and WikiPathways.
when_to_use_negative:
- The input is raw IDAT or normalized beta values rather than a DMC list — use methylation
  preprocessing/differential modules instead (beta tables feed MDS/PCA modules, not
  pathway enrichment)
- Intersections of multiple gene/region sets need visualization (e.g. Venn/UpSet)
  — use a dedicated tool such as Intervene rather than methylR's enrichment module
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0203
tools:
- name: methylR
  role: Shiny-based pipeline providing the pathway enrichment module (KEGG/Reactome/WikiPathways
    over-representation and GSEA) that consumes the DMC list
  repo: https://github.com/JD2112/methylr
- name: shiny
  role: Graphical interface through which the DMC list is loaded and enrichment parameters
    (adjustment method, number of pathways) are set
- name: Docker
  role: Container used to run methylR locally (AMD64 only) before accessing the pathway
    analysis module
  repo: https://github.com/JD2112/methylr-devel
- name: Singularity
  role: Alternative container runtime to run the methylR Docker image directly from
    the terminal (e.g. on clusters)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:7388e0d0a46ef7d8a9fd7250092b3725f022644f5adb5d8097e7e42c3603346d
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
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pathway-over-representation-analysis

## Summary

Performs over-representation and gene-set enrichment analysis of differentially methylated CpG (DMC)-derived gene lists against KEGG, Reactome, and WikiPathways databases within methylR's Shiny GUI, downstream of functional (GO) analysis.

## When to use

Use when a DMC list from Illumina 450K/EPIC differential methylation analysis must be mapped to genes and tested for enriched biological pathways across KEGG, Reactome, and WikiPathways.

## When NOT to use

- The input is raw IDAT or normalized beta values rather than a DMC list — use methylation preprocessing/differential modules instead (beta tables feed MDS/PCA modules, not pathway enrichment)
- Intersections of multiple gene/region sets need visualization (e.g. Venn/UpSet) — use a dedicated tool such as Intervene rather than methylR's enrichment module

## Inputs

- DMC list (differentially methylated CpG list) from the upstream differential methylation step, e.g. testDataFile1
- methylR testData.zip from https://sourceforge.net/projects/methylr/files/testData.zip

## Outputs

- Pathway enrichment results table with per-database (KEGG, Reactome, WikiPathways) pathway entries and enrichment statistics (Figure 1e layout)

## How to apply

Download methylR's test data (testData.zip from SourceForge) and extract the DMC list produced by the upstream differential methylation step. Launch methylR locally from its Docker container (docker run --rm -p 3838:3838 jd21/methylr:latest) or via Singularity, and open the packaged Shiny interface at localhost:3838. Load the DMC list into the pathway enrichment module, which maps CpG identifiers to genes, then run over-representation / gene-set enrichment against the three supported databases (KEGG, Reactome, WikiPathways). Select a P-value adjustment method and the number of pathways to report via the GUI options, then export the enrichment results table (the layout reported in Figure 1e).

## Related tools

- **methylR** (Shiny-based pipeline providing the pathway enrichment module (KEGG/Reactome/WikiPathways over-representation and GSEA) that consumes the DMC list) — https://github.com/JD2112/methylr
- **shiny** (Graphical interface through which the DMC list is loaded and enrichment parameters (adjustment method, number of pathways) are set)
- **Docker** (Container used to run methylR locally (AMD64 only) before accessing the pathway analysis module) — https://github.com/JD2112/methylr-devel
- **Singularity** (Alternative container runtime to run the methylR Docker image directly from the terminal (e.g. on clusters))

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- Exported pathway enrichment table exists and is a parseable tabular file
- Table contains per-database pathway entries (KEGG, Reactome, WikiPathways) with enrichment statistics
- Table layout matches the Figure 1e results-table presentation
- CpG identifiers were successfully mapped to genes before enrichment (non-empty gene set derived from the DMC list)
- Chosen P-value adjustment method and number-of-pathways selection are reflected in the number of rows returned

## Limitations

- ARM64 chipset architecture is not supported by methylR; the Docker container is built on AMD64, so enrichment must run on Intel/AMD64 Linux, macOS, or Windows (with WSL2)
- Pathway analysis requires a complete upstream differential methylation analysis (ChAMP or minfi); close the browser after each complete analysis to clear temporary memory before starting again
- Only three pathway databases (KEGG, Reactome, WikiPathways) are supported within this module

## Evidence

- [results] Finding: pathway module performs ORA and GSEA against three databases: [evidence span withheld — anchor status: not_found]
- [intro] Scope: single Shiny solution to pathway analysis: "methylr: a single shiny solution from sequencer data to pathway analysis"
- [readme] Test data source for DMC list input: [evidence span withheld — anchor status: not_found]
- [methods] Test data download location for full pipeline run: "If you want to run with the test data, Please download the testdata from [https://sourceforge.net/projects/methylr/files/testData.zip]"
- [readme] Docker run command grounding the invocation: [evidence span withheld — anchor status: external_doc]
