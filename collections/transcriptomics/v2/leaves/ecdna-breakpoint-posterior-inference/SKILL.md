---
name: ecdna-breakpoint-posterior-inference
description: Use when ATAC-Seq reads already support a candidate ecDNA junction (discordant
  pairs with anomalous orientation/insert size plus soft-clipped/supplementary-aligned
  pileups) and you need an exact breakpoint MAP estimate with a 95% CI, without full
  read realignment.
when_to_use_negative:
- When no candidate junction evidence exists yet (no discordant pairs or clipped-read
  pileups) — circlehunter may not identify the ecDNA without this information
- When the target is whole-genome-sequencing-based ecDNA reconstruction requiring
  copy-number input, e.g. AmpliconArchitect; this skill is specific to ATAC-Seq junction
  evidence
- For structural variants other than ecDNA-type focal amplifications — the authors
  cannot guarantee ecDNA call accuracy for all focal amplification classes
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: circlehunter
  role: Implements the integrated Bayesian breakpoint model and the ATAC-Seq ecDNA
    reconstruction pipeline driven by Snakemake
  repo: https://github.com/suda-huanglab/circlehunter
- name: ATAC-Seq
  role: Source assay whose enhanced chromatin accessibility over ecDNA supplies the
    discordant-pair and soft-clipped-read junction evidence for breakpoint estimation
- name: Snakemake
  role: Workflow engine that executes the circlehunter pipeline (Snakefile) with a
    yaml config for all samples and outputs
  repo: https://github.com/suda-huanglab/circlehunter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:36166b39c233b73defe00696ea4265cc308ed6432b151f7106ef47798b717af3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - ATAC-Seq
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# ecdna-breakpoint-posterior-inference

## Summary

A Bayesian grid-search procedure, as implemented in circlehunter, that reconstructs exact ecDNA breakpoints from ATAC-Seq junction evidence without read realignment, and quantifies how breakpoint confidence intervals widen as read length shrinks (down to 35 bp).

## When to use

Use when ATAC-Seq reads already support a candidate ecDNA junction (discordant pairs with anomalous orientation/insert size plus soft-clipped/supplementary-aligned pileups) and you need an exact breakpoint MAP estimate with a 95% CI, without full read realignment.

## When NOT to use

- When no candidate junction evidence exists yet (no discordant pairs or clipped-read pileups) — circlehunter may not identify the ecDNA without this information
- When the target is whole-genome-sequencing-based ecDNA reconstruction requiring copy-number input, e.g. AmpliconArchitect; this skill is specific to ATAC-Seq junction evidence
- For structural variants other than ecDNA-type focal amplifications — the authors cannot guarantee ecDNA call accuracy for all focal amplification classes

## Inputs

- Simulated or experimental ATAC-Seq reads (BAM) spanning a read-length grid down to 35 bp at 30x local depth
- Simulated breakpoint ground-truth coordinates per circle (from the task_001 simulation)
- circlehunter environment (environment.yml) and yaml config file describing samples and outputs

## Outputs

- breakpoint_ci_vs_readlength.csv tabulating MAP estimate and 95% CI width per replicate and read length
- fig2c_ci_width_vs_readlength.png (CI width versus read length, Fig. 2C-style plot)
- Per-circle breakpoint MAP positions with 95% confidence intervals

## How to apply

First extract junction evidence around each candidate breakpoint from the BAM: discordant read pairs with anomalous orientation or insert size define a fragment interval, and soft-clipped or supplementary-aligned reads provide offset positions at the junction. Build the breakpoint posterior by combining a uniform (German-tank) likelihood over the fragment interval spanned by each discordant pair with a Gaussian likelihood centered on each clipped-read offset. Grid-search the breakpoint position to obtain the MAP estimate and the 95% confidence interval per circle. Run this across the simulated read-length grid (down to 35 bp at 30x local depth) and tabulate MAP and 95% CI width per replicate and read length; the paper expects shorter read lengths to yield wider CIs (Fig. 2C). Validate against the reported robustness regime: F1 = 0.93 at 30x local depth with reads as short as 35 bp, with CI widths increasing monotonically as read length decreases.

## Related tools

- **circlehunter** (Implements the integrated Bayesian breakpoint model and the ATAC-Seq ecDNA reconstruction pipeline driven by Snakemake) — https://github.com/suda-huanglab/circlehunter
- **ATAC-Seq** (Source assay whose enhanced chromatin accessibility over ecDNA supplies the discordant-pair and soft-clipped-read junction evidence for breakpoint estimation)
- **Snakemake** (Workflow engine that executes the circlehunter pipeline (Snakefile) with a yaml config for all samples and outputs) — https://github.com/suda-huanglab/circlehunter

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- 95% CI widths increase monotonically as read length decreases (Fig. 2C expectation)
- F1 score ≈ 0.93 at 30x local depth with read lengths as short as 35 bp on simulated data
- MAP breakpoint estimates fall within the fragment interval defined by each supporting discordant read pair
- Gaussian clipped-read likelihoods center on the observed clipped-read offsets at the junction
- Pipeline completes via Snakemake without rule failures for every read length in the benchmark grid

## Limitations

- Short read-length sequencing has a natural disadvantage for detecting structural variants, which may lead circlehunter to discover more computational sub-structures (e.g. COLO320DM)
- The current benchmarking strategy has limitations and accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed
- Application to single-cell ATAC-Seq data has yet to be explored

## Evidence

- [results] The integrated Bayesian model estimates the exact breakpoint, and read length affects this estimate: shorter read lengths yield larger breakpoint confidence intervals (95% CI, Fig. 2C).: [evidence span withheld — anchor status: not_found]
- [abstract] circlehunter achieves F1 = 0.93 at 30x depth down to 35 bp reads: "circlehunter has an F1 score of 0.93 at 30× local depth and read lengths as short as 35 bp"
- [intro] circlehunter discovers junctions from supplementary alignment reads in ATAC-Seq data: "the Circle_finder method, which uses assay for transposase-accessible chromatin using sequencing (ATAC-Seq) data to discover junctions by collecting supplementary alignment reads"
- [discussion] circlehunter may miss ecDNA lacking the supporting signal: "may not be identified by circlehunter without this information"
- [readme] Pipeline is run with Snakemake and a yaml config file: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
