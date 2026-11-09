---
name: circrna-output-table-parsing
description: Use when a CircAST result file (tab-separated, 13 columns ending in junction
  read counts, FPKM, TPM, assigned reads) must be loaded and structured for downstream
  RI stratification, isoform comparison, or expression-accuracy analysis.
when_to_use_negative:
- Input is a raw circRNA junction list from CIRCexplorer2/CIRI2/UROBORUS rather than
  CircAST's post-assembly output table
- Analysis requires intronic or intergenic circular transcripts, which the current
  CircAST pipeline may miss
- The sample was not RNase R-treated, since CircAST can be applied only to RNase R-treated
  samples
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0085
tools:
- name: CircAST
  role: Produces the 13-column output table of assembled circular transcripts with
    FPKM/TPM expression levels that this skill parses
  repo: https://github.com/xiaofengsong/CircAST
- name: pandas
  role: Loads and structures the tab-separated CircAST output table for RI computation
    and stratified metric calculation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:b6c432f6a8c3ab0430894ac2a4c824072352ebeb3d88b1c64e7cc851ee7a644a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CircAST
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# circrna-output-table-parsing

## Summary

Parse and structure the tab-separated output table produced by CircAST's full-length circRNA assembly and quantification run, whose 13 columns (bed-like coordinates through FPKM/TPM and assigned read counts) serve as the substrate for downstream analyses such as relation-index (RI) stratification of assembly accuracy.

## When to use

Use when a CircAST result file (tab-separated, 13 columns ending in junction read counts, FPKM, TPM, assigned reads) must be loaded and structured for downstream RI stratification, isoform comparison, or expression-accuracy analysis.

## When NOT to use

- Input is a raw circRNA junction list from CIRCexplorer2/CIRI2/UROBORUS rather than CircAST's post-assembly output table
- Analysis requires intronic or intergenic circular transcripts, which the current CircAST pipeline may miss
- The sample was not RNase R-treated, since CircAST can be applied only to RNase R-treated samples

## Inputs

- CircAST output table (tab-separated, 13 columns, first 3 in bed format: chromosome, start, end)
- Simulated ground-truth circular transcript definitions (for sensitivity/precision comparison)
- Simulated BSJ site coordinates (for relation-index computation)

## Outputs

- Per-site RI assignments keyed by BSJ genomic coordinates
- RI-stratified performance metrics table (sensitivity, precision, expression correlation), e.g. ri_performance_metrics.csv
- Structured DataFrame of CircAST-assembled circular transcripts with junction counts, FPKM, TPM, and assigned reads

## How to apply

Load the CircAST output table with pandas (tab-separated, no header) and name columns per the documented schema: chromosome, start, end, circular transcript name, parental gene, strand, exon count, exon sizes, exon offsets, junction read counts, FPKM, TPM, and assigned reads. The first three columns follow bed format, so chromosome/start/end can be used directly to reconstruct back-splice junction (BSJ) genomic coordinates and derive the relation index (RI) as the cardinality of each site's overlap equivalence class. The FPKM/TPM columns support correlation of estimated versus theoretical expression, and the transcript-name plus exon-structure columns (number of exons, exon sizes, exon offsets) support comparison of assembled circular transcripts against simulated ground truth. Validate parsing by confirming the row count matches the junction input and that coordinate, read-count, and expression columns are numeric.

## Related tools

- **CircAST** (Produces the 13-column output table of assembled circular transcripts with FPKM/TPM expression levels that this skill parses) — https://github.com/xiaofengsong/CircAST
- **pandas** (Loads and structures the tab-separated CircAST output table for RI computation and stratified metric calculation)

## Examples

```
import pandas as pd; df = pd.read_csv('CircAST_result.txt', sep='\t', header=None, names=['chr','start','end','circ_name','gene','strand','n_exons','exon_sizes','exon_offsets','junction_reads','fpkm','tpm','assigned_reads'])
```

## Evaluation signals

- Parsed table has exactly 13 columns matching the documented schema (chromosome through read counts of assigned reads)
- Chromosome/start/end columns parse as valid bed-format genomic coordinates for every assembled transcript
- Junction read counts, FPKM, and TPM columns are numeric and non-negative; row count is consistent with the input junction list filtered at the supporting-read threshold (default 10)
- RI-stratified sensitivity falls above 93% for RI 1-4 and drops to about 86% for RI > 4, within the reported simulation ranges (sensitivity 85.63%-94.32%, precision 81.96%-87.55%)
- Correlation between theoretical and CircAST-estimated FPKM/TPM per RI bin falls within the reported 0.85-0.99 range

## Limitations

- CircAST can be applied only to RNase R-treated samples; remnant linear mRNA can affect assembly and generate false transcripts
- The current CircAST pipeline may miss intronic or intergenic circular transcripts, so parsed tables cover only exon-annotated circular transcripts
- Output from other BSJ detection tools requires format conversion to the CircAST input format before an equivalent table can be produced

## Evidence

- [readme] The first 3 columns are the same with bed file format.: "The first 3 columns are the same with bed file format."
- [readme] Output columns 10-13 define quantification fields: "10) Read counts (junction reads) 11) Expression level (FPKM) 12) Expression level (TPM) 13) Read counts (assigned reads)"
- [abstract] Simulation sensitivity/precision ranges: "CircAST correctly assembled the full sequences of circRNAs with a sensitivity of 85.63%-94.32% and a precision of 81.96%-87.55%"
- [discussion] CircAST applies only to RNase R-treated samples: "CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over."
