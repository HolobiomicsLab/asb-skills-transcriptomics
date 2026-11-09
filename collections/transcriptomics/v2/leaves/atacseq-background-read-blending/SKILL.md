---
name: atacseq-background-read-blending
description: 'Use when a benchmark of an ATAC-Seq-based ecDNA detector requires positive
  controls with known ground-truth coordinates: simulated ecDNA reads must be blended
  into real ATAC-Seq background at controlled local depths and read lengths.'
when_to_use_negative:
- The goal is discovering ecDNA in real tumor ATAC-Seq data, not evaluating a detector
  against known ground truth — blending only builds synthetic benchmark libraries
- Available data is WGS-based ecDNA detection (e.g. AmpliconArchitect-style, which
  requires copy number variation as input) rather than ATAC-Seq reads
- Input is single-cell ATAC-Seq data, which the article notes has yet to be explored
  for this application
edam_operation: http://edamontology.org/operation_0284
tools:
- name: ART
  role: Simulates sequencing reads for the 500 mock ecDNAs (5 kb-10 Mb, 1-50 segments
    from GRCh38) that are blended into background
- name: circlehunter
  role: Snakemake pipeline whose benchmarking consumes the blended libraries; environment
    built from environment.yml
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Orchestrates the simulation/blend/analysis pipeline via the Snakefile and
    config.yaml
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:2a4ef5bd654f8df77cbd3c403f5896be01cbb0dbf927d79a9548e25226962c1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - ART
  - Circle_finder
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_0284
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# atacseq-background-read-blending

## Summary

A benchmark-construction technique in which ART-simulated mock ecDNA reads (sampled from GRCh38) are spiked into real GM12878 ATAC-Seq background reads (ENCODE GSE170245) to create controlled test libraries spanning defined local depths and read lengths for evaluating ecDNA callers such as circlehunter and Circle_finder.

## When to use

Use when a benchmark of an ATAC-Seq-based ecDNA detector requires positive controls with known ground-truth coordinates: simulated ecDNA reads must be blended into real ATAC-Seq background at controlled local depths and read lengths.

## When NOT to use

- The goal is discovering ecDNA in real tumor ATAC-Seq data, not evaluating a detector against known ground truth — blending only builds synthetic benchmark libraries
- Available data is WGS-based ecDNA detection (e.g. AmpliconArchitect-style, which requires copy number variation as input) rather than ATAC-Seq reads
- Input is single-cell ATAC-Seq data, which the article notes has yet to be explored for this application

## Inputs

- Mock ecDNA coordinate definitions sampled from GRCh38 (each 5 kb-10 Mb, 1-50 genomic segments)
- ART-simulated sequencing reads of the mock ecDNAs
- GM12878 ATAC-Seq background reads (ENCODE GSE170245)
- YAML config enumerating all simulated samples and their outputs (config.yaml from config-template.yaml)

## Outputs

- Blended test ATAC-Seq libraries spanning varying local depths and read lengths
- Per-condition precision, recall, and F1 tables comparing detector calls to simulated ground truth (e.g. f1_scores_fig2ab.csv)

## How to apply

Use ART to generate sequencing reads for 500 mock ecDNAs sampled from GRCh38, each 5 kb-10 Mb in size and composed of 1-50 genomic segments. Blend the simulated ecDNA reads with GM12878 ATAC-Seq background reads (ENCODE GSE170245) to build test libraries spanning a grid of local depths (e.g. up to and above 30x) and read lengths (e.g. down to 35 bp). Enumerate all blended samples and their outputs in a YAML config (copied from config-template.yaml) and drive the blend/analysis through the Snakemake pipeline. Run each detector (circlehunter and Circle_finder) on every simulated library, compare calls against the simulated ground-truth ecDNA coordinates, and compute precision, recall, and F1 per depth and read-length condition (e.g. assembling f1_scores_fig2ab.csv). The rationale is that blending gives exact control of local depth and read length while retaining realistic ATAC-Seq background structure.

## Related tools

- **ART** (Simulates sequencing reads for the 500 mock ecDNAs (5 kb-10 Mb, 1-50 segments from GRCh38) that are blended into background)
- **circlehunter** (Snakemake pipeline whose benchmarking consumes the blended libraries; environment built from environment.yml) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Orchestrates the simulation/blend/analysis pipeline via the Snakefile and config.yaml)

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- circlehunter on the blended libraries achieves F1 = 0.93 at local depth > 30x (abstract reports 0.93 at 30x local depth)
- circlehunter F1 is nearly invariant to read length down to 35 bp, while Circle_finder fails below 75 bp reads
- circlehunter outperforms Circle_finder at all local depth levels except below 10x
- Every blended library has a matched ground-truth ecDNA coordinate set so precision/recall/F1 are computable per depth and read-length condition
- The YAML config enumerates all simulated samples and outputs before launching Snakemake with -j 16

## Limitations

- The article states the current benchmarking strategy has limitations and cannot guarantee accuracy of ecDNA calls for all types of focal amplifications
- Performance degrades below 10x local depth (circlehunter does not outperform Circle_finder there)
- Short read-length sequencing may cause circlehunter to discover more computational sub-structures (e.g. COLO320DM), affecting benchmark interpretation

## Evidence

- [methods] Blend the simulated ecDNA reads with GM12878 ATAC-Seq background reads (ENCODE GSE170245) to build test libraries spanning varying local depths and read lengths.: [evidence span withheld — anchor status: not_found]
- [methods] Use ART to simulate 500 mock ecDNAs sampled from GRCh38, each 5 kb-10 Mb in size and composed of 1-50 genomic segments, and generate their sequencing reads.: [evidence span withheld — anchor status: not_found]
- [abstract] circlehunter has an F1 score of 0.93 at 30× local depth and read lengths as short as 35 bp: "circlehunter has an F1 score of 0.93 at 30× local depth and read lengths as short as 35 bp"
- [readme] snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
