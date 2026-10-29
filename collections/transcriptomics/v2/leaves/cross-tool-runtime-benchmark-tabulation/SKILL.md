---
name: cross-tool-runtime-benchmark-tabulation
description: Use when comparing the IGHV-reconstruction completeness or runtime of
  competing BCR reconstruction tools across the same set of RNA-seq samples, e.g.
  reproducing the CRIS vs V'DJer/TRUST/MiXCR benchmark on GSE66228 CLL samples.
when_to_use_negative:
- The analysis goal is a single tool's IGHV hypermutation/SHM classification rather
  than a head-to-head multi-tool comparison — this skill only tabulates comparative
  assembly and runtime metrics
- Input data are not RNA-seq BAMs aligned to the hg38 (GRCh38) build, which CRIS requires
  as input
- Samples have fewer than ~25M reads, below the minimum CRIS states it needs per sample
tools:
- name: CRIS
  role: Benchmark subject; run via cris:v1 Docker container with 16 threads and 16G
    memory; percent IGHV reconstructed read from its IgBLAST-based output
  repo: https://github.com/Rashedul/CRIS
- name: V'DJer
  role: Comparison tool run on the same 16 BAMs at 16 threads to record runtime and
    percent IGHV gene assembled
- name: TRUST
  role: Comparison tool (v3.0.3) run on the same BAMs at 16 threads to record runtime
    and percent IGHV gene assembled
- name: MiXCR
  role: Comparison tool (v3.0.3) run on the same BAMs at 16 threads to record runtime
    and percent IGHV gene assembled
- name: Docker
  role: Containerization used to execute CRIS reproducibly with fixed threads and
    memory parameters
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:9f7ae9c2f8ddb51152d64df2039af231b923a24e88f49e62a82de93318e894a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - CRIS
  - Docker
  - MiXCR
  - TRUST
  - V'DJer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# Cross-tool runtime benchmark tabulation

## Summary

A benchmarking procedure that runs multiple BCR/IGHV reconstruction tools (CRIS, V'DJer, TRUST, MiXCR) on identical CLL RNA-seq BAMs under matched thread/memory conditions and tabulates percent IGHV gene assembled plus wall-clock runtime into a single summary table (benchmark_table.tsv).

## When to use

Use when comparing the IGHV-reconstruction completeness or runtime of competing BCR reconstruction tools across the same set of RNA-seq samples, e.g. reproducing the CRIS vs V'DJer/TRUST/MiXCR benchmark on GSE66228 CLL samples.

## When NOT to use

- The analysis goal is a single tool's IGHV hypermutation/SHM classification rather than a head-to-head multi-tool comparison — this skill only tabulates comparative assembly and runtime metrics
- Input data are not RNA-seq BAMs aligned to the hg38 (GRCh38) build, which CRIS requires as input
- Samples have fewer than ~25M reads, below the minimum CRIS states it needs per sample

## Inputs

- 16 GSE66228 CLL RNA-seq samples from GEO (FASTQ)
- Coordinate-sorted, indexed BAM files aligned to hg38 (GRCh38)
- Tool containers/executables: cris:v1 Docker image, V'DJer, TRUST v3.0.3, MiXCR v3.0.3

## Outputs

- benchmark_table.tsv with percent IGHV gene assembled and wall-clock runtime per tool per sample
- Per-sample tool outputs (e.g. CRIS IgBLAST_out.txt percent-identity alignments) from which the benchmark metrics are derived

## How to apply

Retrieve the 16 GSE66228 CLL RNA-seq samples from GEO and align reads to the GRCh38 (hg38) build with STAR, producing coordinate-sorted and indexed BAMs. Run each of the four tools (CRIS via the cris:v1 Docker container, V'DJer, TRUST v3.0.3, MiXCR v3.0.3) on the same BAMs under identical 16-thread conditions, with CRIS allocated 16G assembly memory. For each run record wall-clock runtime and the percent of the full IGHV gene reconstructed — for CRIS this is derived from the IgBLAST-based output (IgBLAST_out.txt percent identity/alignment to the top germline IGHV hit). Tabulate percent IGHV assembled and runtime for all four tools across the 16 samples and save as benchmark_table.tsv. The key rationale is fairness: identical thread counts, identical input BAMs, and a completeness metric (percent IGHV gene assembled) that exposes partial-IGHV generation by V'DJer and TRUST.

## Related tools

- **CRIS** (Benchmark subject; run via cris:v1 Docker container with 16 threads and 16G memory; percent IGHV reconstructed read from its IgBLAST-based output) — https://github.com/Rashedul/CRIS
- **V'DJer** (Comparison tool run on the same 16 BAMs at 16 threads to record runtime and percent IGHV gene assembled)
- **TRUST** (Comparison tool (v3.0.3) run on the same BAMs at 16 threads to record runtime and percent IGHV gene assembled)
- **MiXCR** (Comparison tool (v3.0.3) run on the same BAMs at 16 threads to record runtime and percent IGHV gene assembled)
- **Docker** (Containerization used to execute CRIS reproducibly with fixed threads and memory parameters)

## Examples

```
sudo docker run --name cris_analysis -v /fullPath/sample.bam:/cris/sample.bam -t cris:v1 bash CRIS_docker.sh sample.bam threads=16 memory=16G
```

## Evaluation signals

- benchmark_table.tsv contains rows for all four tools across all 16 samples with both percent-IGHV-assembled and wall-clock runtime columns
- CRIS mean percent IGHV gene reconstructed is ≈ 99.74%, versus V'DJer ≈ 75.44%, TRUST ≈ 41.5%, and MiXCR < 10% IGHV assembled
- Runtimes at 16 threads reproduce reported values: CRIS ≈ 3.07 min, V'DJer ≈ 3.50 min, TRUST ≈ 36 min
- All four tools consumed identical input BAMs under identical 16-thread conditions (fair-comparison invariant)
- CRIS per-sample IgBLAST output reports a germline IGHV hit with percent identity consistent with near-complete IGHV reconstruction

## Limitations

- CRIS requires ≥16GB RAM and ≥4 threads, and BAMs aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed — benchmark setups deviating from these constraints are not directly comparable
- V'DJer and TRUST generate only partial IGHV sequences, so the percent-assembly metric structurally favors full-length reconstruction tools like CRIS
- Both ribodepleted and polyA selected RNA-seq are appropriate for CRIS, but a minimum of 25M reads per sample is required

## Evidence

- [other] Benchmark result of CRIS vs V'DJer/TRUST/MiXCR on GSE66228: [evidence span withheld — anchor status: not_found]
- [other] Benchmark tabulation step from source card workflow: [evidence span withheld — anchor status: not_found]
- [readme] Resource requirements constraining fair benchmarking: "A minimum of 16GB RAM and 4 threads are required to run CRIS"
- [readme] CRIS output used to derive percent IGHV reconstruction: "Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits."
