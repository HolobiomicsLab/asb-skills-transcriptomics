---
name: cross-tool-barcode-overlap-analysis
description: Use when you have barcode count tables produced by two or more pre-processing
  tools on the same FASTQ dataset and need to benchmark their detection concordance,
  quantify three-tool overlaps, and compare runtime/memory.
when_to_use_negative:
- Your dataset has a gold-standard truth set with known barcode abundances — direct
  accuracy benchmarking on simulated or spike-in data is preferable (the article notes
  such simulation benchmarking was not performed and would give further insight into
  accuracy and optimal parameter selection)
- The tools being compared operate on different inputs or parameter spaces that cannot
  be harmonized — the article notes differing parameter flexibility between tools
  made benchmarking difficult
- Evolving (editing-prone) barcode methods are the target — the article states these
  are not officially supported
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: BARtab
  role: Nextflow pipeline run in single-bulk reference-free mode to generate the reference
    barcode count table (fastp, cutadapt, starcode clustering) for the benchmark
  repo: https://github.com/DaneVass/BARtab
- name: pycashier
  role: External barcode pre-processing tool run on the same dataset with STAR Methods
    parameterizations as a comparison baseline
  repo: https://github.com/brocklab/pycashier
- name: timemachine
  role: Rewind/TimeMachine scripts run on the same dataset as a second comparison
    baseline
  repo: https://github.com/arjunrajlaboratory/timemachine
- name: bartools
  role: R package used to compute barcode set overlaps across tools and Pearson/Spearman
    correlations against the published Goyal et al. counts
  repo: https://github.com/DaneVass/bartools
- name: starcode
  role: Message-passing sequence clustering (Levenshtein distance 3, ratio 3) to derive
    BARtab's consensus barcode repertoire
  repo: https://github.com/gui11aume/starcode
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:74b47033e1dd142252be20da3ac42157ff55d29e91658308254db2fef9286cd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - R
  - bartools
  - pycashier
  - starcode
  - timemachine
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-tool-barcode-overlap-analysis

## Summary

Quantify agreement between barcode pre-processing tools (BARtab, pycashier, Rewind/TimeMachine) by combining their per-sample barcode count tables and computing set overlaps and count correlations with bartools in R, as done for the 22-sample Goyal population-level barcoding dataset.

## When to use

Use when you have barcode count tables produced by two or more pre-processing tools on the same FASTQ dataset and need to benchmark their detection concordance, quantify three-tool overlaps, and compare runtime/memory.

## When NOT to use

- Your dataset has a gold-standard truth set with known barcode abundances — direct accuracy benchmarking on simulated or spike-in data is preferable (the article notes such simulation benchmarking was not performed and would give further insight into accuracy and optimal parameter selection)
- The tools being compared operate on different inputs or parameter spaces that cannot be harmonized — the article notes differing parameter flexibility between tools made benchmarking difficult
- Evolving (editing-prone) barcode methods are the target — the article states these are not officially supported

## Inputs

- 22-sample population-level barcoding FASTQ dataset (Goyal et al., FateMap 100bp SPLINTR barcode construct)
- Per-sample barcode count tables from each tool (e.g. starcode/<sample_id>_starcode.tsv from BARtab; pycashier and TimeMachine equivalent count outputs)
- Published reference barcode counts from the Goyal et al. dataset for correlation
- Tool runtimes and peak-memory measurements for each of the three tools

## Outputs

- counts/all_counts_combined.tsv — outer-joined barcode count table across samples
- tools_comparison_results.tsv — three-tool overlap (144,745 barcodes, 97.3%), BARtab+TimeMachine-only overlap (1,810), Pearson and Spearman correlations (>0.96), per-tool runtime/memory
- Barcode overlap counts computed with bartools in R

## How to apply

