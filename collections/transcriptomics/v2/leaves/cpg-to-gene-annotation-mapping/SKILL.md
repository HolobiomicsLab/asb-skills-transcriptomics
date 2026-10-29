---
name: cpg-to-gene-annotation-mapping
description: Use when you have a differentially methylated CpG list (DMC output from
  ChAMP/minfi in methylR) and need gene-level identifiers to run gene ontology or
  pathway enrichment analysis.
when_to_use_negative:
- Input is already a gene-level list (e.g. normalized beta value tables like testDataFile2,
  used for MDS/PCA), where CpG-to-gene mapping is unnecessary
- The goal is raw-data processing rather than downstream analysis — methylR's upstream
  ChAMP/minfi modules operate on IDAT-level methylation data, not DMC lists
- Running on ARM64 chipset architecture, which methylR does not support
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3676
tools:
- name: methylR
  role: Shiny GUI whose gene features module maps CpG identifiers from the DMC list
    to genes for downstream enrichment analysis
  repo: https://github.com/JD2112/methylr
- name: Docker
  role: Container runtime that packages methylR locally (AMD64) so the annotation
    and enrichment modules can be launched via the shiny interface
  repo: https://github.com/JD2112/methylr-devel
- name: Singularity
  role: Alternative container runtime to run the methylR Docker image directly from
    the terminal, e.g. on HPC clusters
  repo: https://github.com/JD2112/methylr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:dc988259f1f51c1f7b42f4cc7b1c1c05d93992f9610dc3e72176ba8cfcbc15ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - Docker
  - Singularity
  - methylR
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3676
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cpg-to-gene-annotation-mapping

## Summary

Mapping differentially methylated CpG identifiers from Illumina 450K/EPIC arrays to gene features inside methylR's shiny GUI, so that a DMC list can serve as direct input for downstream GO and pathway (KEGG/Reactome/WikiPathways) enrichment modules.

## When to use

Use when you have a differentially methylated CpG list (DMC output from ChAMP/minfi in methylR) and need gene-level identifiers to run gene ontology or pathway enrichment analysis.

## When NOT to use

- Input is already a gene-level list (e.g. normalized beta value tables like testDataFile2, used for MDS/PCA), where CpG-to-gene mapping is unnecessary
- The goal is raw-data processing rather than downstream analysis — methylR's upstream ChAMP/minfi modules operate on IDAT-level methylation data, not DMC lists
- Running on ARM64 chipset architecture, which methylR does not support

## Inputs

- Differentially methylated CpG (DMC) list from ChAMP or minfi (e.g. methylR testDataFile1)
- Illumina 450K or EPIC array CpG identifiers

## Outputs

- Gene list mapped from CpG identifiers, used as input to GO and pathway enrichment modules
- Pathway enrichment results table (KEGG, Reactome, WikiPathways) in the Figure 1e layout

## How to apply

Run the methylR Docker container (AMD64 only) and open the shiny GUI at localhost:3838. Load the differentially methylated data file (e.g. testDataFile1 from the SourceForge testData.zip deposit) into the gene features / pathway enrichment module. The module maps the CpG identifiers to their annotated genes, and the resulting gene list feeds the gene ontology and pathway enrichment (KEGG, Reactome, WikiPathways) analyses with a few mouse clicks. Validate the mapping by confirming the downstream enrichment table is produced in the Figure 1e results-table layout, since a failed CpG-to-gene mapping yields no enrichment output.

## Related tools

- **methylR** (Shiny GUI whose gene features module maps CpG identifiers from the DMC list to genes for downstream enrichment analysis) — https://github.com/JD2112/methylr
- **Docker** (Container runtime that packages methylR locally (AMD64) so the annotation and enrichment modules can be launched via the shiny interface) — https://github.com/JD2112/methylr-devel
- **Singularity** (Alternative container runtime to run the methylR Docker image directly from the terminal, e.g. on HPC clusters) — https://github.com/JD2112/methylr

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- The loaded DMC file is accepted by the gene features/pathway module without CpG identifier errors
- An exported pathway enrichment table exists and is a parseable tabular file
- The enrichment table matches the Figure 1e results-table layout with per-database (KEGG/Reactome/WikiPathways) pathway entries and enrichment statistics
- Enrichment runs reproducibly after closing the browser between complete analyses to clear temporary memory
- Mapped genes are consistent with the DMC list when cross-checked against the published methylR results for the same publicly available dataset

## Limitations

- The methylR Docker container was built on AMD64 OS architecture; ARM64 chipset architecture is not supported
- ChAMP and minfi analyses require closing the browser after one complete analysis to clear temporary memory before starting again
- PCA plotting requires additional group data information ('groupData') beyond the beta value table, so not all input files carry the needed metadata for every module

## Evidence

- [readme] testDataFile1 feeds gene features and pathway modules: [evidence span withheld — anchor status: not_found]
- [readme] methylR offers enrichment as downstream analysis: [evidence span withheld — anchor status: external_doc]
- [abstract] methylR analyzes raw data and downstream analyses: "we developed methylR a graphical tool that can analyze not only the raw data but also performs different downstream analyses with a few mouse clicks"
- [methods] methylR shipped as a Docker container: "*methylR* is packed into docker container that is available online."
- [readme] ARM64 not supported: "PLEASE NOTE: the ARM64 chipset architecture is actually not supported by *methylR*."
