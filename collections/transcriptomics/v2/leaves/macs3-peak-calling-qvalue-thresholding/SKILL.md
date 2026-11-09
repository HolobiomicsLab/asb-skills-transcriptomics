---
name: macs3-peak-calling-qvalue-thresholding
description: Use when constructing a trustworthy feature space (peaks and non-peaks)
  from a scATAC-seq BAM file, or when peaks of differing confidence are needed for
  count-matrix construction or simulator training.
when_to_use_negative:
- When the goal is scRNA-seq feature definition — this skill applies only to ATAC-seq
  open-chromatin peaks; scRNA-seq uses gene/inter-gene features via Utility.scRNA_CreateFeatureSets
  instead.
- When only a single peak confidence level is needed and the paired stringent/relaxed
  complement design is not required — a single MACS3 call at one threshold suffices.
- When input is already a peak BED or feature table — peak calling has already been
  done; proceed directly to count matrix construction.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: MACS3
  role: Peak caller invoked (directly or via scReadSim's Utility.CallPeak) to call
    scATAC-seq peaks at q-value 0.01 and 0.1 thresholds
  repo: https://github.com/macs3-project/MACS
- name: scReadSim
  role: Wraps MACS3 through Utility.CallPeak and provides Utility.ComplementFeature
    to derive non-peak background features from the relaxed-threshold complement
  repo: https://github.com/JSB-UCLA/scReadSim
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:f5de85a7fdb290726d3d54e23efb504dd584959c8cba4d20c8c1f384f3fdcf9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/macs3-project/MACS
  tools:
  - MACS3
  - SCAN-ATAC-Sim
  - scReadSim
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# macs3-peak-calling-qvalue-thresholding

## Summary

Call scATAC-seq peaks with MACS3 at two q-value thresholds — 0.01 for trustworthy peaks and 0.1 for the relaxed set whose complement defines trustworthy non-peaks — as done in the scReadSim sci-ATAC-seq benchmarking workflow.

## When to use

Use when constructing a trustworthy feature space (peaks and non-peaks) from a scATAC-seq BAM file, or when peaks of differing confidence are needed for count-matrix construction or simulator training.

## When NOT to use

- When the goal is scRNA-seq feature definition — this skill applies only to ATAC-seq open-chromatin peaks; scRNA-seq uses gene/inter-gene features via Utility.scRNA_CreateFeatureSets instead.
- When only a single peak confidence level is needed and the paired stringent/relaxed complement design is not required — a single MACS3 call at one threshold suffices.
- When input is already a peak BED or feature table — peak calling has already been done; proceed directly to count matrix construction.

## Inputs

- scATAC-seq BAM file (e.g., sci-ATAC-seq reads, mm10 reference genome)
- chromosome size file (embedded in scReadSim for mm10)
- q-value thresholds: 0.01 (stringent) and 0.1 (relaxed)

## Outputs

- Trustworthy peak list (MACS3 peaks called at q-value 0.01)
- Trustworthy non-peak regions (complement of MACS3 peaks called at q-value 0.1)
- Feature sets usable for peak-by-cell count matrix construction

## How to apply

Run MACS3 on the real scATAC-seq BAM file twice: once at the stringent q-value threshold 0.01 to obtain trustworthy peaks, and once at the relaxed threshold 0.1; the complement of the relaxed-threshold peaks defines trustworthy non-peaks (e.g., via scReadSim's Utility.ComplementFeature). scReadSim invokes MACS3 through its Utility.CallPeak function. These trustworthy peaks become foreground features for the peak-by-cell count matrix (built with Utility.scATAC_bam2countmat) on which scDesign2 is trained, and serve as the feature space for counting synthetic reads from scReadSim or SCAN-ATAC-Sim output (BAM or BED). The dual-threshold design ensures that peaks used for comparison and training are high-confidence while non-peaks are confidently background.

## Related tools

- **MACS3** (Peak caller invoked (directly or via scReadSim's Utility.CallPeak) to call scATAC-seq peaks at q-value 0.01 and 0.1 thresholds) — https://github.com/macs3-project/MACS
- **scReadSim** (Wraps MACS3 through Utility.CallPeak and provides Utility.ComplementFeature to derive non-peak background features from the relaxed-threshold complement) — https://github.com/JSB-UCLA/scReadSim

## Examples

```
macs3 callpeak -t real_scATAC.bam -f BAMPE -q 0.01 -n trustworthy_peaks && macs3 callpeak -t real_scATAC.bam -f BAMPE -q 0.1 -n relaxed_peaks
```

## Evaluation signals

- MACS3 runs complete at both q-value 0.01 and 0.1 without errors and produce peak BED/narrowPeak outputs.
- The stringent (0.01) peak set is a subset of the relaxed (0.1) peak set, since a lower q-value threshold is stricter.
- Trustworthy non-peaks derived via Utility.ComplementFeature are disjoint from the relaxed-threshold peaks and together cover the feature space.
- Counting real reads over the 0.01-threshold peaks yields a peak-by-cell matrix whose summary statistics (library size, zero proportion) are plausible for downstream scDesign2 training.
- Downstream KS tests of summary statistics and miLISI (~closer to 2) confirm synthetic data generated on this feature space mimics the real matrix.

## Limitations

- MACS3 is a top peak caller per the paper's benchmark, but HMMRATAC performs comparably; the dual-threshold scheme was not validated with other callers.
- Peak quality depends on the input BAM and reference genome (e.g., mm10 with the embedded chromosome size file); low-depth scATAC-seq data may yield too few trustworthy peaks at q-value 0.01.

## Evidence

- [other] Stringent vs relaxed threshold rationale: [evidence span withheld — anchor status: not_found]
- [abstract] MACS3 as top peak caller: "HMMRATAC and MACS3 achieve the top performance in scATAC-seq peak calling"
- [methods] MACS3 invocation within scReadSim: "scReadSim utilizes MACS3 through function `Utility.CallPeak`"
- [methods] Complement feature derivation: "To generate background features for input features, use function `Utility.ComplementFeature` with following arguments"
- [readme] MACS3 repo availability: [evidence span withheld — anchor status: external_doc]
