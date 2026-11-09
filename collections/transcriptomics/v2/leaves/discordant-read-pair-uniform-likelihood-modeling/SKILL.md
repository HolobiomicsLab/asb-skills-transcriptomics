---
name: discordant-read-pair-uniform-likelihood-modeling
description: Use when reconstructing ecDNA breakpoints from ATAC-Seq data where junction
  evidence consists of discordant read pairs with anomalous orientation/insert size
  and soft-clipped or supplementary-aligned read piles, and realignment is too costly.
when_to_use_negative:
- Input data lacks paired-end junction evidence (no discordant read pairs or clipped
  reads), e.g. single-end-only ATAC-Seq
- The target is general structural-variant discovery from WGS depth/copy-number data
  — use depth-balanced junction graph tools like JaBbA or LINX, or copy-number-based
  AmpliconArchitect, instead
- Focal amplifications with misregulated chromatin that are not ecDNA — the accessibility
  signal may yield false-positive circular calls
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: circlehunter
  role: Implements the integrated Bayesian breakpoint model that combines uniform
    discordant-pair likelihoods with clipped-read Gaussian offsets and grid-searches
    the MAP breakpoint
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine that executes the circlehunter pipeline rules producing the
    breakpoint outputs from the configured samples
  repo: https://snakemake.readthedocs.io/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:2ea4268f08cb156c59f5811f9a8e8d0fff1fac10d8aa2853a97c43cb26cfe3bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - Snakemake
  - circlehunter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# discordant-read-pair-uniform-likelihood-modeling

## Summary

A Bayesian breakpoint-estimation approach used by circlehunter to pinpoint the exact breakpoints of ecDNA fragments from ATAC-Seq discordant read pair enriched regions without time-consuming read realignment, combining a uniform (German-tank) likelihood over each discordant pair's fragment interval with a Gaussian likelihood on clipped-read offsets.

## When to use

Use when reconstructing ecDNA breakpoints from ATAC-Seq data where junction evidence consists of discordant read pairs with anomalous orientation/insert size and soft-clipped or supplementary-aligned read piles, and realignment is too costly.

## When NOT to use

- Input data lacks paired-end junction evidence (no discordant read pairs or clipped reads), e.g. single-end-only ATAC-Seq
- The target is general structural-variant discovery from WGS depth/copy-number data — use depth-balanced junction graph tools like JaBbA or LINX, or copy-number-based AmpliconArchitect, instead
- Focal amplifications with misregulated chromatin that are not ecDNA — the accessibility signal may yield false-positive circular calls

## Inputs

- ATAC-Seq reads (simulated or real), spanning a read-length grid down to 35 bp at ~30x local depth
- Discordant read pairs with anomalous orientation/insert size near candidate breakpoints
- Soft-clipped and supplementary-aligned read pileups at junction sites
- Known or simulated breakpoint positions (for interval likelihood construction)

## Outputs

- MAP breakpoint estimates per ecDNA/simulated circle
- 95% confidence intervals per breakpoint
- breakpoint_ci_vs_readlength.csv (MAP and CI width per replicate and read length)
- Fig. 2C-style plot of 95% CI width versus read length (fig2c_ci_width_vs_readlength.png)

## How to apply

First extract junction evidence around each candidate breakpoint: discordant read pairs with anomalous orientation or insert size, plus soft-clipped/supplementary-aligned reads piling up at the junction. Build the breakpoint posterior by combining a uniform (German-tank) likelihood over the fragment interval spanned by each discordant pair with a Gaussian likelihood centered on each clipped-read offset. Grid-search the breakpoint position to obtain the MAP estimate and the 95% confidence interval per simulated circle. Note that read length modulates precision: shorter read lengths (down to 35 bp at 30x local depth) yield wider 95% CIs, so expect monotonic CI inflation as read length decreases. Tabulate MAP and 95% CI width per replicate and read length (e.g. breakpoint_ci_vs_readlength.csv) and plot CI width versus read length as a Fig. 2C-style check.

## Related tools

- **circlehunter** (Implements the integrated Bayesian breakpoint model that combines uniform discordant-pair likelihoods with clipped-read Gaussian offsets and grid-searches the MAP breakpoint) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine that executes the circlehunter pipeline rules producing the breakpoint outputs from the configured samples) — https://snakemake.readthedocs.io/

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- 95% CI widths increase monotonically as read length decreases (down to 35 bp), reproducing the Fig. 2C regime
- Robustness regime holds: F1 = 0.93 at 30x local depth with reads as short as 35 bp on simulated data
- MAP breakpoint estimates fall within the fragment interval spanned by each supporting discordant read pair
- 95% CI width is inversely related to read length — shorter reads yield larger confidence intervals
- Pipeline completes end-to-end via the Snakemake rulegraph for all samples in the config.yaml

## Limitations

- Current benchmarking strategy has limitations; accuracy of ecDNA calls cannot be guaranteed for all types of focal amplifications
- Short read-length sequencing has a natural disadvantage for structural-variant detection and may cause circlehunter to discover more computational sub-structures (e.g. COLO320DM)
- Other focal amplification classes with misregulated chromatin may be detected as false-positive ecDNA

## Evidence

- [results] The integrated Bayesian model estimates the exact breakpoint, and read length affects this estimate: shorter read lengths yield larger breakpoint confidence intervals (95% CI, Fig. 2C).: [evidence span withheld — anchor status: not_found]
- [abstract] circlehunter achieves an F1 score of 0.93 at 30x local depth with read lengths as short as 35 bp on simulated data: "circlehunter has an F1 score of 0.93 at 30× local depth and read lengths as short as 35 bp"
- [readme] circlehunter pipeline executed via Snakemake: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [intro] circlehunter leverages increased ATAC-Seq signals from ecDNA accessibility: "we developed circlehunter to reconstruct ecDNA by taking full advantage of the increased ATAC-Seq signals arising from the high accessibility of ecDNA itself."
