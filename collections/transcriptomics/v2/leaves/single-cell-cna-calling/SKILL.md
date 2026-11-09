---
name: single-cell-cna-calling
description: Use when analyzing PTA-amplified or low-coverage scWGS BAM files to call
  allele-specific CNAs, especially at small size scales (200 kb–5 Mb), or when comparing
  single-cell CNA callers on simulated or brain/meningioma datasets.
when_to_use_negative:
- Input is germline genotyping or sex chromosomes — SCAN2/HiScanner analysis is restricted
  to diploid autosomes 1–22
- The task is bulk-tumor CNA calling — BAF is a standard feature of bulk methods such
  as ASCAT and Battenberg; single-cell callers like HiScanner are tailored to scWGS
  profiles
- Only clonal CNAs at megabase scale in low-coverage cancer cells are needed — existing
  low-coverage callers (CHISEL, Ginkgo, SCOPE, SCYN) may suffice, though HiScanner
  is designed to extend resolution below 5 Mb
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: HiScanner
  role: Primary CNA caller combining RDR, BAF, and haplotype phasing; run via 'hiscanner
    init/run' CLI with normalize, segment, cnv, snp, phase, ado steps
  repo: https://github.com/parklab/HiScanner
- name: SCAN2
  role: Somatic SNV/indel genotyper that must be run separately before using HiScanner;
    its results feed the snp and phase pipeline steps
  repo: https://github.com/parklab/SCAN2
- name: CHISEL
  role: BAF-aware comparator single-cell CNA caller; its likelihood-based method is
    extended by HiScanner for ploidy/allelic copy-number inference
  repo: https://github.com/raphael-group/chisel
- name: Ginkgo
  role: BAF-agnostic comparator caller used to benchmark read-depth-only CNA calls
    and quantify BAF discordance on brain cell data
  repo: https://github.com/robertaboukhalil/ginkgo
- name: ViScanner
  role: Web-based visualization tool built on the HiGlass architecture for inspecting
    HiScanner CNA calls
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:b4efe468056d7db3848696c71c70617c5a48ca54d9ec4e499a759d74dd302b6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - CHISEL
  - Ginkgo
  - HiScanner
  - SCAN2
  - ViScanner
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-cna-calling

## Summary

Detect allele-specific copy number alterations (CNAs) in single-cell whole-genome sequencing (scWGS) data by combining read-depth ratios (RDR), B-allele frequencies (BAF), and reference-based phasing, as implemented in HiScanner, and benchmarked against BAF-agnostic callers like Ginkgo.

## When to use

Use when analyzing PTA-amplified or low-coverage scWGS BAM files to call allele-specific CNAs, especially at small size scales (200 kb–5 Mb), or when comparing single-cell CNA callers on simulated or brain/meningioma datasets.

## When NOT to use

- Input is germline genotyping or sex chromosomes — SCAN2/HiScanner analysis is restricted to diploid autosomes 1–22
- The task is bulk-tumor CNA calling — BAF is a standard feature of bulk methods such as ASCAT and Battenberg; single-cell callers like HiScanner are tailored to scWGS profiles
- Only clonal CNAs at megabase scale in low-coverage cancer cells are needed — existing low-coverage callers (CHISEL, Ginkgo, SCOPE, SCYN) may suffice, though HiScanner is designed to extend resolution below 5 Mb

## Inputs

- Single-cell scWGS BAM files (PTA-amplified or low-coverage, e.g. 65 brain cells or 179 meningioma cells)
- Matched bulk WGS BAMs for germline SNP calling and BAF phasing (GATK HaplotypeCaller + SHAPEIT2)
- Reference genome hg19/GRCh37+decoy and mappability tracks (hg19CRG.100bp or hg38/GRCh38 150mer, figshare DOI 10.6084/m9.figshare.28370357.v1)
- config.yaml and metadata table (bamID, bam, singlecell) for the HiScanner project
- SDX-simulated single-cell/bulk BAM pairs for benchmarking against truth events

## Outputs

- Per-cell integer allele-specific copy-number states and ploidy estimates
- Jointly segmented CNA calls with RDR and pBAF per bin (e.g. 100 kb bins)
- Per-condition precision/sensitivity summary tables versus simulated truth (bin size × coverage grid)
- Per-call BAF concordance/discordance tables when validating against other callers (e.g. ginkgo_baf_concordance.tsv)

## How to apply

