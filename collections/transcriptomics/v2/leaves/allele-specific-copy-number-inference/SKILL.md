---
name: allele-specific-copy-number-inference
description: Use when analyzing high-coverage (>20x) PTA-amplified single-cell WGS
  data where small (100 kb–Mb scale) somatic CNAs must be called with allele-specific
  copy states, and paired bulk WGS is available for phasing.
when_to_use_negative:
- Low-coverage scWGS data from DLP+ or 10x Genomics — callers like CHISEL or Alleloscope
  are designed for that regime, not high-coverage allele-specific inference
- Sex chromosomes — SCAN2 explicitly supports only diploid autosomes 1-22 (analysis
  is restricted to autosomes)
- Data lacking matched bulk WGS for germline SNP calling and reference-based phasing,
  since BAF estimation depends on phased gHETs
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: HiScanner
  role: 'Core caller: performs ADO-based bin-size selection, GC-corrected binning,
    joint segmentation, and joint RDR/BAF likelihood modeling for allele-specific
    copy states'
  repo: https://github.com/parklab/HiScanner
- name: SCAN2
  role: Must be run separately before HiScanner to genotype/prepare germline SNP results
    used for phasing
  repo: https://github.com/parklab/SCAN2
- name: CHISEL
  role: Benchmarked comparator; HiScanner extends CHISEL's likelihood-based method
    to infer ploidy and allelic copy number states
  repo: https://github.com/raphael-group/chisel
- name: Ginkgo
  role: Benchmarked BAF-agnostic single-cell CNA comparator
  repo: https://github.com/robertaboukhalil/ginkgo
- name: BICseq2
  role: Source of the 100mer mappability track, the BICseq2-norm GC correction, and
    the single-sample segmentation algorithm extended to multi-sample joint segmentation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:3bb4413ea10401213d5c8d8686cb446e57c8dbd562916cb938ca47314063e767
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - BICseq2
  - CHISEL
  - Ginkgo
  - HiScanner
  - R
  - SCAN2
  - bcftools
  - samtools
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# allele-specific-copy-number-inference

## Summary

Infer integer allele-specific copy number states and cell ploidy from high-coverage scWGS by jointly modeling read depth ratios (RDR) and B-allele frequencies (BAF) at phased germline heterozygous SNPs, as implemented in HiScanner.

## When to use

Use when analyzing high-coverage (>20x) PTA-amplified single-cell WGS data where small (100 kb–Mb scale) somatic CNAs must be called with allele-specific copy states, and paired bulk WGS is available for phasing.

## When NOT to use

- Low-coverage scWGS data from DLP+ or 10x Genomics — callers like CHISEL or Alleloscope are designed for that regime, not high-coverage allele-specific inference
- Sex chromosomes — SCAN2 explicitly supports only diploid autosomes 1-22 (analysis is restricted to autosomes)
- Data lacking matched bulk WGS for germline SNP calling and reference-based phasing, since BAF estimation depends on phased gHETs

## Inputs

- High-coverage (e.g., >20x) PTA-amplified single-cell WGS BAM files
- Paired bulk whole-genome sequencing BAM files per individual
- Split hg19/GRCh37+decoy reference genome FASTA
- hg19 100mer mappability track from BICseq2 (hg19CRG.100bp)
- SCAN2 output (germline SNP calls phased by SHAPEIT2 with 1000 Genomes Phase 3 panel, field 'phasedgt')
- config.yaml and metadata.txt specifying sample paths and bin size (e.g., 100 kb)

## Outputs

- Per-cell allele-specific integer copy number states and ploidy calls
- Joint segmentation of single-cell RDR profiles (BIC-based multi-sample BICseq2-seg extension)
- Summary table: CNA count and size distribution, FDR estimate, per-cell-type CNA fractions, clonal vs subclonal gain/loss split with odds ratio, cnLOH counts
- GC-corrected, binned read depth ratios and bin-wise pBAF profiles

## How to apply