Run each tool independently on the same 22-sample FASTQ dataset: BARtab in single-bulk reference-free mode via Nextflow (fastp minqual 20, pctqual 80; cutadapt constantmismatches 0.1, min_readlength 20; starcode message-passing clustering at Levenshtein distance 3, ratio 3), then pycashier and TimeMachine with the STAR Methods parameterizations. Combine BARtab's per-sample starcode outputs (<sample_id>_starcode.tsv) into one table with an outer join via COMBINE_BARCODE_COUNTS (counts/all_counts_combined.tsv), and record wall-clock runtime and peak memory per tool (published: BARtab 35 min/6.83 GB; pycashier 36 min/1.81 GB; TimeMachine 4h59min/3.52 GB). In R, use bartools to compute barcode set overlaps across tools — barcodes detected by all three (144,745, i.e. 97.3% of barcodes with at least 0.001% frequency within a sample) and the BARtab+TimeMachine-only overlap (1,810) — and Pearson/Spearman correlation of BARtab counts against the published Goyal et al. dataset (concordance >0.96). Rationale: without a gold-standard dataset with known barcode abundance, three-way overlap plus correlation against published results serves as the concordance benchmark. Output a tools_comparison_results.tsv capturing the overlap counts, correlations, and per-tool runtime/memory figures.

## Related tools

- **BARtab** (Nextflow pipeline run in single-bulk reference-free mode to generate the reference barcode count table (fastp, cutadapt, starcode clustering) for the benchmark) — https://github.com/DaneVass/BARtab
- **pycashier** (External barcode pre-processing tool run on the same dataset with STAR Methods parameterizations as a comparison baseline) — https://github.com/brocklab/pycashier
- **timemachine** (Rewind/TimeMachine scripts run on the same dataset as a second comparison baseline) — https://github.com/arjunrajlaboratory/timemachine
- **bartools** (R package used to compute barcode set overlaps across tools and Pearson/Spearman correlations against the published Goyal et al. counts) — https://github.com/DaneVass/bartools
- **starcode** (Message-passing sequence clustering (Levenshtein distance 3, ratio 3) to derive BARtab's consensus barcode repertoire) — https://github.com/gui11aume/starcode

## Examples

```
nextflow run DaneVass/bartab --mode single-bulk --input 'fastq/*_{1,2}.fastq.gz' --ref 'none' --cluster_distance 3 -profile docker,slurm
```

## Evaluation signals

- tools_comparison_results.tsv reports 144,745 barcodes detected by all three tools, corresponding to 97.3% of barcodes with at least 0.001% frequency within a sample
- The BARtab-and-TimeMachine-only overlap is 1,810 barcodes
- Pearson and Spearman correlations of BARtab counts against the published Goyal et al. dataset exceed 0.96
- Per-tool runtime/memory match published figures: BARtab 35 min / 6.83 GB, pycashier 36 min / 1.81 GB, TimeMachine 4h59min / 3.52 GB
- counts/all_counts_combined.tsv contains all samples joined by outer join with barcode counts per sample

## Limitations

- No gold-standard dataset with known barcode abundance exists, so benchmarking relies on cross-tool agreement rather than accuracy; the article notes differing parameter flexibility between tools complicated comparison
- Benchmarking on simulated data (for accuracy and optimal parameter selection) was not performed
- BARtab and bartools do not officially support evolving barcode methods

## Evidence

- [results] 144,745 barcodes (97.3% of barcodes with at least 0.001% frequency within a sample) were detected by all three tools — BARtab, pycashier, and Rewind/TimeMachine: [evidence span withheld — anchor status: not_found]
- [discussion] benchmarking difficulty due to no gold standard: "benchmarking BARtab against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools."
- [discussion] improved runtime and sensitivity on reproduction: "We benchmarked BARtab by reproducing published results, showing improved runtime and sensitivity."
- [methods] Nextflow invocation of BARtab: "nextflow run DaneVass/bartab --help"
- [methods] tools-comparison code and results location: "Code and results are available here: [https://github.com/DaneVass/bartools_manuscript_code/tree/main/tools-comparison](https://github.com/DaneVass/bartools_manuscript_code/tree/main/tools-comparison)"
- [readme] Comparison of BARtab with pycashier and timemachine in tools-comparison folder: [evidence span withheld — anchor status: external_doc]
