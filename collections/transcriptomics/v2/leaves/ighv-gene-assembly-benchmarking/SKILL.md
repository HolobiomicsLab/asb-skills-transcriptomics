---
name: ighv-gene-assembly-benchmarking
description: Use when comparing immunoglobulin variable-region reconstruction tools
  on RNA-seq data, specifically measuring percent IGHV gene assembled and runtime
  on the same BAM inputs at identical thread counts.
when_to_use_negative:
- The goal is immunogenotyping or SHM/hypermutation classification rather than head-to-head
  reconstruction-completeness benchmarking
- Input BAMs are aligned to a genome build other than hg38/GRCh38, since CRIS requires
  hg38-aligned, coordinate-sorted, indexed BAM input
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3168
tools:
- name: CRIS
  role: Primary tool being benchmarked; reconstructs the full IGHV gene from RNA-seq
    via de novo assembly, run in Docker (cris:v1) with 16 threads and 16G memory
  repo: https://github.com/Rashedul/CRIS
- name: Docker
  role: Container runtime used to execute CRIS reproducibly on each input BAM
- name: V'DJer
  role: Comparison tool benchmarked on the same 16 BAMs; assembles partial IGHV sequences
    (~75.44%)
- name: TRUST
  role: Comparison tool (v3.0.3) benchmarked on the same BAMs; recovers only ~41.5%
    of the IGHV gene
- name: MiXCR
  role: Comparison tool (v3.0.3) benchmarked on the same BAMs; assembles <10% of the
    IGHV gene
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:d8d72b0e195cd7bd8d3fe3b80a8f4ecf8a7ccb05602802147988093fa857d965
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# ighv-gene-assembly-benchmarking

## Summary

Benchmarks BCR/IGHV reconstruction tools (CRIS, V'DJer, TRUST, MiXCR) on CLL RNA-seq samples by tabulating the percent of the full IGHV gene each tool assembles and their wall-clock runtimes under identical thread settings.

## When to use

Use when comparing immunoglobulin variable-region reconstruction tools on RNA-seq data, specifically measuring percent IGHV gene assembled and runtime on the same BAM inputs at identical thread counts.

## When NOT to use

- The goal is immunogenotyping or SHM/hypermutation classification rather than head-to-head reconstruction-completeness benchmarking
- Input BAMs are aligned to a genome build other than hg38/GRCh38, since CRIS requires hg38-aligned, coordinate-sorted, indexed BAM input

## Inputs

- 16 RNA-seq FASTQ samples from GSE66228 (CLL)
- Coordinate-sorted, indexed BAM files aligned to hg38 (GRCh38) build
- Installed tool containers: cris:v1 Docker image, V'DJer, TRUST v3.0.3, MiXCR v3.0.3

## Outputs

- benchmark_table.tsv with percent IGHV gene assembled per tool per sample
- Wall-clock runtime measurements per tool at 16 threads
- CRIS IgBLAST-based output files (e.g. .IgBLAST_out.txt with percent identity and germline IGHV alignments)

## How to apply

Retrieve the RNA-seq samples (e.g. 16 CLL samples from GSE66228), align reads to the hg38 (GRCh38) build with STAR, and produce coordinate-sorted, indexed BAMs. Run CRIS via the cris:v1 Docker container with 16 threads and 16G memory, and run V'DJer, TRUST (v3.0.3), and MiXCR (v3.0.3) on the same BAMs under identical 16-thread conditions. From each tool's output (CRIS's is IgBLAST-based), compute the percent of the full IGHV gene reconstructed — CRIS's de novo assembly of Ig transcripts recovers nearly the whole segment (~99.74%), whereas V'DJer (~75.44%), TRUST (~41.5%), and MiXCR (<10%) generate only partial IGHV sequences. Tabulate percent IGHV assembled and wall-clock runtime per tool per sample and save as benchmark_table.tsv. Rationale: comparing on the same coordinate-sorted hg38 BAMs at the same thread count isolates differences in reconstruction completeness rather than input or compute variation.

## Related tools

- **CRIS** (Primary tool being benchmarked; reconstructs the full IGHV gene from RNA-seq via de novo assembly, run in Docker (cris:v1) with 16 threads and 16G memory) — https://github.com/Rashedul/CRIS
- **Docker** (Container runtime used to execute CRIS reproducibly on each input BAM)
- **V'DJer** (Comparison tool benchmarked on the same 16 BAMs; assembles partial IGHV sequences (~75.44%))
- **TRUST** (Comparison tool (v3.0.3) benchmarked on the same BAMs; recovers only ~41.5% of the IGHV gene)
- **MiXCR** (Comparison tool (v3.0.3) benchmarked on the same BAMs; assembles <10% of the IGHV gene)

## Examples

```
sudo docker run --name cris_analysis -v /fullPath/SRR1814049_test.bam:/cris/SRR1814049_test.bam -t cris:v1 bash CRIS_docker.sh SRR1814049_test.bam threads=4 memory=16G
```

## Evaluation signals

- benchmark_table.tsv reproduces the reported comparison: CRIS 99.74%, V'DJer 75.44%, TRUST 41.5%, MiXCR <10% IGHV assembled on average across the 16 GSE66228 samples
- Runtimes at 16 threads match reported values: CRIS 3.07 min, V'DJer 3.50 min, TRUST 36 min
- All four tools were run on the identical set of coordinate-sorted hg38 BAMs under identical 16-thread conditions
- CRIS output includes the IgBLAST file with percent identity and alignment to top germline IGHV hits

## Limitations

- CRIS requires a minimum of 16GB RAM, 4 threads, and ≥25M sequence reads per sample (ribodepleted or polyA-selected)
- The BAM must be aligned to the hg38 (GRCh38) build using BWA, coordinate-sorted and indexed; other builds are not supported
- Comparison tools reconstruct only a portion of the IGHV gene, so the benchmark measures reconstruction completeness, not downstream immuno-classification accuracy

## Evidence

- [other] task card: tool comparison: [evidence span withheld — anchor status: not_found]
- [abstract] article: CRIS vs partial-reconstruction tools: "CRIS improves on existing tools that infer the B-cell receptor repertoire from RNA-seq data using a portion IGHV gene segment by de novo assembly"
- [readme] README: resource requirements: "A minimum of 16GB RAM and 4 threads are required to run CRIS"
- [readme] README: input requirements: [evidence span withheld — anchor status: not_found]
