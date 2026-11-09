---
name: segment-type-annotation-eval-types
description: Use when you hold hyb-format hybrid records from miRNA/target ligation
  experiments (e.g. qCLASH, CLASH) and need per-record segment-type and miRNA annotations
  before tallying hybrid types or miRNA characteristics across replicates.
when_to_use_negative:
- Input is raw sequencing data (e.g. FASTQ) not yet converted to hyb format — this
  skill operates on existing HybRecord objects, not raw reads
- The goal is target or fold analysis without type/miRNA annotation needs — Target
  Analysis, Grouped Target Analysis, or Fold Analysis may be the appropriate hybkit
  analysis instead
- Records are fold-only data (vienna/CT files) without a paired .hyb file — eval_types()/eval_mirna()
  operate on HybRecord objects parsed from hyb files
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: hybkit
  role: Python API providing HybFile/HybRecord classes and the eval_types() and eval_mirna()
    methods that assign segment types and identify miRNA segments per record
  repo: github.com/RenneLab/hybkit
- name: hyb_check
  role: Run before annotation to parse hyb files and check record integrity (15-16
    tab-delimited columns per line)
  repo: github.com/RenneLab/hybkit
- name: hyb_filter
  role: Removes artifactual ribosomal- and mitochondrial-RNA hybrids (and duplicate
    hybrids by HybRecord.id) prior to type/miRNA annotation and tallying
  repo: github.com/RenneLab/hybkit
- name: hyb_analyze
  role: Consumes the eval_types()/eval_mirna() annotations to quantify sequence types
    and miRNA types in a hyb file and plot results
  repo: github.com/RenneLab/hybkit
- name: hyb_eval
  role: Command-line alternative for evaluating hyb records to identify/assign segment
    types and miRNAs using custom criteria
  repo: github.com/RenneLab/hybkit
- name: matplotlib
  role: Backend for hybkit's built-in plotting of the annotated type/miRNA analysis
    results (pie charts, bar charts)
  repo: https://matplotlib.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:31187dfcb7c9334c3af21dab56b68cad732ceeab09291dfd7beff1c1ad99d156
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioPython
  - Python3
  - hyb_analyze
  - hyb_check
  - hyb_eval
  - hyb_filter
  - hybkit
  - matplotlib
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# segment-type-annotation-eval-types

## Summary

Annotates each record in hyb-format chimeric (hybrid) RNA files with segment types (e.g. miRNA, mRNA, lncRNA) and identifies the miRNA segment (5p/3p/both/none) using hybkit's eval_types() and eval_mirna() HybRecord methods, as preparation for type/miRNA count analyses.

## When to use

Use when you hold hyb-format hybrid records from miRNA/target ligation experiments (e.g. qCLASH, CLASH) and need per-record segment-type and miRNA annotations before tallying hybrid types or miRNA characteristics across replicates.

## When NOT to use

- Input is raw sequencing data (e.g. FASTQ) not yet converted to hyb format — this skill operates on existing HybRecord objects, not raw reads
- The goal is target or fold analysis without type/miRNA annotation needs — Target Analysis, Grouped Target Analysis, or Fold Analysis may be the appropriate hybkit analysis instead
- Records are fold-only data (vienna/CT files) without a paired .hyb file — eval_types()/eval_mirna() operate on HybRecord objects parsed from hyb files

## Inputs

- hyb-format hybrid RNA files (e.g. three qCLASH .hyb files from GEO accession GSE101978)
- HybRecord objects from hybkit.HybFile iteration
- Optionally associated vienna or CT fold files for the same records

## Outputs

- Per-record segment-type annotations on HybRecord objects (e.g. miRNA, mRNA, lncRNA per segment)
- Per-record miRNA identification (5p/3p/both/none)
- Annotated records ready for hyb_analyze type/miRNA count tables and plotting

## How to apply

Iterate over each .hyb file with hybkit's HybFile file-iterator, returning each line as a HybRecord object; first run record integrity checks (hyb_check) so every line carries the expected 15-16 tab-delimited columns, and filter out artifactual ribosomal- and mitochondrial-RNA hybrids with hyb_filter so they do not pollute the tallies. Call eval_types() on each HybRecord to assign segment types to the 5p and 3p segments under hybkit's default criteria (or custom criteria via hyb_eval), then call eval_mirna() to identify which segment(s) are miRNA (5p/3p/both/none). The rationale is that downstream analyses (hyb_analyze type/miRNA analysis, target analysis, fold analysis) all depend on these per-record annotations, so they must be assigned before aggregation. Annotate all three qCLASH replicates (GSE101978, Gay et al. 2018) identically so counts are comparable, then aggregate into the type and miRNA count tables per replicate and combined across the dataset.

## Related tools

- **hybkit** (Python API providing HybFile/HybRecord classes and the eval_types() and eval_mirna() methods that assign segment types and identify miRNA segments per record) — github.com/RenneLab/hybkit
- **hyb_check** (Run before annotation to parse hyb files and check record integrity (15-16 tab-delimited columns per line)) — github.com/RenneLab/hybkit
- **hyb_filter** (Removes artifactual ribosomal- and mitochondrial-RNA hybrids (and duplicate hybrids by HybRecord.id) prior to type/miRNA annotation and tallying) — github.com/RenneLab/hybkit
- **hyb_analyze** (Consumes the eval_types()/eval_mirna() annotations to quantify sequence types and miRNA types in a hyb file and plot results) — github.com/RenneLab/hybkit
- **hyb_eval** (Command-line alternative for evaluating hyb records to identify/assign segment types and miRNAs using custom criteria) — github.com/RenneLab/hybkit
- **matplotlib** (Backend for hybkit's built-in plotting of the annotated type/miRNA analysis results (pie charts, bar charts)) — https://matplotlib.org/

## Examples

```
with hybkit.HybFile.open(in_file, 'r') as hyb_file: for hyb_record in hyb_file: hyb_record.eval_types()
```

## Evaluation signals

- hyb_check reports every line in each replicate's .hyb file carries the expected 15-16 tab-delimited columns before annotation
- Ribosomal- and mitochondrial-RNA hybrid counts removed by hyb_filter are excluded, and final type/miRNA count table totals equal the number of filtered hybrid records per replicate
- Combined hybrid-type output categories (e.g. miRNA/mRNA-containing classes) mirror the example_01_type_mirna_analysis outputs shipped in the hybkit repository
- The combined_analysis_type_hybrid_types.csv table and pie chart render with expected segment-type categories alongside the CSV
- eval_mirna() reports a coherent 5p/3p/both/none assignment for every miRNA-containing record, with no records left untyped

## Limitations

- Segment-type assignment depends on hybkit's default or custom criteria (via hyb_eval); unexpected segment types require criteria customization
- Older hybkit scripts used type_analysis/mirna_analysis method names — these were renamed to eval_types and eval_mirna, so legacy API code must be updated
- Artifactual ribosomal- and mitochondrial-RNA hybrids and duplicate hybrids (by HybRecord.id) must be filtered first, or they inflate the type/miRNA tallies

## Evidence

- [methods] eval_types/eval_mirna renaming: "Renamed HybRecord type_analysis and mirna_analysis to eval_types and eval_mirna, respectively"
- [methods] record integrity QC: "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [methods] artifact removal before analysis: "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
- [readme] hybkit scope: "Welcome to *hybkit*, a toolkit for analysis of hyb-format chimeric (hybrid) RNA sequence data defined with the Hyb software package"
- [readme] HybRecord class: "The primary utility is provided by a class for hyb records (HybRecord), a class for fold records (FoldRecord)"
