---
name: ribosomal-mitochondrial-artifact-filtration
description: Use when a hyb-format qCLASH/CLASH file (with optional paired vienna/CT
  fold file) contains artifactual rRNA- and mtRNA-derived hybrid records that would
  contaminate type/miRNA or target analyses.
when_to_use_negative:
- The input is not hyb-format chimeric/hybrid RNA data (e.g. raw FASTQ or an already-summarized
  count table) — hyb_filter operates on hyb (and fold) records.
- The analysis goal is to study rRNA- or mtRNA-derived chimeras themselves; this step
  deliberately discards those records as artifactual.
- The hyb file has not passed hyb_check integrity validation, so filtration totals
  could not be reconciled against verified record counts.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: hybkit
  role: Toolkit providing the hyb_filter command-line utility and hybkit API used
    to remove artifactual ribosomal- and mitochondrial-RNA hybrids from hyb files
  repo: github.com/RenneLab/hybkit
- name: hyb_filter
  role: Command-line utility that filters hyb (and fold) records to a specific custom
    subset, including removal of artifactual rRNA/mtRNA hybrids
  repo: github.com/RenneLab/hybkit
- name: hyb_check
  role: Used immediately before filtration to verify hyb (and fold) file data integrity
  repo: github.com/RenneLab/hybkit
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:a3b2067340ebc637453831c247213be06a6bbf855ad18f421b0efd1d3c80abc9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioPython
  - Python3
  - hyb_check
  - hyb_filter
  - hybkit
  - typing_extensions
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Ribosomal-mitochondrial artifact filtration

## Summary

A quality-control step in the Hybkit qCLASH workflow that removes artifactual ribosomal-RNA and mitochondrial-RNA hybrids from hyb-format files using hyb_filter, so that downstream type/miRNA, target, and grouped-target analyses count only genuine miRNA-target chimeras.

## When to use

Use when a hyb-format qCLASH/CLASH file (with optional paired vienna/CT fold file) contains artifactual rRNA- and mtRNA-derived hybrid records that would contaminate type/miRNA or target analyses.

## When NOT to use

- The input is not hyb-format chimeric/hybrid RNA data (e.g. raw FASTQ or an already-summarized count table) — hyb_filter operates on hyb (and fold) records.
- The analysis goal is to study rRNA- or mtRNA-derived chimeras themselves; this step deliberately discards those records as artifactual.
- The hyb file has not passed hyb_check integrity validation, so filtration totals could not be reconciled against verified record counts.

## Inputs

- hyb-format hybrid RNA file (e.g. from GEO GSE101978 qCLASH replicates)
- Optional paired fold file in vienna or CT format associated with the hyb file

## Outputs

- Filtered .hyb file containing only non-ribosomal, non-mitochondrial hybrid records
- Filtered paired vienna/CT fold file (when a fold file is supplied)
- Record counts before/after filtration, which must reconcile with downstream type/miRNA count tables

## How to apply

After parsing and integrity-checking each .hyb file with hyb_check (confirming the 15-16 tab-delimited columns per record), run hyb_filter to drop hybrids whose segments are ribosomal- or mitochondrial-RNA, which the Hybkit authors identify as ligation artifacts. If a paired fold file (.vienna or CT) is present, pass it with -f so the hyb and fold records remain synchronized through the same filter. Apply this filtration before HybRecord.eval_types()/eval_mirna() annotation and before tallying with hyb_analyze or hybkit.analysis.Analysis, because the type/miRNA and grouped-target example analyses (tasks over GSE101978 replicates) expect counts over the filtered record set. The duplicate-hybrid removal option (by HybRecord.id) available in hyb_filter can be combined in the same pass if duplicates are a concern.

## Related tools

- **hybkit** (Toolkit providing the hyb_filter command-line utility and hybkit API used to remove artifactual ribosomal- and mitochondrial-RNA hybrids from hyb files) — github.com/RenneLab/hybkit
- **hyb_filter** (Command-line utility that filters hyb (and fold) records to a specific custom subset, including removal of artifactual rRNA/mtRNA hybrids) — github.com/RenneLab/hybkit
- **hyb_check** (Used immediately before filtration to verify hyb (and fold) file data integrity) — github.com/RenneLab/hybkit

## Examples

```
hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv
```

## Evaluation signals

- The number of records in the filtered .hyb file equals the total of the downstream type/miRNA count tables for that replicate (workflow validation: 'totals match the number of filtered hybrid records').
- Any paired vienna/CT fold file contains exactly the same set of record identifiers as the filtered .hyb file (hyb_filter was invoked with -f on both).
- No remaining record has a segment annotated as ribosomal or mitochondrial RNA after eval_types().
- Filtered per-replicate outputs reproduce the segment-type categories of the example_01 outputs shipped in the hybkit repository.

## Limitations

- The documented scope is removal of 'artifactual ribosomal- and mitochondrial-RNA hybrids'; the provided context gives no numeric thresholds or custom criteria for other artifact classes.
- If a paired fold file is not passed alongside the hyb file, the hyb and fold records can fall out of register for subsequent fold analysis.
- This filtration is replicate-level preprocessing; it does not itself annotate segment types — eval_types()/eval_mirna() must still be run afterwards.

## Evidence

- [methods] removing artifactual ribosomal- and mitochondrial-RNA hybrids: "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
- [methods] hyb_filter example command: "$ hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv"
- [readme] hyb_filter description: "Filter hyb (and fold) records to a specific"
- [methods] quality control before filtering: "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
