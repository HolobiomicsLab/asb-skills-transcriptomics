---
name: simulation-based-method-evaluation
description: Use when evaluating a CNA caller's sensitivity/precision across variant
  sizes and coverages, or choosing bin size and depth for scWGS analysis using simulated
  data with known truth events.
when_to_use_negative:
- Input data are real biological samples without simulated truth events — precision/sensitivity
  cannot be computed without ground truth
- The analysis goal is CNA discovery in a new dataset rather than method benchmarking
  or parameter selection
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: HiScanner
  role: Primary CNA caller benchmarked across bin-size and depth grid; run via 'hiscanner
    run --step all' on simulated BAMs
  repo: github.com/parklab/HiScanner
- name: CHISEL
  role: BAF-aware comparator single-cell CNA caller run on identical simulated inputs
  repo: github.com/raphael-group/chisel
- name: Ginkgo
  role: BAF-agnostic comparator single-cell CNA caller run on identical simulated
    inputs
  repo: github.com/robertaboukhalil/ginkgo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:07a7df61777aa4e46b46642c73f083903b4e7582335acb80d5f8beed4d5ca9eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHISEL
  - Ginkgo
  - HiScanner
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# simulation-based-method-evaluation

## Summary

Benchmark a single-cell CNA caller by grid-searching bin size (50/100/500 kb) and sequencing depth (0.5×/10×/30×) on simulated scWGS data, matching called segments against truth events to compute precision and sensitivity per CNA type and size class. Used to identify the parameter combination with the best precision-sensitivity trade-off, which for HiScanner was 100 kb bins at 30×.

## When to use

Use when evaluating a CNA caller's sensitivity/precision across variant sizes and coverages, or choosing bin size and depth for scWGS analysis using simulated data with known truth events.

## When NOT to use

- Input data are real biological samples without simulated truth events — precision/sensitivity cannot be computed without ground truth
- The analysis goal is CNA discovery in a new dataset rather than method benchmarking or parameter selection

## Inputs

- SDX-simulated single-cell scWGS BAM files with known truth CNV events
- Paired bulk scWGS BAM files from the simulations
- Downsampled simulation BAMs at 10× and 0.5× coverage
- HiScanner config.yaml and metadata table with bamID, bam, singlecell columns
- Simulated truth-event segment coordinates for precision/sensitivity matching

## Outputs

- Per-condition (coverage × bin size) precision and sensitivity values per CNA type and size class
- Summary tables of precision/sensitivity across the parameter grid
- Identified optimal parameter setting (30× coverage, 100 kb bins)

## How to apply

Initialize a HiScanner project ('hiscanner init --output ./my_project'), populate config.yaml and the metadata table (bamID, bam, singlecell columns), and validate with 'hiscanner validate'. Run the full pipeline ('hiscanner run --step all', which executes GC-corrected read-depth normalization, joint BIC-based segmentation, and CNV calling) on SDX-simulated single-cell and paired bulk BAMs at 30× across the 50/100/500 kb bin grid, then repeat on 10× and 0.5× downsampled simulations. Run comparator callers (CHISEL, a BAF-aware method; Ginkgo, a BAF-agnostic method) on identical simulated inputs. Match called CNV segments against the simulated truth events and compute precision and sensitivity per CNA type and size class for each coverage × bin-size condition. Select the setting maximizing the precision-sensitivity balance — 100 kb bins at 30× — while noting that 50 kb bins improve sub-500 kb sensitivity but reduce precision through noise, 500 kb bins perform worse for small variants, and 0.5× depth substantially degrades single-copy and small-variant detection.

## Related tools

- **HiScanner** (Primary CNA caller benchmarked across bin-size and depth grid; run via 'hiscanner run --step all' on simulated BAMs) — github.com/parklab/HiScanner
- **CHISEL** (BAF-aware comparator single-cell CNA caller run on identical simulated inputs) — github.com/raphael-group/chisel
- **Ginkgo** (BAF-agnostic comparator single-cell CNA caller run on identical simulated inputs) — github.com/robertaboukhalil/ginkgo

## Examples

```
hiscanner init --output ./my_project && hiscanner validate && hiscanner run --step all
```

## Evaluation signals

- The optimal condition from the grid matches the reported result: 30× coverage with 100 kb bins, consistent with Supplementary Fig. 3
- 'hiscanner validate' passes before pipeline runs, confirming config.yaml and metadata table (bamID, bam, singlecell) are populated correctly
- Called segments match simulated truth events only within the matched condition, with precision/sensitivity computed per CNA type and size class
- 50 kb bins show improved sub-500 kb sensitivity but reduced precision relative to 100 kb, and 500 kb bins show worse small-variant performance — reproducing the expected trade-off pattern
- 0.5× coverage runs show substantially lower performance, particularly for single-copy events and smaller variants, relative to 10× and 30×

## Limitations

- Coverage down to 0.5× substantially degrades performance, particularly for single-copy events and smaller variants, so conclusions do not extend below tested depths
- Simulation results (SDX) may not fully capture real-data artifacts such as amplification noise that confounds CNA detection
- The optimum is assay-specific; HiScanner also considers assay-specific ADO patterns to find an optimal bin size, so the 100 kb/30× result may not transfer to other assays

## Evidence

- [results] Optimal bin size and depth: [evidence span withheld — anchor status: not_found]
- [results] Bin-size trade-offs: [evidence span withheld — anchor status: not_found]
- [results] Depth effect: [evidence span withheld — anchor status: not_found]
- [intro] HiScanner algorithm: "HiScanner performs read depth normalization at nucleotide resolution, followed by joint segmentation of single-cell profiles using the Bayesian information criterion (BIC)"
- [readme] README demo workflow: "We provide a demo dataset and tutorial to help you get started. After installation, see https://github.com/parklab/hiscanner_demo for instructions."
- [intro] ADO-based bin selection: "It also considers assay-specific ADO patterns to find an optimal bin size, a nuanced yet crucial parameter in CNA analysis"
