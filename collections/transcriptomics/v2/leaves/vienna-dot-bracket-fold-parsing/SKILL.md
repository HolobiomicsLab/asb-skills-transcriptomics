---
name: vienna-dot-bracket-fold-parsing
description: Use when a CLASH/qCLASH workflow requires reading predicted secondary-structure
  data in Vienna (dot-bracket) or CT format, or pairing each fold record with its
  corresponding hyb record (e.g., to detect hyb/Vienna sequence mismatches before
  analysis).
when_to_use_negative:
- Input is a plain hyb file with no associated Vienna/CT fold data and only type/miRNA
  or target analyses are needed — the hyb-only HybFile iterator suffices.
- Data are raw sequencing reads (FASTQ) or other non-hybrid formats — hybkit is specific
  to hyb-format chimeric RNA data from RNA proximity-ligation methods.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: hybkit
  role: Python3 API providing ViennaFile, CTFile, FoldRecord, and HybFoldIter for
    dot-bracket fold parsing and hyb/fold pairing
  repo: http://www.github.com/RenneLab/hybkit
- name: hyb_check
  role: Command-line utility that parses hyb and fold files and checks for errors,
    consuming the Vienna/fold parsing capability
  repo: http://www.github.com/RenneLab/hybkit
- name: Python3
  role: Runtime for the hybkit API (Python 3.8+ required)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad721
    title: 'Hybkit: a Python API and command-line toolkit for hybrid sequence data
      from chimeric RNA methods'
  source_content_hash: sha256:f91943cf49098c0a23707a992911c5ed09efffa3e6aedf24f5c66ff8727bdbd4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python3
  - hyb_check
  - hybkit
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# vienna-dot-bracket-fold-parsing

## Summary

Parses Vienna-format files line-by-line into FoldRecord objects carrying dot-bracket secondary-structure data using the hybkit API (ViennaFile), enabling lockstep iteration with hyb records via HybFoldIter for validation and fold-aware analysis of chimeric RNA hybrid data.

## When to use

Use when a CLASH/qCLASH workflow requires reading predicted secondary-structure data in Vienna (dot-bracket) or CT format, or pairing each fold record with its corresponding hyb record (e.g., to detect hyb/Vienna sequence mismatches before analysis).

## When NOT to use

- Input is a plain hyb file with no associated Vienna/CT fold data and only type/miRNA or target analyses are needed — the hyb-only HybFile iterator suffices.
- Data are raw sequencing reads (FASTQ) or other non-hybrid formats — hybkit is specific to hyb-format chimeric RNA data from RNA proximity-ligation methods.

## Inputs

- Vienna-format file (.vienna) with one FoldRecord per line containing dot-bracket secondary structure, paired to a .hyb file
- CT-format fold file (via hybkit.CTFile)
- Corresponding hyb-format chimeric RNA file (15-or-16 tab-delimited columns) for paired HybFoldIter iteration

## Outputs

- FoldRecord objects exposing dot-bracket secondary-structure strings
- HybRecord objects enriched with merged fold data (via HybFoldIter)
- Validation report (e.g., hyb_check_report.txt) listing per-record format errors and hyb/fold sequence mismatches

## How to apply

Open the Vienna file with hybkit.ViennaFile, which returns each line as a FoldRecord containing the dot-bracket secondary-structure annotation; the same API exposes CTFile for CT-format folds. If a paired hyb file is present, iterate both files in lockstep with hybkit.HybFoldIter, which merges FoldRecord data into the corresponding HybRecord and checks for errors/mismatches between the two files' sequences and identifiers. This is the mechanism behind the hyb_check quality-control step, which collects format errors and hyb/fold sequence mismatches into a single validation report. Fold-aware records can then feed downstream utilities (hyb_eval, hyb_filter, hyb_analyze fold analysis). Rationale: hybkit intentionally integrates predicted secondary-structure information into both the API and command-line utilities, so fold parsing is the entry point for all fold analyses on hybrid RNA data.

## Related tools

- **hybkit** (Python3 API providing ViennaFile, CTFile, FoldRecord, and HybFoldIter for dot-bracket fold parsing and hyb/fold pairing) — http://www.github.com/RenneLab/hybkit
- **hyb_check** (Command-line utility that parses hyb and fold files and checks for errors, consuming the Vienna/fold parsing capability) — http://www.github.com/RenneLab/hybkit
- **Python3** (Runtime for the hybkit API (Python 3.8+ required))

## Examples

```
import hybkit
with hybkit.ViennaFile.open('my_hyb_file.vienna', 'r') as vf:
    for fold_record in vf:
        print(fold_record.id)
```

## Evaluation signals

- A clean input yields zero reported errors: every record passes hyb_check's parse and hyb/fold mismatch checks.
- HybFoldIter confirms record identifiers match between paired hyb and fold entries; no sequence mismatches are flagged.
- Each parsed FoldRecord yields a dot-bracket structure string of the same sequence length as its hybrid sequence.
- Fold-parsed records retain valid column structure when written to downstream hyb_eval/hyb_filter/hyb_analyze steps.
- The hyb_check validation report is produced with per-record error details and summary counts.

## Limitations

- hybkit expects hyb files in the 15-or-16 tab-delimited column format defined by the Hyb software package; malformed columns or disallowed header/comment lines raise format errors.
- Paired iteration requires hyb and Vienna files to be in the same record order with matching identifiers; mismatches are treated as validation failures rather than silently resynchronized.
- Secondary-structure data must be predicted content (Vienna/CT format); hybkit does not perform structure prediction itself.

## Evidence

- [readme] hyb_check: "hyb_check                           Parse hyb (and fold) files and check for errors"
- [readme] api_classes: "class for fold records (FoldRecord), and file-iterator classes (HybFile, ViennaFile, CTFile, HybFoldIter)."
- [readme] fold_integration: "Integrated analysis of predicted secondary structure (fold) information for the API and command-line utilities."
- [methods] qc_step: "Each analysis performs quality control steps on the data by checking data integrity"
- [readme] cli_usage: "These scripts are used on the command line with hyb (and associated "vienna" or "CT") files."
