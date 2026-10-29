---
name: pipeline-runtime-memory-benchmarking
description: Use when comparing barcode pre-processing pipelines on identical input
  FASTQ data and you need wall-clock runtime, peak memory, and barcode-detection/correlation
  concordance metrics.
when_to_use_negative:
- No gold-standard dataset with known barcode abundance is available to judge sensitivity
  — the article notes this made benchmarking against similar tools difficult
- The candidate tools have differing parameter flexibility such that parameterizations
  cannot be matched fairly across pipelines
- Your goal is accuracy assessment on simulated data — the authors explicitly did
  not benchmark on simulated data and flag it as future work
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: BARtab
  role: Pipeline under test; run in single-bulk reference-free mode via Nextflow to
    generate barcode counts tables
  repo: https://github.com/DaneVass/BARtab
- name: pycashier
  role: Comparator tool for barcode extraction/processing on the same dataset
  repo: https://github.com/brocklab/pycashier
- name: timemachine
  role: Comparator tool (Rewind/TimeMachine) for barcode detection on the same dataset
  repo: https://github.com/arjunrajlaboratory/timemachine
- name: bartools
  role: R package used to compute barcode set overlaps and Pearson/Spearman correlations
    across tool outputs
  repo: https://github.com/DaneVass/bartools
- name: starcode
  role: Message-passing clustering (Levenshtein distance) of trimmed barcode reads
    to derive consensus repertoires
  repo: https://github.com/gui11aume/starcode
- name: fastp
  role: Quality filtering of reads prior to barcode extraction
  repo: https://github.com/OpenGene/fastp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:96ff306c4136d6cc3a6d2c9865242c0217ad179d6dbe1e4253f3b1698bc86d58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - R
  - bartools
  - fastp
  - pycashier
  - starcode
  - timemachine
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# pipeline-runtime-memory-benchmarking

## Summary

Benchmark the runtime, peak memory, and barcode-detection concordance of bioinformatics pipelines by running competing tools (BARtab, pycashier, TimeMachine) on the same 22-sample Goyal et al. FASTQ dataset and comparing their outputs with bartools in R.

## When to use

Use when comparing barcode pre-processing pipelines on identical input FASTQ data and you need wall-clock runtime, peak memory, and barcode-detection/correlation concordance metrics.

## When NOT to use

- No gold-standard dataset with known barcode abundance is available to judge sensitivity — the article notes this made benchmarking against similar tools difficult
- The candidate tools have differing parameter flexibility such that parameterizations cannot be matched fairly across pipelines
- Your goal is accuracy assessment on simulated data — the authors explicitly did not benchmark on simulated data and flag it as future work

## Inputs

- 22-sample population-level barcoding FASTQ dataset (Goyal et al., 100bp FateMap barcode construct)
- BARtab Nextflow pipeline (DaneVass/bartab) parameterization (fastp minqual 20 / pctqual 80, cutadapt constantmismatches 0.1 / min_readlength 20, starcode cluster_distance 3 / cluster_ratio 3)
- pycashier and TimeMachine runs with STAR Methods parameterizations
- Published Goyal et al. barcode counts table (correlation reference)

## Outputs

- Per-sample starcode consensus tables (starcode/<sample_id>_starcode.tsv)
- Combined counts table (counts/all_counts_combined.tsv)
- tools_comparison_results.tsv with three-tool barcode overlap, pairwise-only overlaps, Pearson/Spearman correlations, and per-tool runtime/peak-memory figures

## How to apply

Run each tool (BARtab via Nextflow in single-bulk reference-free mode, pycashier, and TimeMachine) on the same 22-sample FASTQ dataset using the parameterizations reported in STAR Methods, while recording wall-clock runtime and peak memory. For BARtab, quality-filter with fastp (minqual 20, pctqual 80), trim constant regions with cutadapt (constantmismatches 0.1, min_readlength 20), then cluster trimmed reads with starcode message-passing at Levenshtein cluster_distance 3 and cluster_ratio 3 (distance 8 for the 100bp FateMap construct). Combine per-sample counts into counts/all_counts_combined.tsv via an outer join, then use bartools in R to compute barcode set overlaps across tools and Pearson/Spearman correlations of BARtab counts against the published Goyal et al. dataset. Reported reference results: 144,745 barcodes (97.3%) detected by all three tools, 1,810 shared only by BARtab and TimeMachine, concordance >0.96, and runtimes/memory of BARtab 35 min / 6.83 GB, pycashier 36 min / 1.81 GB, TimeMachine 4h59min / 3.52 GB.

## Related tools

- **BARtab** (Pipeline under test; run in single-bulk reference-free mode via Nextflow to generate barcode counts tables) — https://github.com/DaneVass/BARtab
- **pycashier** (Comparator tool for barcode extraction/processing on the same dataset) — https://github.com/brocklab/pycashier
- **timemachine** (Comparator tool (Rewind/TimeMachine) for barcode detection on the same dataset) — https://github.com/arjunrajlaboratory/timemachine
- **bartools** (R package used to compute barcode set overlaps and Pearson/Spearman correlations across tool outputs) — https://github.com/DaneVass/bartools
- **starcode** (Message-passing clustering (Levenshtein distance) of trimmed barcode reads to derive consensus repertoires) — https://github.com/gui11aume/starcode
- **fastp** (Quality filtering of reads prior to barcode extraction) — https://github.com/OpenGene/fastp

## Examples

```
nextflow run DaneVass/bartab -profile test_SE_ref_free,docker --minqual 20 --pctqual 80 --constantmismatches 0.1 --min_readlength 20 --cluster_distance 3 --cluster_ratio 3
```

## Evaluation signals

- tools_comparison_results.tsv reports a three-tool barcode overlap of 144,745 barcodes (97.3% of barcodes with at least 0.001% frequency within a sample)
- BARtab+TimeMachine-only overlap equals 1,810 barcodes
- Pearson and Spearman correlations of BARtab counts against the published Goyal et al. dataset exceed 0.96
- Per-tool runtime/peak-memory figures reproduce BARtab 35 min / 6.83 GB, pycashier 36 min / 1.81 GB, and TimeMachine 4h59min / 3.52 GB
- counts/all_counts_combined.tsv is produced by an outer join of the per-sample starcode TSVs and covers all 22 samples

## Limitations

- Benchmarking was difficult due to the lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools
- Accuracy and optimal parameter selection were not assessed on simulated data; the authors note this could give further insight
- Starcode clustering distance is construct-dependent: 3 is conservative for typical constructs, while the 100bp FateMap barcode construct recommends a distance of 8

## Evidence

- [discussion] Benchmarking BARtab against similar tools was difficult due to lack of a gold-standard dataset and differing parameter flexibility between tools: "benchmarking BARtab against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance"
- [discussion] BARtab was benchmarked by reproducing published results, showing improved runtime and sensitivity: "We benchmarked BARtab by reproducing published results, showing improved runtime and sensitivity."
- [other] Recording runtime and memory for the three compared tools: [evidence span withheld — anchor status: not_found]
- [methods] Comparison code and results location in the manuscript repository: "Code and results are available here: [https://github.com/DaneVass/bartools_manuscript_code/tree/main/tools-comparison]"
- [readme] README confirming the tools-comparison folder contents: [evidence span withheld — anchor status: external_doc]
