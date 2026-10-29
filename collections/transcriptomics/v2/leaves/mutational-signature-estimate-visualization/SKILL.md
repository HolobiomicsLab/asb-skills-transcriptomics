---
name: mutational-signature-estimate-visualization
description: Use when you have run mm_fit_signatures on 96-channel trinucleotide catalogs
  and need to inspect per-sample signature contribution estimates or bootstrap confidence
  interval distributions before interpreting or reporting signature calls.
when_to_use_negative:
- Input is raw mutation calls or a 5-column VCF-like table that has not yet been fitted
  with mm_fit_signatures — visualization requires sig_out$estimate and sig_out$bootstrap
  objects
- Targeted capture panels covering a few megabases yield too low mutational burdens
  (< ~500 mutations) for meaningful per-patient signature fitting, so the resulting
  estimates/plots are unreliable
- De novo signature extraction is needed — this skill visualizes fits against reference
  signatures (e.g. COSMIC v3.1), not NMF-extracted signatures
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: mmsig
  role: Provides mm_fit_signatures (producing estimate and bootstrap objects), plot_signatures
    for signature profile plots, and bootSigsPlot for bootstrap confidence interval
    plots
  repo: github.com/evenrus/mmsig
- name: R
  role: Runtime environment in which mmsig visualization functions are invoked
- name: devtools
  role: Used to install the mmsig development version from GitHub
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:231efd75a62f87585c183d175941bba48267fd5b6cc0205760afae76c3f9cf78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - mmsig
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mutational-signature-estimate-visualization

## Summary

Visualize mutational signature contribution estimates and bootstrapped confidence intervals from an mmsig fit using plot_signatures and bootSigsPlot, enabling inspection of per-sample signature profiles and CI width as a QC step.

## When to use

Use when you have run mm_fit_signatures on 96-channel trinucleotide catalogs and need to inspect per-sample signature contribution estimates or bootstrap confidence interval distributions before interpreting or reporting signature calls.

## When NOT to use

- Input is raw mutation calls or a 5-column VCF-like table that has not yet been fitted with mm_fit_signatures — visualization requires sig_out$estimate and sig_out$bootstrap objects
- Targeted capture panels covering a few megabases yield too low mutational burdens (< ~500 mutations) for meaningful per-patient signature fitting, so the resulting estimates/plots are unreliable
- De novo signature extraction is needed — this skill visualizes fits against reference signatures (e.g. COSMIC v3.1), not NMF-extracted signatures

## Inputs

- sig_out$estimate — fitted per-sample signature contribution estimates from mm_fit_signatures
- sig_out$bootstrap — per-signature bootstrap contribution distributions from mm_fit_signatures
- sig_out$strand_bias_mm1 — transcriptional strand bias table (optional)
- signature reference matrix in sub/tri/96-channel format (e.g. signature_ref subset like SBS1, SBS2, SBS5, SBS8, SBS9, SBS13, SBS18, SBS-MM1)

## Outputs

- Per-sample 96-channel signature profile plots from plot_signatures
- Bootstrapped confidence interval plots per signature from bootSigsPlot
- Strand bias QC table with ratio, p_poisson, and MM1_flag columns

## How to apply

After running mm_fit_signatures (with parameters such as bootstrap=TRUE, iterations=1000 for stable results, cos_sim_threshold=0.01, refcheck=TRUE), pass sig_out$estimate to plot_signatures with samples=T and a chosen sig_order (e.g. SBS1, SBS2, SBS13, SBS5, SBS8, SBS9, SBS18, SBS-MM1, SBS35) to render per-sample 96-channel signature profiles. Pass sig_out$bootstrap to bootSigsPlot to display bootstrapped 95% confidence interval distributions per signature, which underpins mmsig's specificity rule that a signature is called present only when its 95% CI is non-zero. Use the bootstrap CI plot as a QC check — in the calibration analysis, the 95% CI enclosed the true pre-determined contribution ~94% of the time on simulated genomes. Optionally inspect sig_out$strand_bias_mm1 for transcriptional strand bias flags supporting SBS-MM1 calls.

## Related tools

- **mmsig** (Provides mm_fit_signatures (producing estimate and bootstrap objects), plot_signatures for signature profile plots, and bootSigsPlot for bootstrap confidence interval plots) — github.com/evenrus/mmsig
- **R** (Runtime environment in which mmsig visualization functions are invoked)
- **devtools** (Used to install the mmsig development version from GitHub)

## Examples

```
library(mmsig); plot_signatures(sig_out$estimate, samples=T, sig_order=c("SBS1","SBS2","SBS13","SBS5","SBS8","SBS9","SBS18","SBS-MM1","SBS35")); bootSigsPlot(sig_out$bootstrap)
```

## Evaluation signals

- plot_signatures renders 96-channel bar profiles per sample matching the sig_order specified (e.g. SBS1, SBS2, SBS13, SBS5, SBS8, SBS9, SBS18, SBS-MM1, SBS35)
- bootSigsPlot CI distributions are computed from ≥1000 bootstrap iterations (20 only for speed in demos); narrower/stable CIs indicate sufficient iterations
- On simulated genomes with known contributions, the 2.5th–97.5th percentile CI should enclose the true contribution ~94% of the time (paper's Results Fig. 6 calibration)
- Non-zero 95% CI in the bootstrap plot corresponds to mmsig calling the signature as present; zero-width CIs on low-contribution, less-distinctive signatures indicate error suppression
- Strand bias table shows plausible ratio/p_poisson values and MM1_flag markers consistent with transcription-coupled repair expectations for SBS-MM1

## Limitations

- Bootstrapping large datasets with many iterations significantly increases runtime (README: 'Bootstrapping large datasets with many iterations can significantly increase runtime')
- Flat mutational signatures may be falsely removed by mmsig because all mutations can be re-classified to another signature with relatively low cosine similarity cost
- With whole-exome sequencing, the expected signature profile in coding and non-coding regions can differ, affecting the displayed/estimated profiles
- Signature fitting accuracy is optimal only for catalogs of more than 500 mutations, consistently obtainable by WGS

## Evidence

- [readme] README plot usage: "plot_signatures(sig_out$estimate, samples = T, sig_order = c("SBS1", "SBS2", "SBS13", "SBS5", "SBS8", "SBS9", "SBS18", "SBS-MM1", "SBS35"))"
- [readme] README bootstrap CI plot: "bootSigsPlot(sig_out$bootstrap)"
- [readme] README runtime note: "Bootstrapping large datasets with many iterations can significantly increase runtime."
- [discussion] CI estimation and strand bias: "estimating 95% CI for the contribution of each mutational signature and checking for transcriptional strand bias typical of signatures associated with transcription-coupled repair"
- [discussion] CIs as specificity mechanism: "requiring non-zero 95% CI to call a mutational signature as present was highly effective at improving specificity"
- [methods] recommended iterations: "iterations = 20, # 1000 iterations recommended for stable results"
