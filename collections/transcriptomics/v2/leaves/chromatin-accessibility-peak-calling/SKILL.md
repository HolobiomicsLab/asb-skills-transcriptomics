---
name: chromatin-accessibility-peak-calling
description: Use when DC3 has assigned single cells to subpopulations and you need
  subpopulation-specific open peaks and motif enrichment scores to nominate candidate
  key regulatory transcription factors.
when_to_use_negative:
- When single cells have not yet been assigned to subpopulations — peak calling should
  be done on DC3-merged per-subpopulation samples, not the raw heterogeneous mixture
- When the goal is deconvolving bulk HiChIP loop counts into subpopulation-specific
  contacts rather than calling accessibility peaks and motif enrichment
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: MACS2
  role: Calls open chromatin peaks on scATAC-seq samples merged per DC3 subpopulation
  repo: https://github.com/SUwonglab/DC3
- name: Homer
  role: Computes motif enrichment scores on the called peaks, defined as the geometric
    mean of -log10(p-value) and fold change
- name: DC3
  role: Provides the coupled clustering of single cells into subpopulations that defines
    which scATAC-seq cells are merged before peak calling
  repo: https://github.com/SUwonglab/DC3
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:d1613584d30eafa883808cf94e18afde044bb8247aedf9424ae5a432a2643ffc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/DC3
  tools:
  - DC3
  - Homer
  - MACS2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chromatin-accessibility-peak-calling

## Summary

Calls open chromatin peaks on merged scATAC-seq data from DC3-assigned subpopulations using MACS2, then computes Homer motif enrichment (geometric mean of -log10(p-value) and fold change) to support key transcription factor identification in each subpopulation.

## When to use

Use when DC3 has assigned single cells to subpopulations and you need subpopulation-specific open peaks and motif enrichment scores to nominate candidate key regulatory transcription factors.

## When NOT to use

- When single cells have not yet been assigned to subpopulations — peak calling should be done on DC3-merged per-subpopulation samples, not the raw heterogeneous mixture
- When the goal is deconvolving bulk HiChIP loop counts into subpopulation-specific contacts rather than calling accessibility peaks and motif enrichment

## Inputs

- Per-cell scATAC-seq .bam files for the heterogeneous cell population
- DC3 scATAC clustering results (scATAC-result.txt) mapping cells to subpopulations
- Subpopulation-specific gene expression in FPKM (merged scRNA-seq per DC3 subpopulation)
- Differential expression test results between subpopulations

## Outputs

- Open chromatin peak sets (MACS2) per DC3 subpopulation
- Homer motif enrichment scores per subpopulation (geometric mean of -log10(p-value) and fold change)
- Ranked key-transcription-factor lists per subpopulation (importance = log2(FPKM) x expression specificity x motif enrichment score)

## How to apply

First take the DC3 clustering results (scATAC-result.txt) and merge the .bam files of all cells assigned to each subpopulation into one sample per subpopulation. Call open peaks on each merged sample with MACS2 callpeak, then compute motif enrichment scores with Homer for the peak sets, defining the score as the geometric mean of -log10(p-value) and fold change. Combine the accessibility evidence with subpopulation-specific expression (FPKM merged from scRNA-seq cells per subpopulation) and differential expression (t-test, adjusted p-value < 0.01 via scipy) to select candidate key regulators with FPKM > 10 and motif enrichment score > 2. Rationale: DC3's coupled clustering of scRNA-seq and scATAC-seq yields subpopulation membership, and merged per-subpopulation ATAC signal plus motif enrichment links active regulatory elements to the TFs that bind them.

## Related tools

- **MACS2** (Calls open chromatin peaks on scATAC-seq samples merged per DC3 subpopulation) — https://github.com/SUwonglab/DC3
- **Homer** (Computes motif enrichment scores on the called peaks, defined as the geometric mean of -log10(p-value) and fold change)
- **DC3** (Provides the coupled clustering of single cells into subpopulations that defines which scATAC-seq cells are merged before peak calling) — https://github.com/SUwonglab/DC3

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Key regulator counts per subpopulation match the expected 58, 42, and 71 for subpopulations 1, 2, and 3 of the RA-day 4 mixture
- Subpopulation-specific TFs recover the reported sets: {Lhx1, Neurod1} in subpopulation 1, {Gata4, Sox17, Foxa2} in subpopulation 2, {Rfx4, Sox3} in subpopulation 3
- Every retained key regulator passes all three criteria: FPKM > 10, motif enrichment score > 2, and differential expression with adjusted p-value < 0.01
- Motif enrichment score equals the geometric mean of -log10(p-value) and fold change for each subpopulation peak set

## Limitations

- Requires DC3 (or equivalent) subpopulation assignments first — the merged per-subpopulation samples depend on coupled clustering quality
- The README preprocessing assumes per-cell .bam files organized in a folder and a standard pipeline producing the openness matrix; deviations may break the workflow
- Differential expression relies on a simple t-test with a Bonferroni-style adjusted p-value cutoff, which may not capture subtler specificity patterns

## Evidence

- [other] Subpopulation-specific peak calling and motif enrichment workflow: [evidence span withheld — anchor status: not_found]
- [other] Subpopulation merging prior to peak calling: [evidence span withheld — anchor status: not_found]
- [other] Key regulator criteria: [evidence span withheld — anchor status: not_found]
- [abstract] DC3 provides subpopulation assignments: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data"
- [readme] scATAC-seq preprocessing expectations from README: "For preprocessing the scATAC-seq data, please first put all the .bam files for each cell into a folder. Then run the preprossing script we provided to get the the openness matrix"
