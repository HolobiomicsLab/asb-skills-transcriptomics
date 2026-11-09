---
name: simulated-atac-seq-read-length-benchmarking
description: Use when validating or benchmarking ecDNA breakpoint reconstruction from
  ATAC-Seq under controlled read lengths/depths, or when reproducing the paper's robustness
  claim (F1 = 0.93 at 30× local depth with reads as short as 35 bp).
when_to_use_negative:
- Input is real patient tumor ATAC-Seq rather than simulated reads — circlehunter's
  authors note 'the current benchmarking strategy has limitations, and we cannot guarantee
  the accuracy of ecDNA calls for all types of focal amplifications'
- Evaluating complex structural variants beyond simple circles — 'Future studies should
  explore the rate at which this happens' for failures on complex structural variants
- The goal is whole-genome ecDNA reconstruction from WGS copy-number data (e.g. AmpliconArchitect-style),
  not ATAC-Seq-based breakpoint estimation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: circlehunter
  role: Pipeline under benchmarking; its integrated Bayesian model supplies the breakpoint
    posterior (uniform discordant-pair likelihood + Gaussian clipped-read likelihood)
    whose MAP and 95% CI are evaluated across read lengths
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine that executes the circlehunter pipeline steps being benchmarked
    over the simulated read-length grid
- name: ATAC-Seq
  role: Assay simulated at 30× local depth across the read-length grid; ecDNA's high
    chromatin accessibility yields the enrichment signal the breakpoint model consumes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:1620736b9dba1454aa23db11abf031b899aec087c70602f630ccccff0e65dfae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - ATAC-Seq
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Simulated ATAC-Seq read-length benchmarking

## Summary

Benchmark circlehunter's Bayesian breakpoint-estimation model on simulated ATAC-Seq reads spanning a read-length grid (down to 35 bp) at 30× local depth, quantifying how MAP breakpoint precision (95% CI width) degrades as reads shorten (Fig. 2C).

## When to use

Use when validating or benchmarking ecDNA breakpoint reconstruction from ATAC-Seq under controlled read lengths/depths, or when reproducing the paper's robustness claim (F1 = 0.93 at 30× local depth with reads as short as 35 bp).

## When NOT to use

- Input is real patient tumor ATAC-Seq rather than simulated reads — circlehunter's authors note 'the current benchmarking strategy has limitations, and we cannot guarantee the accuracy of ecDNA calls for all types of focal amplifications'
- Evaluating complex structural variants beyond simple circles — 'Future studies should explore the rate at which this happens' for failures on complex structural variants
- The goal is whole-genome ecDNA reconstruction from WGS copy-number data (e.g. AmpliconArchitect-style), not ATAC-Seq-based breakpoint estimation

## Inputs

- Simulated ATAC-Seq reads (BAM/FASTQ) spanning the benchmark read-length grid down to 35 bp at 30× local depth
- Known ground-truth simulated ecDNA breakpoint positions per replicate
- circlehunter Snakemake pipeline configuration (yaml config file)

## Outputs

- breakpoint_ci_vs_readlength.csv (MAP estimate and 95% CI width per replicate and read length)
- fig2c_ci_width_vs_readlength.png (CI width versus read length, Fig. 2C style)
- F1-score assessment of breakpoint calls at 30× local depth per read length

## How to apply

Load the simulated ATAC-Seq reads produced upstream (benchmark read-length grid down to 35 bp at 30× local depth) and extract junction evidence around each simulated breakpoint: discordant read pairs with anomalous orientation/insert size plus soft-clipped or supplementary-aligned reads piling up at the junction. Build the breakpoint posterior by combining a uniform (German-tank) likelihood over the fragment interval spanned by each discordant pair with a Gaussian likelihood centered on each clipped-read offset. Grid-search the breakpoint position to obtain the MAP estimate and the 95% confidence interval for every simulated circle. Tabulate MAP and 95% CI width per replicate and read length into breakpoint_ci_vs_readlength.csv and render the CI-width-versus-read-length figure in the Fig. 2C style. Validate against the paper's reported robustness regime by confirming 95% CI widths increase monotonically as read length decreases, consistent with F1 = 0.93 at 30× local depth down to 35 bp.

## Related tools

- **circlehunter** (Pipeline under benchmarking; its integrated Bayesian model supplies the breakpoint posterior (uniform discordant-pair likelihood + Gaussian clipped-read likelihood) whose MAP and 95% CI are evaluated across read lengths) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine that executes the circlehunter pipeline steps being benchmarked over the simulated read-length grid)
- **ATAC-Seq** (Assay simulated at 30× local depth across the read-length grid; ecDNA's high chromatin accessibility yields the enrichment signal the breakpoint model consumes)

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- 95% CI widths increase monotonically as read length decreases (shorter reads yield wider intervals, as in Fig. 2C)
- F1 score reaches 0.93 at 30× local depth with read lengths as short as 35 bp on the simulated data
- MAP breakpoint estimate falls within the fragment interval spanned by its supporting discordant pair for every simulated circle
- breakpoint_ci_vs_readlength.csv contains a MAP and 95% CI row for every replicate × read-length combination with no missing entries
- fig2c_ci_width_vs_readlength.png reproduces the monotone CI-width trend visible in the paper's Fig. 2C

## Limitations

- Short read-length sequencing has a natural disadvantage for detecting structural variants, which may lead circlehunter to discover more computational sub-structures (e.g. COLO320DM)
- The current benchmarking strategy has limitations; accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed
- Read length directly affects breakpoint estimate precision — shorter reads yield larger breakpoint confidence intervals (95% CI, Fig. 2C)

## Evidence

- [abstract] Reported F1 robustness across the read-length grid: "circlehunter has an F1 score of 0.93 at 30× local depth and read lengths as short as 35 bp"
- [other] Source-card finding on Bayesian breakpoint estimation and CI width vs read length (Fig. 2C): [evidence span withheld — anchor status: not_found]
- [methods] Pipeline execution command used to run the benchmarked workflow: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [discussion] Failure mode under short reads: "short read-length sequencing has a natural disadvantage for detecting structural variants, which may lead circlehunter to discover more computational sub-structures"
- [readme] Tool scope statement from repository README: "circlehunter is a pipeline for identifying complex ecDNA using ATAC-Seq data."
