---
name: allelic-dropout-modeling-single-cell-wgs
description: Use when a single-cell WGS CNA analysis must pick a per-dataset bin size
  that balances CNA detection resolution against amplification noise, e.g. before
  binning PTA- or MDA-amplified scWGS with HiScanner.
when_to_use_negative:
- No phased gHET calls or bulk VAFs are available (e.g., BAF-agnostic RDR-only pipelines
  or unphased bulk genotypes) — the HMM requires binarized BAF at SHAPEIT2-phased
  sites with bulk VAF 0.1–0.9
- The single-cell library uses a whole-genome amplification assay for which no matched
  control cells exist to retrain the ADO HMM — dropout size distributions are assay-specific
  (PTA ~100 kb vs MDA ~10^5 bp-scale)
- Input is low-coverage scWGS (e.g., DLP+, 10x Genomics) better suited to CHISEL/Alleloscope-style
  analysis, where dropout-length-based bin size selection at ~100 kb is not meaningful
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3474
tools:
- name: HiScanner
  role: Implements the two-state ADO/non-ADO Gaussian HMM and the 'hiscanner run --step
    ado' pipeline step that builds the dropout size distribution and derives the optimal
    bin size
  repo: https://github.com/parklab/HiScanner
- name: SCAN2
  role: Run separately before HiScanner; its results feed the snp/phase steps that
    supply the phased heterozygous sites consumed by ADO modeling
  repo: https://github.com/parklab/SCAN2
- name: SHAPEIT2
  role: Phases gHETs with the 1000 Genomes Phase 3 panel prior to BAF binarization;
    only SHAPEIT2-phased gHETs are retained
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:bfcdfdee83d3841b7f542f7f10e46d62b5b07c748e7f35cafdbc5bc62d65c0a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - HiScanner
  - SCAN2
  - SHAPEIT2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# allelic-dropout-modeling-single-cell-wgs

## Summary

Fits a two-state Gaussian hidden Markov model ('ADO' vs 'non-ADO') to binarized B-allele frequencies at phased germline heterozygous sites (gHETs) in control single cells, then uses the predicted allelic dropout event size distribution to set the smallest permissible CNA bin size per amplification assay in HiScanner.

## When to use

Use when a single-cell WGS CNA analysis must pick a per-dataset bin size that balances CNA detection resolution against amplification noise, e.g. before binning PTA- or MDA-amplified scWGS with HiScanner.

## When NOT to use

- No phased gHET calls or bulk VAFs are available (e.g., BAF-agnostic RDR-only pipelines or unphased bulk genotypes) — the HMM requires binarized BAF at SHAPEIT2-phased sites with bulk VAF 0.1–0.9
- The single-cell library uses a whole-genome amplification assay for which no matched control cells exist to retrain the ADO HMM — dropout size distributions are assay-specific (PTA ~100 kb vs MDA ~10^5 bp-scale)
- Input is low-coverage scWGS (e.g., DLP+, 10x Genomics) better suited to CHISEL/Alleloscope-style analysis, where dropout-length-based bin size selection at ~100 kb is not meaningful

## Inputs

- shapeit/phased_hets.vcf — gHETs phased by SHAPEIT2 using the 1000 Genomes Phase 3 integrated haplotype panel
- Bulk variant allele frequencies at gHETs from GATK HaplotypeCaller on matched bulk WGS
- Per-cell BAF at gHET sites from single-cell BAMs (uniquely mappable reads, MAPQ >= 30), binarized
- Control diploid cells for HMM training: 1278-Oligo-5 (PTA assay) or 1278_ct_p1G9 (MDA assay)

## Outputs

- ado/dropout_size_distribution.tsv — sample-specific dropout size distribution and derived 95th-percentile optimal (smallest permissible) bin size
- Per-cell decoded ADO/non-ADO state assignments at gHET sites
- List of predicted ADO events with genomic lengths

## How to apply

Parse the phased heterozygous gHET calls (shapeit/phased_hets.vcf), retaining only gHETs phased by SHAPEIT2 (1000 Genomes Phase 3 panel) whose bulk variant allele frequency falls between 0.1 and 0.9, and binarize each cell's BAF at these sites. Fit a two-state Gaussian HMM with 'ADO' and 'non-ADO' states via Baum-Welch expectation-maximization on assay-matched control cells — 1278-Oligo-5 for the PTA assay and 1278_ct_p1G9 for the MDA assay — using the hiscanner 'run --step ado' step. Apply the fitted HMM to held-out single cells, decode dropout states, and group consecutive ADO-state sites into dropout events. Compute the genomic-length distribution of predicted dropout events and take its 95th percentile as the smallest permissible bin size (~100 kb expected for PTA; ~10^5 bp-scale events expected for MDA); the rationale is that bins smaller than the dropout length scale would confuse true loss of heterozygosity with amplification artifacts.

## Related tools

- **HiScanner** (Implements the two-state ADO/non-ADO Gaussian HMM and the 'hiscanner run --step ado' pipeline step that builds the dropout size distribution and derives the optimal bin size) — https://github.com/parklab/HiScanner
- **SCAN2** (Run separately before HiScanner; its results feed the snp/phase steps that supply the phased heterozygous sites consumed by ADO modeling) — https://github.com/parklab/SCAN2
- **SHAPEIT2** (Phases gHETs with the 1000 Genomes Phase 3 panel prior to BAF binarization; only SHAPEIT2-phased gHETs are retained)

## Examples

```
hiscanner run --step ado
```

## Evaluation signals

- The ado/ results directory and ado/dropout_size_distribution.tsv exist after running 'hiscanner run --step ado'
- The 95th-percentile dropout event length (smallest permissible bin size) is consistent with the reported ~100 kb for PTA and ~10^5 bp-scale for MDA
- Input gHET set is restricted to SHAPEIT2-phased sites with bulk variant allele frequency between 0.1 and 0.9
- Baum-Welch fitting used only the designated control cells (1278-Oligo-5 for PTA; 1278_ct_p1G9 for MDA), with held-out cells only decoded
- Consecutive ADO-state sites are grouped into events and lengths are reported as genomic (bp) sizes

## Limitations

- The ADO size distribution is assay-specific; the HMM must be retrained on control cells when switching amplification chemistry (e.g., PTA vs MDA)
- The underlying model assumes independence between single-cell genomes, which may be violated for cells sharing clonal origins
- Analysis is restricted to autosomes (e.g., SCAN2 supports only diploid autosomes 1-22), and ADO-derived bin size assumes the control cells are diploid/balanced

## Evidence

- [discussion] Two-state HMM design: "we designed a HMM with two distinct states—“ADO” and “non-ADO”—and used Gaussian distribution probabilities to describe them"
- [discussion] gHET retention filter: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [methods] ADO pipeline step: "hiscanner run --step ado      # ADO analysis to identify optimal bin size"
- [intro] Assay-specific bin size rationale: "It also considers assay-specific ADO patterns to find an optimal bin size, a nuanced yet crucial parameter in CNA analysis"
