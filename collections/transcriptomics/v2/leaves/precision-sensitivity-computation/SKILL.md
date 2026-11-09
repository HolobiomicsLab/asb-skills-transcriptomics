---
name: precision-sensitivity-computation
description: Use when benchmarking CNA callers (HiScanner, CHISEL, Ginkgo, SCOPE,
  SCYN) on simulated scWGS/bulk WGS data with known ground-truth CNAs and you need
  per-type, per-size precision and sensitivity curves.
when_to_use_negative:
- Input data has no ground-truth CNA annotations (e.g., real patient tumors without
  simulated truth events) — precision/sensitivity cannot be computed, only orthogonal
  validation or FDR estimates
- Calls are clonal, megabase-scale events in low-coverage data — the benchmark targets
  non-clonal CNAs of 200 kb–5 Mb and pooled metrics would be uninformative
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: HiScanner
  role: Primary caller under benchmark; run pipeline (phase, ado, normalize, segment,
    cnv) at 100 kb resolution on simulated BAMs before metric computation
  repo: https://github.com/parklab/HiScanner
- name: CHISEL
  role: BAF-aware comparator caller; bioconda package run on identical simulated inputs
  repo: https://github.com/raphael-group/chisel
- name: Ginkgo
  role: BAF-agnostic comparator caller; command-line version used on simulated data
  repo: https://github.com/robertaboukhalil/ginkgo
- name: SCOPE
  role: BAF-agnostic comparator (R package) run at 100 kb resolution with MAPQ threshold
    40
- name: SCYN
  role: BAF-agnostic comparator caller run on the same simulated inputs
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:1204825757226d21fda2f0923d63f3268ce00e994415daa5cc5cb4d62820e255
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - CHISEL
  - Ginkgo
  - HiScanner
  - SCOPE
  - SCYN
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# precision-sensitivity-computation

## Summary

Computes per-caller precision (TP/(TP+FP)) and sensitivity (TP/(TP+FN)) for single-cell CNA callers by matching called CNV segments against simulated truth events (SDX), stratified by CNA type (single-copy loss/gain) and size class (200 kb–5 Mb), reproducing the HiScanner benchmark of Nature Communications 10.1038/s41467-025-60446-5.

## When to use

Use when benchmarking CNA callers (HiScanner, CHISEL, Ginkgo, SCOPE, SCYN) on simulated scWGS/bulk WGS data with known ground-truth CNAs and you need per-type, per-size precision and sensitivity curves.

## When NOT to use

- Input data has no ground-truth CNA annotations (e.g., real patient tumors without simulated truth events) — precision/sensitivity cannot be computed, only orthogonal validation or FDR estimates
- Calls are clonal, megabase-scale events in low-coverage data — the benchmark targets non-clonal CNAs of 200 kb–5 Mb and pooled metrics would be uninformative

## Inputs

- SDX-simulated single-cell WGS BAM files at 30x coverage with ground-truth CNA annotations
- SDX-simulated paired bulk WGS data (for phasing/gHET support)
- Harmonized CNA calls per caller in bed-like format with start, end, and allele-specific copy-number state

## Outputs

- benchmark_metrics.csv with precision and sensitivity per caller, per CNA type, and per size bin
- Per-condition (coverage x bin size) precision/sensitivity summary tables for sensitivity analysis

## How to apply

First harmonize every caller's output to a common bed-like format with start, end, and allele-specific copy-number state; HiScanner calls come from its phase/ado/normalize/segment/cnv pipeline (100 kb bins, GC-corrected read depth, joint BIC segmentation), CHISEL from its bioconda package, Ginkgo from its command-line version, and SCOPE with 100 kb resolution and MAPQ threshold 40. Match harmonized calls against the SDX-simulated ground-truth events by overlap, counting true positives, false positives, and false negatives per CNA type (single-copy loss, single-copy gain, other simulated classes) and per size bin (200 kb–5 Mb). Compute precision = TP/(TP+FP) and sensitivity = TP/(TP+FN) per caller, per size, and per type, then assemble into a summary table (benchmark_metrics.csv). Rationale: calls must be size- and type-stratified because all callers degrade on CNAs below 500 kb and performance varies by variant type, so pooled metrics mask the differences the benchmark is designed to detect.

## Related tools

- **HiScanner** (Primary caller under benchmark; run pipeline (phase, ado, normalize, segment, cnv) at 100 kb resolution on simulated BAMs before metric computation) — https://github.com/parklab/HiScanner
- **CHISEL** (BAF-aware comparator caller; bioconda package run on identical simulated inputs) — https://github.com/raphael-group/chisel
- **Ginkgo** (BAF-agnostic comparator caller; command-line version used on simulated data) — https://github.com/robertaboukhalil/ginkgo
- **SCOPE** (BAF-agnostic comparator (R package) run at 100 kb resolution with MAPQ threshold 40)
- **SCYN** (BAF-agnostic comparator caller run on the same simulated inputs)

## Examples

```
hiscanner run --step all
```

## Evaluation signals

- Every caller's output harmonizes to the same schema: start, end, and allele-specific copy-number state, with no unconverted caller-specific states remaining
- TP + FN per size bin equals the number of simulated truth events in that bin (no truth events dropped by the matching step)
- Expected reference values reproduce Fig. 2b-i: HiScanner sensitivity ~0.90 / precision ~0.96 for >=1 Mb single-copy losses, dropping to ~0.65 sensitivity for 500 kb single-copy gains
- All callers show reduced sensitivity for CNAs below 500 kb, consistent with the reported benchmark trend
- HiScanner metrics exceed the four comparators across CNA types and sizes, reproducing the reported consistent outperformance

## Limitations

- All callers struggled with CNAs below 500 kb, so sub-500 kb precision/sensitivity estimates are noisy and the FDR estimate is conservative since FDR increases as CNA size decreases
- CHISEL required parameter fine-tuning and showed high-ploidy predictions, overfitting, and GC bias at 100 kb bins, which affects fair comparison
- Simulated SDX conditions may not capture real amplification-artifact profiles; findings are grounded in simulation, not validated on real tumor data

## Evidence

- [abstract] In simulated data, HiScanner consistently outperforms state-of-the-art methods across various CNA types and sizes: "In simulated data, HiScanner consistently outperforms state-of-the-art methods across various CNA types and sizes"
- [intro] HiScanner outperforms four representative single-cell CNA methods—CHISEL (BAF-aware), Ginkgo, SCOPE, and SCYN (all BAF-agnostic methods): "HiScanner outperforms four representative single-cell CNA methods—CHISEL (BAF-aware), Ginkgo, SCOPE, and SCYN (all BAF-agnostic methods)"
- [discussion] SCOPE processing parameters used in the benchmark comparison: "we used the R package to process BAM files with a 100 kb resolution, MAPQ threshold of 40"
- [other] Metric definitions from the source card workflow: [evidence span withheld — anchor status: not_found]
