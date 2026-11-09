---
name: simulation-based-cna-caller-benchmarking
description: Use when you need to compare precision and sensitivity of single-cell
  CNA callers against simulated ground-truth events (200 kb–5 Mb single-copy losses/gains)
  on high-coverage scWGS benchmark data.
when_to_use_negative:
- The task is calling clonal, megabase-scale CNAs in low-coverage (<0.05x) scWGS cancer
  data — CHISEL and the other comparators were designed for that regime, not for high-resolution
  benchmarking of small non-clonal events
- Input is real (non-simulated) data without a ground-truth CNA event list — precision/sensitivity
  against truth cannot be computed; use qualitative comparison instead
- Only bulk WGS is available with no single-cell BAMs — none of the five single-cell
  callers can be run
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HiScanner
  role: BAF/phasing-aware caller under test; run its phase/ado/normalize/segment/cnv
    pipeline at 100 kb bins on simulated BAMs
  repo: https://github.com/parklab/HiScanner
- name: CHISEL
  role: BAF-aware comparator caller, installed from its bioconda package and run on
    the same simulated inputs
  repo: https://github.com/raphael-group/chisel
- name: Ginkgo
  role: BAF-agnostic comparator caller; its command-line version is run on the simulated
    BAMs
  repo: https://github.com/robertaboukhalil/ginkgo
- name: SCOPE
  role: BAF-agnostic R-package comparator run at 100 kb resolution with MAPQ threshold
    40
- name: SCYN
  role: BAF-agnostic comparator caller run on the same simulated inputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:aa70f1b443ec802005b74b0b04005763f93762d8237939bdea5c048f925a05a3
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# simulation-based-cna-caller-benchmarking

## Summary

Reproduce the precision/sensitivity benchmark of HiScanner versus Ginkgo, SCOPE, SCYN, and CHISEL for detecting simulated non-clonal CNAs (200 kb–5 Mb) on high-coverage (~30x) SDX single-cell WGS data, computing TP/FP/FN counts and per-size, per-type metrics from harmonized allele-specific CNA calls.

## When to use

Use when you need to compare precision and sensitivity of single-cell CNA callers against simulated ground-truth events (200 kb–5 Mb single-copy losses/gains) on high-coverage scWGS benchmark data.

## When NOT to use

- The task is calling clonal, megabase-scale CNAs in low-coverage (<0.05x) scWGS cancer data — CHISEL and the other comparators were designed for that regime, not for high-resolution benchmarking of small non-clonal events
- Input is real (non-simulated) data without a ground-truth CNA event list — precision/sensitivity against truth cannot be computed; use qualitative comparison instead
- Only bulk WGS is available with no single-cell BAMs — none of the five single-cell callers can be run

## Inputs

- Simulated SDX single-cell BAM files at ~30x coverage with ground-truth non-clonal CNAs of 200 kb–5 Mb
- Matched bulk WGS data for phasing and germline gHET calling (GATK HaplotypeCaller + SHAPEIT2)
- Simulated ground-truth CNA event list with coordinates, CNA type, and size
- Caller parameters (e.g., 100 kb bin resolution, SCOPE MAPQ threshold 40)

## Outputs

- benchmark_metrics.csv with precision and sensitivity per caller, per CNA size bin, and per CNA type
- Harmonized bed-like CNA calls (start, end, allele-specific copy-number state) for each of the five callers
- Per-size TP/FP/FN count tables matching the reported Fig. 2b-i patterns

## How to apply

Load the simulated SDX benchmark single-cell and matched bulk WGS data generated at 30x coverage, then run HiScanner's pipeline (phase, ado, normalize, segment, cnv) on the simulated BAMs at 100 kb bin resolution. Run the four comparator callers on the same simulated inputs: CHISEL (BAF-aware, via its bioconda package) and Ginkgo, SCOPE, and SCYN (all BAF-agnostic), with SCOPE processed at 100 kb resolution and MAPQ threshold 40. Harmonize all callers' calls to a common bed-like format with start, end, and allele-specific copy-number state, then match calls against ground-truth simulated events by CNA type (e.g., single-copy loss, single-copy gain) and size bin to count TP, FP, and FN. Compute precision (TP/(TP+FP)) and sensitivity (TP/(TP+FN)) per caller, size, and type, and summarize in benchmark_metrics.csv; the rationale is that read depth alone (BAF-agnostic callers) loses power at small sizes, so the BAF/phasing-aware HiScanner is expected to win consistently, especially below 1 Mb.

## Related tools

- **HiScanner** (BAF/phasing-aware caller under test; run its phase/ado/normalize/segment/cnv pipeline at 100 kb bins on simulated BAMs) — https://github.com/parklab/HiScanner
- **CHISEL** (BAF-aware comparator caller, installed from its bioconda package and run on the same simulated inputs) — https://github.com/raphael-group/chisel
- **Ginkgo** (BAF-agnostic comparator caller; its command-line version is run on the simulated BAMs) — https://github.com/robertaboukhalil/ginkgo
- **SCOPE** (BAF-agnostic R-package comparator run at 100 kb resolution with MAPQ threshold 40)
- **SCYN** (BAF-agnostic comparator caller run on the same simulated inputs)

## Examples

```
git clone https://github.com/raphael-group/chisel && cd chisel && bash install_full.sh && source conda/bin/activate chisel
```

## Evaluation signals

- HiScanner shows the most consistent caller performance across the four simulated CNA types and all size bins, outperforming all four comparators
- Sensitivity for ≥1 Mb single-copy losses reaches ~0.90 and precision ~0.96 for HiScanner
- Sensitivity drops to ~0.65 for 500 kb single-copy gains, and all callers struggle with CNAs below 500 kb
- benchmark_metrics.csv contains per-caller, per-size, per-type precision and sensitivity rows whose per-size trends reproduce the Fig. 2b-i patterns
- All five callers' outputs have been converted to a common bed-like schema with start, end, and allele-specific copy-number state before matching

## Limitations

- All callers, including HiScanner, struggle with CNAs below 500 kb
- CHISEL required parameter fine-tuning and showed a tendency for high-ploidy predictions, overfitting, and strong GC bias in its preprocessed RDR at 100 kb bins due to lack of GC normalization
- Performance varies by CNA type and size across the BAF-agnostic comparators, so single aggregate metrics can obscure type-specific differences

## Evidence

- [abstract] HiScanner consistently outperforms state-of-the-art methods across various CNA types and sizes: "In simulated data, HiScanner consistently outperforms state-of-the-art methods across various CNA types and sizes"
- [discussion] Ginkgo command-line source used in the comparison: "Ginkgo’s command line version was downloaded from https://github.com/robertaboukhalil/ginkgo"
- [discussion] SCOPE benchmark parameters: "we used the R package to process BAM files with a 100 kb resolution, MAPQ threshold of 40"
- [readme] CHISEL scope statement from its README: [evidence span withheld — anchor status: external_doc]