First prepare SCAN2 prerequisites per individual: call germline SNPs on bulk with GATK HaplotypeCaller, phase gHETs with SHAPEIT2 against the 1000 Genomes Phase 3 panel retaining only gHETs with bulk VAF 0.1–0.9, and ensure the phased genotype field is named 'phasedgt'. Initialize the HiScanner project (hiscanner init, edit config.yaml with single-cell and bulk BAM paths, split hg19 reference, 100mer mappability track, 100 kb bins, autosomes only) and run hiscanner validate. Then run the pipeline steps: --step ado fits a two-state ADO/non-ADO HMM (Baum-Welch, Gaussian emissions) to select the optimal bin size from assay-specific allelic dropout; --step normalize bins uniquely mappable reads at MAPQ >= 30 and applies BICseq2-norm-style GC correction; --step segment jointly segments all single-cell RDR profiles with the BIC-based multi-sample extension of BICseq2-seg; --step cnv infers cell ploidy and integer allele-specific copy states by joint RDR/BAF likelihood modeling (extending CHISEL's likelihood approach). Finally remove single-cell calls overlapping bulk CNA calls and exclude aneuploid cells from size/prevalence statistics.

## Related tools

- **HiScanner** (Core caller: performs ADO-based bin-size selection, GC-corrected binning, joint segmentation, and joint RDR/BAF likelihood modeling for allele-specific copy states) — https://github.com/parklab/HiScanner
- **SCAN2** (Must be run separately before HiScanner to genotype/prepare germline SNP results used for phasing) — https://github.com/parklab/SCAN2
- **CHISEL** (Benchmarked comparator; HiScanner extends CHISEL's likelihood-based method to infer ploidy and allelic copy number states) — https://github.com/raphael-group/chisel
- **Ginkgo** (Benchmarked BAF-agnostic single-cell CNA comparator) — https://github.com/robertaboukhalil/ginkgo
- **BICseq2** (Source of the 100mer mappability track, the BICseq2-norm GC correction, and the single-sample segmentation algorithm extended to multi-sample joint segmentation)

## Examples

```
hiscanner init && hiscanner validate && hiscanner run --step ado && hiscanner run --step normalize --use-cluster && hiscanner run --step segment && hiscanner run --step cnv
```

## Evaluation signals

- hiscanner validate passes after config.yaml setup (paths, mappability track, bin size, autosomes only)
- Simulation benchmark shows HiScanner outperforms CHISEL, Ginkgo, SCOPE, and SCYN at small CNA sizes (200 kb–5 Mb)
- Application to 65 neurotypical brain cells recovers ≥1 CNA in 77% (24/31) of neurons and 53% (17/32) of oligodendrocytes
- Single-cell CNA calls overlapping bulk CNA calls are removed and aneuploid cells are excluded before computing CNA size/prevalence statistics
- Reported FDR estimate is conservative and increases as CNA size decreases

## Limitations

- The model assumes independence between single-cell genomes, which may be violated for cells sharing clonal origins
- Clonal classification may conflate true clonal expansions with independent recurrent mutations sharing identical coordinates
- Amplification artifacts in single-cell whole-genome amplification can confound CNA detection even at high sequencing depth

## Evidence

- [abstract] Combines RDR and phased BAF for allele-specific calls: "we introduce HiScanner (High-resolution Single-Cell Allelic copy Number callER), which combines read depth, B-allele frequency, and haplotype phasing to identify CNAs with high resolution"
- [intro] Joint modeling of RDR/BAF infers ploidy and integer copy states: "By simultaneously modeling RDR and BAF for individual cells, HiScanner infers cell ploidy and integer allele-specific copy number states"
- [discussion] ADO HMM selects optimal bin size: "we designed a HMM with two distinct states—“ADO” and “non-ADO”—and used Gaussian distribution probabilities to describe them"
- [discussion] gHET VAF filter for phasing: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [discussion] MAPQ and mappability filter: "Only uniquely mappable reads with a minimum mapping quality of 30 were considered"
- [methods] SCAN2 prerequisite for HiScanner: "[SCAN2](https://github.com/parklab/SCAN2) needs to be run separately before using HiScanner."
- [readme] Demo pipeline invocation: [evidence span withheld — anchor status: external_doc]
- [discussion] Exclusion of aneuploid cells from downstream statistics: "we excluded the aneuploid neurons from subsequent analyses of CNA length and prevalence"
