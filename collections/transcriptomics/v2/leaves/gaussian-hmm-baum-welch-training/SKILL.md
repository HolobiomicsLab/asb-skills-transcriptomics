---
name: gaussian-hmm-baum-welch-training
description: 'Use when a single-cell WGS analysis must pick a data-driven bin size
  before segmentation: specifically, when gHET calls phased by SHAPEIT2 are available
  and the amplification assay''s ADO length distribution must be measured to set the
  minimum resolvable CNA bin.'
when_to_use_negative:
- The amplification assay is uniform enough that ADO is negligible and a default bin
  size is acceptable without assay-specific calibration
- No phased gHET calls exist (phasing pipeline not run), since the HMM operates on
  phased, bulk-VAF-filtered (0.1–0.9) heterozygous sites
- Applying this to sex chromosomes or non-autosomal regions, since the analysis is
  restricted to diploid autosomes
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HiScanner
  role: Runs the ADO-analysis step that trains the two-state Gaussian HMM via Baum-Welch
    and derives the optimal bin size
  repo: https://github.com/parklab/HiScanner
- name: SCAN2
  role: Must be run separately before HiScanner (provides upstream genotyping results
    processed by the --step snp/--step phase steps)
  repo: https://github.com/parklab/SCAN2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:885631ee6df101c702797e5652c0074ad6c93833c0f119733247c4c2bc081af3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - HiScanner
  - SCAN2
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Gaussian HMM with Baum-Welch training

## Summary

Fits a two-state Gaussian hidden Markov model ('ADO'/'non-ADO') on binned B-allele frequencies at phased germline heterozygous SNPs using Baum-Welch expectation-maximization, so HiScanner can characterize assay-specific allelic dropout events and derive the smallest permissible bin size (the 95th percentile of predicted dropout event lengths).

## When to use

Use when a single-cell WGS analysis must pick a data-driven bin size before segmentation: specifically, when gHET calls phased by SHAPEIT2 are available and the amplification assay's ADO length distribution must be measured to set the minimum resolvable CNA bin.

## When NOT to use

- The amplification assay is uniform enough that ADO is negligible and a default bin size is acceptable without assay-specific calibration
- No phased gHET calls exist (phasing pipeline not run), since the HMM operates on phased, bulk-VAF-filtered (0.1–0.9) heterozygous sites
- Applying this to sex chromosomes or non-autosomal regions, since the analysis is restricted to diploid autosomes

## Inputs

- Phased germline heterozygous SNP calls (shapeit/phased_hets.vcf) from bulk sequencing, phased by SHAPEIT2 with the 1000 Genomes Phase 3 panel
- Single-cell BAF values at gHET sites for control cells (e.g., 1278-Oligo-5 for PTA; 1278_ct_p1G9 for MDA) plus held-out cells
- HiScanner project config (config.yaml) and metadata.txt from hiscanner init

## Outputs

- Fitted two-state Gaussian HMM ('ADO'/'non-ADO') parameters
- Per-cell ADO-state decodings grouped into dropout events
- Dropout-size distribution and 95th-percentile optimal bin size written to ado/dropout_size_distribution.tsv

## How to apply

Parse the phased gHET calls (shapeit/phased_hets.vcf), retaining only gHETs phased by SHAPEIT2 with bulk variant allele frequency between 0.1 and 0.9, and binarize each cell's BAF at these sites. Fit a two-state Gaussian HMM with 'ADO' and 'non-ADO' states via Baum-Welch expectation-maximization on a control cell (1278-Oligo-5 for the PTA assay; 1278_ct_p1G9 for the MDA assay) using `hiscanner run --step ado`. Decode dropout states in held-out single cells (Viterbi/posterior decoding) and group consecutive ADO-state sites into dropout events. Compute the genomic-length distribution of predicted dropout events and take the 95th percentile as the smallest permissible bin size (~100 kb expected for PTA; ~10^5 bp-scale events for MDA). The rationale is that amplification artifacts cause allelic dropout that would masquerade as copy-number signal at bin sizes below the dropout event scale, so bin size must exceed the assay's dropout footprint.

## Related tools

- **HiScanner** (Runs the ADO-analysis step that trains the two-state Gaussian HMM via Baum-Welch and derives the optimal bin size) — https://github.com/parklab/HiScanner
- **SCAN2** (Must be run separately before HiScanner (provides upstream genotyping results processed by the --step snp/--step phase steps)) — https://github.com/parklab/SCAN2

## Examples

```
hiscanner run --step ado
```

## Evaluation signals

- The ado/ results directory and ado/dropout_size_distribution.tsv exist after `hiscanner run --step ado`
- The derived 95th-percentile bin size is consistent with reported values: ~100 kb for the PTA assay and ~10^5 bp-scale events for the MDA assay
- HMM fitting uses only gHETs phased by SHAPEIT2 with bulk VAF in [0.1, 0.9]; verify the retained-site count matches this filter
- The fitted model has exactly two Gaussian-emission states ('ADO' and 'non-ADO') trained on the designated control cell (1278-Oligo-5 for PTA, 1278_ct_p1G9 for MDA)
- Decoded dropout events in held-out cells form a coherent genomic-length distribution rather than degenerate single-site events

## Limitations

- The derived bin size is assay-specific — a dropout distribution trained on PTA control cells (1278-Oligo-5) is not transferable to MDA or other whole-genome amplification chemistries
- State decoding depends on binarized BAF at gHET sites, so sparse gHET coverage or unphased SNPs reduce the reliability of dropout-event length estimates
- Even at the derived optimal bin size, PTA amplification artifacts can still confound CNA detection through large read-depth fluctuations

## Evidence

- [discussion] Two-state HMM design: "we designed a HMM with two distinct states—“ADO” and “non-ADO”—and used Gaussian distribution probabilities to describe them"
- [discussion] gHET retention filter: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [intro] Purpose: optimal bin size: "It also considers assay-specific ADO patterns to find an optimal bin size, a nuanced yet crucial parameter in CNA analysis"
- [methods] CLI invocation: "hiscanner run --step ado      # ADO analysis to identify optimal bin size"
