---
name: qclash-hybrid-qc-filtering
description: Use when input is a hyb-format chimeric RNA file from miRNA/target ligation
  experiments (CLASH, qCLASH) that needs integrity checking before downstream type,
  target, or fold analyses.
when_to_use_negative:
- Input is already a filtered/annotated dataset downstream of hybkit QC (e.g. a CSV
  of fold-analysis energies) rather than raw .hyb records
- Data are not hyb-format hybrid reads (e.g. raw FASTQ or plain RNA-seq alignments),
  which hyb_check cannot parse
- Only a single .hyb file is available with no fold analysis planned — QC still applies,
  but pairing with a Vienna file via -f does not
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3512
tools:
- name: Hybkit
  role: Provides the hyb_check and hyb_filter command-line utilities used for integrity
    checking and artifact/duplicate filtering of paired hyb and Vienna files
  repo: github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API and toolkit (Python3.8+), enabling programmatic
    QC via HybFile/HybRecord iteration
  repo: https://pypi.org/project/hybkit/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:898a14079eb6d7a8d914b31fdaaaf8fb18db7570a1b2316d28ffb92bcebb13b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hybkit
  - Python3
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# qclash-hybrid-qc-filtering

## Summary

Quality-control and artifact-filtering of qCLASH/CLASH hybrid (hyb-format) datasets using hybkit's hyb_check and hyb_filter utilities, applied jointly to a .hyb file and its paired Vienna fold file.

## When to use

Use when input is a hyb-format chimeric RNA file from miRNA/target ligation experiments (CLASH, qCLASH) that needs integrity checking before downstream type, target, or fold analyses.

## When NOT to use

- Input is already a filtered/annotated dataset downstream of hybkit QC (e.g. a CSV of fold-analysis energies) rather than raw .hyb records
- Data are not hyb-format hybrid reads (e.g. raw FASTQ or plain RNA-seq alignments), which hyb_check cannot parse
- Only a single .hyb file is available with no fold analysis planned — QC still applies, but pairing with a Vienna file via -f does not

## Inputs

- hyb-format chimeric RNA file (.hyb) from CLASH/qCLASH, e.g. from GEO GSE101978
- paired predicted secondary-structure file in Vienna or CT format

## Outputs

- QC-validated hyb file (parse errors flagged by hyb_check)
- filtered hyb + Vienna file pair containing only retained hybrids
- hyb_filter log of applied filters (segment-content match, duplicate removal by HybRecord.id)

## How to apply

Run hyb_check on the .hyb file (and its associated Vienna or CT fold file) to parse records and surface format errors before any analysis, since every hybkit analysis begins with this integrity check. Then apply hyb_filter to remove artifactual ribosomal- and mitochondrial-RNA hybrids and to drop duplicate hybrids by HybRecord.id, keeping the hyb and Vienna files in sync by passing both with -i and -f. For subset retention, use string filters such as --filter any_seg_contains to keep only hybrids whose segment identifiers match a criterion (e.g. 'kshv'). Rationale: artifactual rRNA/mtRNA chimeras and duplicates would otherwise inflate type, miRNA, target, and fold-analysis counts; the filter is applied to the paired fold file so HybFoldIter can later merge FoldRecord data onto the surviving HybRecords.

## Related tools

- **Hybkit** (Provides the hyb_check and hyb_filter command-line utilities used for integrity checking and artifact/duplicate filtering of paired hyb and Vienna files) — github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API and toolkit (Python3.8+), enabling programmatic QC via HybFile/HybRecord iteration) — https://pypi.org/project/hybkit/

## Examples

```
hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv
```

## Evaluation signals

- hyb_check completes without reporting parse errors on both the .hyb and Vienna files
- Output hyb and Vienna files remain record-aligned (equal record counts and matching identifiers) after joint -i/-f filtering
- No retained record's segments match ribosomal- or mitochondrial-RNA artifacts after hyb_filter
- Record identifiers in the filtered output are unique with respect to HybRecord.id (duplicates removed)
- --filter any_seg_contains retains only records whose identifiers/segments contain the target string (e.g. 'kshv')

## Limitations

- Filters are string/property-based; artifacts not matching the configured criteria (e.g. unannotated contaminants) pass through
- Duplicate filtration by HybRecord.id removes only identifier-identical hybrids, not sequence-similar duplicates
- hybkit targets the 'hyb' file format defined by the Hyb software package; other chimera formats require conversion first

## Evidence

- [methods] Each analysis performs quality control steps on the data by checking data integrity (hyb_check): "Each analysis performs quality control steps on the data by checking data integrity (:ref:`hyb_check`)"
- [methods] removing artifactual ribosomal- and mitochondrial-RNA hybrids: "removing artifactual ribosomal- and mitochondrial-RNA hybrids (:ref:`hyb_filter`)"
- [methods] Add duplicate hybrid filtration (by HybRecord.id) options to hyb_filter: "Add duplicate hybrid filtration (by HybRecord.id) options to hyb_filter"
- [readme] $ hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv: "$ hyb_filter -i my_hyb_file.hyb -f my_hyb_file.vienna --filter any_seg_contains kshv"
- [readme] hyb_check — Parse hyb (and fold) files and check for errors: "hyb_check                           Parse hyb (and fold) files and check for errors"
