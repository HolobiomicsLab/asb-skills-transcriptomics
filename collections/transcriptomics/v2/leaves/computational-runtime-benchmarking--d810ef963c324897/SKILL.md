---
name: computational-runtime-benchmarking
description: 'Use when comparing computational cost of two small RNA-seq pipelines:
  you have identical FASTQ inputs (e.g., 50 nt serum reads), matched parameters, and
  need per-sample runtime for a stand-alone vs web-based tool.'
when_to_use_negative:
- Inputs are not the original FASTQ files (e.g., already-processed count matrices),
  since the benchmark requires timing full pipelines from raw reads
- Comparing functional output quality (RNA diversity/abundance detection) rather than
  computational cost — use tool-output comparison instead
- Inputs are very large datasets that cannot run through a web-based upload tool,
  since exceRpt requires uploading original sequencing files, which the article notes
  is unworkable for big data
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: COMPSRA
  role: Stand-alone pipeline being benchmarked; run per-sample on 30 GB RAM server
    with parameters matching exceRpt
  repo: https://github.com/cougarlj/COMPSRA
- name: exceRpt
  role: Web-based comparison pipeline run on the same 12 FASTQ inputs to record per-sample
    runtime
- name: STAR
  role: Default aligner inside the timed COMPSRA Alignment module (hg38 mapping)
- name: BLAST
  role: Alignment engine of the timed COMPSRA Microbe module against the UCSC nt database
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:1de6af70d3d8b7a737e5fb59fda05f25a86dbd00281d77f3bc64606ec47ac387
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - BLAST
  - COMPSRA
  - Java
  - STAR
  - exceRpt
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# computational-runtime-benchmarking

## Summary

Benchmark per-sample runtime of a bioinformatics pipeline (e.g., COMPSRA, a Java stand-alone small RNA-seq platform) against a comparable web-based tool (exceRpt) by running both on identical inputs with matched parameters, as done for 12 healthy human serum small RNA-seq samples.

## When to use

Use when comparing computational cost of two small RNA-seq pipelines: you have identical FASTQ inputs (e.g., 50 nt serum reads), matched parameters, and need per-sample runtime for a stand-alone vs web-based tool.

## When NOT to use

- Inputs are not the original FASTQ files (e.g., already-processed count matrices), since the benchmark requires timing full pipelines from raw reads
- Comparing functional output quality (RNA diversity/abundance detection) rather than computational cost — use tool-output comparison instead
- Inputs are very large datasets that cannot run through a web-based upload tool, since exceRpt requires uploading original sequencing files, which the article notes is unworkable for big data

## Inputs

- 12 small RNA-seq FASTQ files (50 nt raw reads) from healthy human serum controls
- COMPSRA v1.0 (Java JAR) installation with STAR aligner and prebuilt hg38 annotation databases
- exceRpt web pipeline access with the same parameters as COMPSRA
- BLAST plus UCSC nt microbial databases (blast_archaea, blast_bacteria, blast_fungi, blast_viruses) for the Microbe module

## Outputs

- Per-sample runtime measurements for COMPSRA (QC + Alignment + Annotation)
- Per-sample runtime measurements for exceRpt
- Average runtime of the COMPSRA Microbe module across the 12 samples
- runtime_benchmark.csv aggregating measured and reported reference timings per tool/module

## How to apply

Obtain the same FASTQ inputs for both tools — here 12 serum samples from healthy human controls with 50 nt raw reads. Run COMPSRA v1.0 (Java) on a server with 30 GB RAM using parameters set identically to exceRpt, timing the full QC → STAR alignment (hg38) → Annotation pipeline per sample. Run exceRpt on the same 12 FASTQ files and record its per-sample runtime. Optionally time the COMPSRA Microbe module (BLAST of unmapped reads against the UCSC nt database, covering viruses, archaea, bacteria and fungi) and average across the 12 samples. Aggregate measured timings per tool and module into a runtime table (e.g., runtime_benchmark.csv) including a column with the paper's reported reference values (COMPSRA ~10 min/sample, exceRpt ~20 min/sample, ~3.5 h average for the microbiome module).

## Related tools

- **COMPSRA** (Stand-alone pipeline being benchmarked; run per-sample on 30 GB RAM server with parameters matching exceRpt) — https://github.com/cougarlj/COMPSRA
- **exceRpt** (Web-based comparison pipeline run on the same 12 FASTQ inputs to record per-sample runtime)
- **STAR** (Default aligner inside the timed COMPSRA Alignment module (hg38 mapping))
- **BLAST** (Alignment engine of the timed COMPSRA Microbe module against the UCSC nt database)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1,2,3,4,5,6 -inf ./example/sample.list -out ./example_out/ (timed per sample; compare against exceRpt on the same FASTQ list)
```

## Evaluation signals

- Measured COMPSRA runtime per sample approximates ~10 minutes on the 12 serum samples
- Measured exceRpt runtime per sample approximates ~20 minutes
- Average Microbe module runtime across the 12 samples approximates ~3.5 hours (viruses, archaea, bacteria, fungi)
- Both tools were run on the same 12 FASTQ inputs with identical parameters on a 30 GB RAM server
- runtime_benchmark.csv contains measured values alongside the reported reference values for each tool and module

## Limitations

- Results are hardware- and parameter-dependent: the reference values assume a server with 30 GB RAM and parameters matched to exceRpt; timings will differ on other machines or thread counts (COMPSRA -t default is 1 thread)
- The article's results section does not explicitly state the COMPSRA software version used for the benchmark
- Web-based exceRpt requires uploading original sequencing files, which is unworkable for big data, limiting this comparison to small-to-moderate datasets
- First-time STAR index building inside COMPSRA takes about 3 hours and should be excluded from per-sample runtime accounting

## Evidence

- [results] COMPSRA ~10 min/sample vs exceRpt ~20 min/sample: "COMPSRA takes ~10 minutes per sample, but exceRpt will cost ~20 minutes for each sample (see Supplementary Fig. S1)"
- [results] Matched parameters and hardware: "We run COMPSRA on server with 30 GB RAM and set the same parameters as exceRpt"
- [results] Microbe module ~3.5 h average: "For the 12 test samples, the estimated time on average is 3.5 hours, including viruses, archaea, bacteria and fungi (see Supplementary Table S1)"
- [readme] STAR index build cost caveat: "Note: -mbi was only needed for the first time when you run COMPSRA and built the index files for STAR. This process will cost about 3 hours."
- [intro] Web upload limitation of exceRpt-class tools: "All the tools need users to upload the original sequencing files, which is un-workable for big data"
