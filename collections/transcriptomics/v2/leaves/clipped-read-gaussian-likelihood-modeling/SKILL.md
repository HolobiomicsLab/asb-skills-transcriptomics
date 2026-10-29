---
name: clipped-read-gaussian-likelihood-modeling
description: Use when ATAC-Seq junction evidence (discordant read pairs plus soft-clipped/supplementary-aligned
  reads) must be converted into an exact ecDNA breakpoint posterior, e.g. to quantify
  how 95% CI width scales with read length down to 35 bp.
when_to_use_negative:
- The input is raw FASTQ or an unaligned ATAC-Seq library with no junction-level evidence
  (disc read pairs or clipped reads) to anchor the likelihoods.
- The goal is ecDNA detection itself rather than breakpoint localization — run the
  full circlehunter Snakemake pipeline first and only apply this model to its junction
  regions.
- Structural variants are expected to be much larger or more complex than what read-pair/clip
  evidence can bracket (e.g. COLO320DM-like sub-structures), where short-read likelihoods
  fragment into computational sub-structures.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: circlehunter
  role: 'Implements the Bayesian breakpoint model: combines discordant-pair interval
    likelihood with clipped-read Gaussian offsets and grid-searches for the MAP breakpoint
    without read realignment'
  repo: https://github.com/suda-huanglab/circlehunter
- name: ATAC-Seq
  role: 'Assay supplying the junction evidence: enhanced chromatin accessibility of
    ecDNA yields soft-clipped/supplementary-aligned reads piling up at ecDNA breakpoints'
- name: Snakemake
  role: Workflow engine that runs the circlehunter rules over samples defined in the
    yaml config to produce the breakpoint estimates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:d5e4b392b1702bdf77297f2e28b23354bab1a87ff0c2ee5bca469341a5a3eeb4
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# clipped-read-gaussian-likelihood-modeling

## Summary

A Bayesian breakpoint-estimation approach used by circlehunter that combines a uniform likelihood over the fragment interval spanned by each discordant read pair with a Gaussian likelihood centered on each soft-clipped/supplementary-aligned read offset, then grid-searches the breakpoint to obtain MAP estimates and 95% confidence intervals without read realignment.

## When to use

Use when ATAC-Seq junction evidence (discordant read pairs plus soft-clipped/supplementary-aligned reads) must be converted into an exact ecDNA breakpoint posterior, e.g. to quantify how 95% CI width scales with read length down to 35 bp.

## When NOT to use

- The input is raw FASTQ or an unaligned ATAC-Seq library with no junction-level evidence (disc read pairs or clipped reads) to anchor the likelihoods.
- The goal is ecDNA detection itself rather than breakpoint localization — run the full circlehunter Snakemake pipeline first and only apply this model to its junction regions.
- Structural variants are expected to be much larger or more complex than what read-pair/clip evidence can bracket (e.g. COLO320DM-like sub-structures), where short-read likelihoods fragment into computational sub-structures.

## Inputs

- Simulated ATAC-Seq read alignments (BAM) at 30x local depth spanning a benchmark read-length grid down to 35 bp
- Simulated breakpoint ground-truth positions per circle
- Junction evidence records: discordant read-pair fragment intervals and soft-clipped/supplementary-aligned read offsets

## Outputs

- breakpoint_ci_vs_readlength.csv with MAP breakpoint and 95% CI width per replicate and read length
- fig2c_ci_width_vs_readlength.png (95% CI width versus read length)
- Per-circle breakpoint posterior with MAP estimate

## How to apply

Extract junction evidence around each candidate breakpoint from the ATAC-Seq alignment (BAM): discordant read pairs with anomalous orientation/insert size define a uniform ('German-tank') likelihood over the fragment interval they span, while each soft-clipped or supplementary-aligned read contributes a Gaussian likelihood centered on its offset at the junction. Multiply these likelihoods with the prior into a breakpoint posterior and grid-search the breakpoint position to find the MAP estimate. Report the 95% confidence interval from the posterior for every simulated circle. Read length directly controls precision: shorter reads (down to 35 bp at 30x local depth) yield wider intervals, so tabulate MAP and 95% CI width per replicate and read length (breakpoint_ci_vs_readlength.csv) and render the Fig. 2C-style CI-width-vs-read-length plot (fig2c_ci_width_vs_readlength.png). Validate by checking that 95% CI widths increase monotonically as read length decreases while detection quality stays robust (F1 = 0.93 at 30x, 35 bp).

## Related tools

- **circlehunter** (Implements the Bayesian breakpoint model: combines discordant-pair interval likelihood with clipped-read Gaussian offsets and grid-searches for the MAP breakpoint without read realignment) — https://github.com/suda-huanglab/circlehunter
- **ATAC-Seq** (Assay supplying the junction evidence: enhanced chromatin accessibility of ecDNA yields soft-clipped/supplementary-aligned reads piling up at ecDNA breakpoints)
- **Snakemake** (Workflow engine that runs the circlehunter rules over samples defined in the yaml config to produce the breakpoint estimates)

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- 95% CI widths increase monotonically as simulated read length decreases (down to 35 bp), reproducing the Fig. 2C trend.
- MAP breakpoint falls within the fragment interval spanned by each contributing discordant read pair.
- Detection quality in the robustness regime matches the reported F1 = 0.93 at 30x local depth with 35 bp reads.
- Posterior grid-search reproduces the paper's exact-breakpoint estimates without any read realignment step.
- Output table breakpoint_ci_vs_readlength.csv contains MAP and 95% CI width for every replicate x read-length combination.

## Limitations

- Short read lengths have a natural disadvantage for detecting structural variants and may cause circlehunter to split ecDNA into more computational sub-structures.
- The current benchmarking strategy has limitations, and accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed; failure rates on complex structural variants are unexplored.
- The model relies on supplementary-alignment/clipped-read information at the junction; without it, breakpoints may not be identified.
