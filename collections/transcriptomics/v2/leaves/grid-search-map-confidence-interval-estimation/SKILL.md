---
name: grid-search-map-confidence-interval-estimation
description: Use when reconstructing exact ecDNA breakpoints from ATAC-Seq junction
  evidence (discordant read pairs, soft-clipped/supplementary reads) and quantifying
  breakpoint confidence intervals across read lengths, e.g. down to 35 bp.
when_to_use_negative:
- Benchmarking or calling ecDNA for focal amplifications that are not accessible ecDNA
  circles — circlehunter may detect misregulated-chromatin amplifications as false-positive
  ecDNA
- Detecting complex structural variants whose junctions lack ATAC-Seq signal — circlehunter
  reports such ecDNA 'may not be identified by circlehunter without this information'
- Single-cell ATAC-Seq data — application to single-cell data has yet to be explored
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: circlehunter
  role: ATAC-Seq pipeline that implements the integrated Bayesian breakpoint model
    with grid-searched MAP and 95% CI estimates for ecDNA junctions
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine that orchestrates the circlehunter pipeline execution (Snakefile
    + yaml config)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:55757f35b772282453753555094b96813f002099b7f515865e0739437cf9431e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Grid-search MAP confidence interval estimation

## Summary

Bayesian breakpoint-estimation method from circlehunter that combines a uniform (German-tank) likelihood over discordant-pair-supported fragment intervals with a Gaussian likelihood on soft-clip offsets, then grid-searches the ecDNA junction position for the MAP estimate and its 95% CI without read realignment.

## When to use

Use when reconstructing exact ecDNA breakpoints from ATAC-Seq junction evidence (discordant read pairs, soft-clipped/supplementary reads) and quantifying breakpoint confidence intervals across read lengths, e.g. down to 35 bp.

## When NOT to use

- Benchmarking or calling ecDNA for focal amplifications that are not accessible ecDNA circles — circlehunter may detect misregulated-chromatin amplifications as false-positive ecDNA
- Detecting complex structural variants whose junctions lack ATAC-Seq signal — circlehunter reports such ecDNA 'may not be identified by circlehunter without this information'
- Single-cell ATAC-Seq data — application to single-cell data has yet to be explored

## Inputs

- ATAC-Seq reads (FASTQ/BAM) spanning a read-length grid down to 35 bp at 30x local depth (e.g. simulated by task_001)
- Discordant read pairs with anomalous orientation/insert size at candidate junctions
- Soft-clipped or supplementary-aligned read pileups at junctions
- Known or simulated breakpoint positions for validation

## Outputs

- breakpoint_ci_vs_readlength.csv (MAP estimate and 95% CI width per replicate and read length)
- Fig. 2C-style plot of 95% CI width versus read length (fig2c_ci_width_vs_readlength.png)
- Per-circle MAP breakpoint coordinates with 95% confidence intervals

## How to apply

Extract junction evidence around each candidate breakpoint: discordant read pairs with anomalous orientation/insert size and soft-clipped or supplementary-aligned read pileups at the junction. Build the breakpoint posterior by combining a uniform (German-tank) likelihood over the fragment interval spanned by each discordant pair with a Gaussian likelihood centered on each clipped-read offset. Grid-search the breakpoint position to obtain the MAP estimate and the 95% confidence interval per circle, avoiding time-consuming read realignment. Tabulate MAP and 95% CI width per replicate and read length (simulated ATAC-Seq down to 35 bp at 30x local depth) and verify that CI widths increase monotonically as read length decreases, consistent with the paper's robustness regime.

## Related tools

- **circlehunter** (ATAC-Seq pipeline that implements the integrated Bayesian breakpoint model with grid-searched MAP and 95% CI estimates for ecDNA junctions) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine that orchestrates the circlehunter pipeline execution (Snakefile + yaml config))

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- F1 = 0.93 at 30x local depth with read lengths as short as 35 bp on simulated data
- 95% CI widths increase monotonically as read length decreases (shorter reads yield larger breakpoint CIs)
- MAP breakpoint estimates match known simulated breakpoint positions within the reported 95% CI
- Posterior combines both evidence types: every discordant pair contributes a uniform (German-tank) likelihood and every clipped read contributes a Gaussian likelihood centered on its offset

## Limitations

- Short read-length sequencing has a natural disadvantage for detecting structural variants, which may lead circlehunter to discover more computational sub-structures (e.g. COLO320DM)
- Current benchmarking strategy has limitations and accuracy for all types of focal amplifications cannot be guaranteed
- Rate of failures on complex structural variants (e.g. depth-balanced junction graphs detected by JaBbA or LINX) not yet explored

## Evidence

- [abstract] F1 at short read lengths: "circlehunter has an F1 score of 0.93 at 30× local depth and read lengths as short as 35 bp"
- [results] Bayesian breakpoint estimate: [evidence span withheld — anchor status: not_found]
- [abstract] tool identification: "We developed circlehunter ( https://github.com/suda-huanglab/circlehunter ), a tool for identifying ecDNA from ATAC-Seq data"
- [readme] pipeline execution: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [discussion] short-read caveat: "short read-length sequencing has a natural disadvantage for detecting structural variants"
