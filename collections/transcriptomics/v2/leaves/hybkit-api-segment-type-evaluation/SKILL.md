---
name: hybkit-api-segment-type-evaluation
description: Use when input is hyb-format chimeric (hybrid) RNA sequence data from
  miRNA/target ligation experiments (e.g. CLASH/qCLASH) and records need segment-type
  and miRNA annotations assigned before analysis or filtering.
when_to_use_negative:
- Input data is not in the hyb file format (e.g. raw FASTQ or already-summarized CLIP
  peak tables); hybkit is specific to hyb-format hybrid records
- Records have already been evaluated and you only need to subset them (use hyb_filter
  with options like any_seg_contains instead of re-evaluating)
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3170
tools:
- name: hybkit
  role: Python3 API providing HybRecord.eval_types() and eval_mirna() methods for
    assigning segment types and miRNA identities to hybrid records
  repo: http://www.github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API (Python3.8+ required)
- name: BioPython
  role: Dependency (>= 1.79) of the hybkit package whose API design it inspired
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:9e1400f57180029dc3c68caf433bb4f0fbd21cbe8fe654a05abc3ee65539ea8b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioPython
  - Hybkit
  - Python3
  - hybkit
  - typing_extensions
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# hybkit-api-segment-type-evaluation

## Summary

Evaluate each record of a hyb-format chimeric RNA file with hybkit's HybRecord.eval_types() and eval_mirna() methods to assign segment types and miRNA identities to the hybrid segments, a prerequisite step for all downstream hybkit analyses (type/miRNA, target, grouped target, fold).

## When to use

Use when input is hyb-format chimeric (hybrid) RNA sequence data from miRNA/target ligation experiments (e.g. CLASH/qCLASH) and records need segment-type and miRNA annotations assigned before analysis or filtering.

## When NOT to use

- Input data is not in the hyb file format (e.g. raw FASTQ or already-summarized CLIP peak tables); hybkit is specific to hyb-format hybrid records
- Records have already been evaluated and you only need to subset them (use hyb_filter with options like any_seg_contains instead of re-evaluating)

## Inputs

- hyb-format chimeric RNA hybrid file (.hyb), e.g. from qCLASH of KSHV miRNAs (GEO GSE101978)
- Associated secondary-structure file in vienna or CT format (optional, for fold-aware evaluation)

## Outputs

- HybRecord objects annotated with segment types (via eval_types())
- HybRecord objects annotated with miRNA identities (via eval_mirna())
- Evaluated hyb (and fold) files consumable by hyb_filter and hyb_analyze

## How to apply

Open the hyb file (and associated vienna/fold file) with hybkit's HybFile/HybFoldIter iterators and iterate over HybRecord objects, calling hyb_record.eval_types() on each record to assign segment types based on custom criteria, then eval_mirna() to identify miRNA-containing records (note: these methods were renamed from type_analysis and mirna_analysis in earlier hybkit versions). The equivalent command-line step is the hyb_eval utility, which evaluates hyb (and fold) records to identify/assign segment types and miRNAs using custom criteria. Evaluation must occur before filtering (e.g. removing artifactual ribosomal- and mitochondrial-RNA hybrids with hyb_filter) and before analyses such as hyb_analyze, because downstream type/miRNA and target analyses depend on the assigned annotations. Evaluation results can then be consumed by hybkit.plot for tabular (CSV) and graphical outputs.

## Related tools

- **hybkit** (Python3 API providing HybRecord.eval_types() and eval_mirna() methods for assigning segment types and miRNA identities to hybrid records) — http://www.github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API (Python3.8+ required))
- **BioPython** (Dependency (>= 1.79) of the hybkit package whose API design it inspired)

## Examples

```
import hybkit
with hybkit.HybFile.open(in_file, 'r') as hyb_file:
    for hyb_record in hyb_file:
        hyb_record.eval_types()
        hyb_record.eval_mirna()
```

## Evaluation signals

- Each HybRecord in the output carries a segment-type annotation after eval_types() and a miRNA identity after eval_mirna()
- Downstream hyb_filter/hyb_analyze steps run without errors on the evaluated records, confirming annotation integrity
- Type/miRNA analysis summary CSV tables (e.g. combined_analysis_type_hybrid_types.csv) are produced and readable
- Summary counts match the values reported by the corresponding example analysis in the Hybkit documentation for the GSE101978 qCLASH dataset

## Limitations

- Segment-type assignment depends on custom criteria; hybkit renamed these methods (type_analysis/mirna_analysis to eval_types/eval_mirna), so older example scripts may not run unchanged on recent versions
- Artifactual ribosomal- and mitochondrial-RNA hybrids remain in the data after evaluation and must be removed in a separate hyb_filter step

## Evidence

- [readme] Record attributes can be analyzed, set, and evaluated using included class methods.: "The primary utility is provided by a class for hyb records (HybRecord), a class for fold records (FoldRecord), and file-iterator classes (HybFile, ViennaFile, CTFile, HybFoldIter)."
- [methods] eval_types assignment of segment types: "# Analyze each record to assign segment types
hyb_record.eval_types()"
- [methods] method renaming to eval_types/eval_mirna: "Renamed HybRecord type_analysis and mirna_analysis to eval_types and eval_mirna, respectively"
- [readme] hyb_eval CLI utility description: "hyb_eval                            Evaluate hyb (and fold) records to identify / assign
                                            segment types and miRNAs using custom criteria"
- [abstract] purpose of hybkit evaluation: "analysis of hybrid sequence data in the "hyb" file format to enable customizable evaluation and annotation of hybrid characteristics"
