---
name: mirna-target-count-tabulation
description: Use when hyb-format hybrid (chimeric miRNA/target ligation) records must
  be annotated by segment type and miRNA, then filtered to one miRNA of interest and
  its targets counted and tabulated per replicate.
when_to_use_negative:
- Data are hybrid records pooled across multiple experimental replicates — use Hybkit's
  Grouped Target Analysis instead of single-replicate Target Analysis
- The goal is to analyze predicted miRNA folding patterns in miRNA-containing hybrids
  — use Fold Analysis, not target counting
- Input is not in hyb format (nor associated vienna/CT fold format) and cannot be
  converted to hyb records
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: hybkit
  role: Python API (HybFile, HybRecord.eval_types/eval_mirna, Analysis class) and
    command-line utilities (hyb_check, hyb_filter, hyb_analyze) used for QC, filtering,
    miRNA annotation, target counting, and plotting
  repo: github.com/RenneLab/hybkit
- name: Python3
  role: Runtime (Python 3.8+) for the hybkit API workflow and record iteration
- name: matplotlib
  role: Plotting backend (>= 3.7.1) used by hybkit.plot to render target distribution
    figures
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:3b889219b83b9ab3f1f5b891629eb40fc7ab8980f8eac5d3abdd88eb09cd33b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python3
  - hybkit
  - matplotlib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mirna-target-count-tabulation

## Summary

Count and characterize the hybrid-record targets of a specific miRNA (e.g., KSHV kshv-miR-k12-5) from a single experimental replicate of hyb-format qCLASH/CLASH data using Hybkit's Target Analysis, returning CSV count tables and distribution plots.

## When to use

Use when hyb-format hybrid (chimeric miRNA/target ligation) records must be annotated by segment type and miRNA, then filtered to one miRNA of interest and its targets counted and tabulated per replicate.

## When NOT to use

- Data are hybrid records pooled across multiple experimental replicates — use Hybkit's Grouped Target Analysis instead of single-replicate Target Analysis
- The goal is to analyze predicted miRNA folding patterns in miRNA-containing hybrids — use Fold Analysis, not target counting
- Input is not in hyb format (nor associated vienna/CT fold format) and cannot be converted to hyb records

## Inputs

- hyb-format hybrid sequence file (e.g., qCLASH hybrids from GEO GSE101978)
- Optional associated fold data in vienna (or CT) format
- Segment-type / miRNA annotation criteria (hybkit type_finder criteria)
- Identifier string of the miRNA of interest (e.g., 'kshv' for kshv-miR-k12-5)

## Outputs

- CSV miRNA/target count tables
- PNG target distribution plots (via hybkit.plot / matplotlib)
- Type/miRNA-annotated HybRecord subset (miRNA-containing hybrids)

## How to apply

Load the hyb file with hybkit's HybFile iterator (e.g., qCLASH data from GEO accession GSE101978), first running hyb_check-style integrity QC and using hyb_filter to remove artifactual ribosomal- and mitochondrial-RNA hybrids (optionally also duplicate filtration by HybRecord.id). Annotate every record with HybRecord.eval_types() and HybRecord.eval_mirna() so type_finder criteria assign segment types (e.g., miRNA, lncRNA) and miRNA identities; a miRNA-containing hybrid is identified via properties such as has_prop('any_seg_type_contains', ...). Select the subset of miRNA-containing hybrids for the miRNA of interest — for example, filtering any segment whose identifier contains 'kshv' (hyb_filter --filter any_seg_contains kshv) to isolate kshv-miR-k12-5 hybrids. Run Hybkit's Target Analysis (Analysis class via the Python API, or the hyb_analyze target command-line mode), which is designed to 'analyze targets of a set of miRNAs from a single experimental replicate'. Export the miRNA/target counts as CSV tables and render the target distribution plots with hybkit.plot (requires matplotlib >= 3.7.1). The rationale is that miRNA/target ligation methods yield chimeric sequences whose segment annotation is the prerequisite for meaningful per-miRNA target counting.

## Related tools

- **hybkit** (Python API (HybFile, HybRecord.eval_types/eval_mirna, Analysis class) and command-line utilities (hyb_check, hyb_filter, hyb_analyze) used for QC, filtering, miRNA annotation, target counting, and plotting) — github.com/RenneLab/hybkit
- **Python3** (Runtime (Python 3.8+) for the hybkit API workflow and record iteration)
- **matplotlib** (Plotting backend (>= 3.7.1) used by hybkit.plot to render target distribution figures)

## Examples

```
hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv
```

## Evaluation signals

- The exported CSV count tables exist and are parseable as tabular data
- The PNG target distribution plot exists and renders a readable graph
- Per-miRNA target counts are internally consistent with the number of miRNA-containing records retained after hyb_check/hyb_filter QC
- Each analyzed record has segment types assigned (eval_types) and miRNA identity assigned (eval_mirna) before counting
- kshv-miR-k12-5 selection is confirmed by the presence of the 'kshv' identifier string in the retained hybrids

## Limitations

- Target Analysis operates on a single experimental replicate; pooled replicates require the separate Grouped Target Analysis mode
- Accurate counting depends on prior type/miRNA annotation via type_finder criteria; records lacking recognizable segment types or miRNA identities will be missed
- Artifactual ribosomal- and mitochondrial-RNA hybrids must be removed beforehand or they will inflate counts
- Plot rendering depends on matplotlib >= 3.7.1 (and BioPython >= 1.79, typing_extensions >= 4.8.0 dependencies)

## Evidence

- [methods] removing artifactual ribosomal- and mitochondrial-RNA hybrids: "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
- [methods] Analyze targets of a set of miRNAs from a single experimental replicate: "Target Analysis               Analyze targets of a set of miRNAs from a single
                                      experimental replicate"
- [methods] eval_types and eval_mirna record evaluation methods: "Renamed HybRecord type_analysis and mirna_analysis to eval_types and eval_mirna, respectively"
- [readme] Example qCLASH analyses in scripts and API: "Example analyses for publicly available qCLASH hybrid sequence data implemented in each of the command-line scripts and hybkit Python API."
- [readme] Analyses return tabular and graph outputs: "These analyses provide analysis results in both tabular and graph form."