Initialize a HiScanner project ('hiscanner init --output ./my_project'), populate config.yaml and a metadata table (bamID, bam, singlecell columns), and validate with 'hiscanner validate'. Run SCAN2 separately on the single-cell BAMs first, then run the pipeline ('hiscanner run --step all') to execute GC-corrected read-depth normalization (BICseq2-norm, uniquely mappable reads with MAPQ ≥ 30), joint multi-sample BIC-based segmentation, and likelihood-based inference of ploidy and integer allele-specific copy states extended from CHISEL. BAF estimation retains only phased gHETs with bulk VAF between 0.1 and 0.9, phased with SHAPEIT2 against the 1000 Genomes Phase 3 panel; a two-state HMM ('ADO'/'non-ADO') with Baum-Welch determines assay-specific allelic dropout and the optimal bin size (100 kb at 30× was optimal in SDX simulations). Restrict analysis to autosomes and remove single-cell CNA calls overlapping bulk CNA calls; validate by reproducing reported precision/sensitivity on simulated truth events or BAF-discordance fractions from Ginkgo comparisons.

## Related tools

- **HiScanner** (Primary CNA caller combining RDR, BAF, and haplotype phasing; run via 'hiscanner init/run' CLI with normalize, segment, cnv, snp, phase, ado steps) — https://github.com/parklab/HiScanner
- **SCAN2** (Somatic SNV/indel genotyper that must be run separately before using HiScanner; its results feed the snp and phase pipeline steps) — https://github.com/parklab/SCAN2
- **CHISEL** (BAF-aware comparator single-cell CNA caller; its likelihood-based method is extended by HiScanner for ploidy/allelic copy-number inference) — https://github.com/raphael-group/chisel
- **Ginkgo** (BAF-agnostic comparator caller used to benchmark read-depth-only CNA calls and quantify BAF discordance on brain cell data) — https://github.com/robertaboukhalil/ginkgo
- **ViScanner** (Web-based visualization tool built on the HiGlass architecture for inspecting HiScanner CNA calls)

## Examples

```
hiscanner init --output ./my_project && hiscanner validate && hiscanner run --step all
```

## Evaluation signals

- On SDX simulations at 30× coverage, precision/sensitivity must peak at 100 kb bins, matching the reported optimum in Supplementary Fig. 3 (0.5× coverage shows substantially lower performance)
- BAF-discordance audit of comparator calls must reproduce the reported values: 97% (581/599) discordant in oligodendrocytes and 66% (1410/2121) in neurons
- Called segments matched against simulated truth events must show greater sensitivity than CHISEL, Ginkgo, SCOPE, and SCYN at 200 kb–5 Mb size scales
- 'hiscanner validate' must pass on config.yaml and metadata table before the run; pipeline steps (normalize, segment, cnv, snp, phase, ado) complete without errors
- All retained gHETs have bulk VAF between 0.1 and 0.9, analysis restricted to autosomes, and single-cell calls overlapping bulk CNA calls removed

## Limitations

- Amplification artifacts from single-cell whole-genome amplification confound CNA detection by causing large read-depth fluctuations, even at high sequencing depth
- The model assumes independence between single-cell genomes, which may be violated for cells sharing clonal origins
- CHISEL comparisons required parameter fine-tuning; CHISEL showed high-ploidy predictions, overfitting, and strong GC bias at 100 kb bins due to lack of GC normalization
- No data accession numbers were provided for the brain scWGS or meningioma single-cell datasets, complicating direct reproduction

## Evidence

- [abstract] HiScanner combines read depth, B-allele frequency, and haplotype phasing to identify CNAs with high resolution: "we introduce HiScanner (High-resolution Single-Cell Allelic copy Number callER), which combines read depth, B-allele frequency, and haplotype phasing to identify CNAs with high resolution"
- [discussion] Only gHETs with bulk VAF 0.1–0.9 are retained: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [discussion] Read filtering: uniquely mappable, MAPQ ≥ 30: "Only uniquely mappable reads with a minimum mapping quality of 30 were considered"
- [methods] SCAN2 must run before HiScanner: "[SCAN2](https://github.com/parklab/SCAN2) needs to be run separately before using HiScanner."
- [readme] HiScanner pipeline steps in README: "hiscanner run --step ado      # ADO analysis to identify optimal bin size"
- [discussion] Ginkgo CLI source for benchmarking: "Ginkgo’s command line version was downloaded from https://github.com/robertaboukhalil/ginkgo"
