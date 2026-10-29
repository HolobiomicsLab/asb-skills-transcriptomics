---
name: fusion-gene-annotation-interpretation
description: Use when an INTEGRATE-Circ (or compatible BEDPE-style) fusion-caller
  output contains junction definitions for fcircRNAs and exon-level backsplice visualizations
  with cytoband context are needed for interpretation.
when_to_use_negative:
- Input is raw FASTQ sequencing reads with no fusion-caller output — detection must
  first be performed with INTEGRATE-Circ before visualization is possible
- Goal is unbiased fcircRNA detection or quantification rather than visualization/interpretation
  of already-called junctions
- Fusion calls cannot be converted to the BEDPE/fcirc.txt junction format expected
  by INTEGRATE-Vis
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: INTEGRATE-Vis
  role: Renders the default-setting exon-level backsplice structure and cytoband context
    figures for each fcircRNA junction definition
  repo: https://github.com/ChrisMaherLab/INTEGRATE-Vis
- name: INTEGRATE-Circ
  role: Upstream fusion junction detector whose fcirc.txt output supplies the fcircRNA
    junction definitions to be visualized
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad569
    title: 'INTEGRATE-Circ and INTEGRATE-Vis: unbiased detection and visualization
      of fusion-derived circular RNA'
  source_content_hash: sha256:46984bf1241862a42b8b38e5842e274a2196fe8c5d457efdb031400d161a22b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChrisMaherLab/INTEGRATE-Vis
  tools:
  - INTEGRATE-Circ
  - INTEGRATE-Vis
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# fusion-gene-annotation-interpretation

## Summary

Interprets fusion-caller junction annotations (e.g., INTEGRATE-Circ's fcirc.txt) by rendering each fusion-derived circRNA (fcircRNA) as an exon-level backsplice structure with cytoband/genomic context using INTEGRATE-Vis, so detected fusion junctions become inspectable figures.

## When to use

Use when an INTEGRATE-Circ (or compatible BEDPE-style) fusion-caller output contains junction definitions for fcircRNAs and exon-level backsplice visualizations with cytoband context are needed for interpretation.

## When NOT to use

- Input is raw FASTQ sequencing reads with no fusion-caller output — detection must first be performed with INTEGRATE-Circ before visualization is possible
- Goal is unbiased fcircRNA detection or quantification rather than visualization/interpretation of already-called junctions
- Fusion calls cannot be converted to the BEDPE/fcirc.txt junction format expected by INTEGRATE-Vis

## Inputs

- fcirc.txt junction-definition output from INTEGRATE-Circ
- Ensembl GTF annotation file (e.g., Homo_sapiens.GRCh37.75.gtf or Homo_sapiens.GRCh38.85.gtf)
- Ideogram cytoband TSV (Ideogram.37.tsv or Ideogram.38.tsv)
- Reference genome FASTA (e.g., GRCh37_r75.all.fa)
- Optionally: BEDPE fusion file and RNA-seq BAM alignments for the structure utility

## Outputs

- Rendered visualization panels, one per fcircRNA, showing exon-level backsplice structure and cytoband/genomic context (analogous to article Fig 5A-C)

## How to apply

Load the fcirc.txt output from the upstream INTEGRATE-Circ run and extract the junction definitions for the fcircRNAs of interest (e.g., the three validated HCC1395 fcircRNAs circTTC33::PRKAA1, circLINC00630::LLOXNC01-237H1.2, and circRP11-540B6.3::FAN1). Parse a GTF-derived exon annotation (Ensembl GRCh37 v75 or GRCh38 v85 GTF per the INTEGRATE-Vis README) to obtain canonical exon boundaries for each fusion partner gene. Run the INTEGRATE-Vis structure utility on each junction, supplying the GTF, reference FASTA, and the ideogram cytoband TSV (Ideogram.37.tsv or Ideogram.38.tsv) so each fcircRNA is rendered with its backsplice exons and genomic context, using default settings. INTEGRATE-Vis requires Python, CMake, GCC, Matplotlib, samtools, bedtools, and gtfToGenePred to be installed beforehand.

## Related tools

- **INTEGRATE-Vis** (Renders the default-setting exon-level backsplice structure and cytoband context figures for each fcircRNA junction definition) — https://github.com/ChrisMaherLab/INTEGRATE-Vis
- **INTEGRATE-Circ** (Upstream fusion junction detector whose fcirc.txt output supplies the fcircRNA junction definitions to be visualized) — https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC

## Examples

```
python ~/opt/bin/Integrate-vis.py structure --help
```

## Evaluation signals

- The expected number of rendered panels is produced — one per supplied fcircRNA junction (e.g., three panels for the three validated HCC1395 fcircRNAs)
- Each panel depicts the exon-level backsplice structure of exactly one fcircRNA, with exons matching the GTF annotation of the fusion partner genes
- Each panel includes cytoband/ideogram genomic context consistent with the ideogram TSV supplied
- Rendered panels qualitatively match the article's Fig 5A-C depictions of the HCC1395 fcircRNAs under default settings
- The junction definitions rendered correspond one-to-one with entries extracted from the upstream fcirc.txt file

## Limitations

- No version number is stated for INTEGRATE-Circ in the manuscript's installation instructions; behavior may drift with repository updates
- INTEGRATE-Vis depends on multiple external prerequisites (Python, CMake, GCC, Matplotlib, samtools, bedtools, gtfToGenePred) that must be installed separately
- INTEGRATE-Vis expects BEDPE-format fusion input; outputs from other fusion callers must be converted to BEDPE first
- Validation of the rendered figures in the source task is qualitative (comparison to Fig 5A-C), not automated

## Evidence

- [methods] the output fcirc.txt file can be supplied as input to the most recent version of the [INTEGRATE-vis tool developed by the Chris Maher Lab]: "the output fcirc.txt file can be supplied as input to the most recent version of the [INTEGRATE-vis tool developed by the Chris Maher Lab]"
- [readme] INTEGRATE-Vis is a gene fusion visualization tool. It is written in Python.: [evidence span withheld — anchor status: external_doc]
- [readme] Input file formats for INTEGRATE-Vis include BEDPE, BAM, GTF, FASTA, and TSV.: [evidence span withheld — anchor status: external_doc]
- [intro] capable of identifying gene fusion transcripts and various isoforms that may result from these fusions, including fusion-derived circRNA (fcircRNA): "INTEGRATE-Circ is a fusion junction detection tool, capable of identifying gene fusion transcripts and various isoforms that may result from these fusions, including fusion-derived circRNA (fcircRNA)."
