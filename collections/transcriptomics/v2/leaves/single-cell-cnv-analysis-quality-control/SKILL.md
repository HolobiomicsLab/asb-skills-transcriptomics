---
name: single-cell-cnv-analysis-quality-control
description: Use when preparing single-cell WGS (e.g., PTA- or MDA-amplified) data
  for CNA calling and an assay-specific minimum bin size must be chosen from measured
  allelic dropout, or when verifying that single-cell CNA calls pass QC thresholds
  (mappability, MAPQ, VAF filters).
when_to_use_negative:
- When the goal is CNA calling in bulk tumor data — tools like ASCAT and Battenberg
  already handle BAF there and no single-cell ADO calibration applies
- 'When input cells are expected aneuploid (e.g., tumor cells): the HMM must be trained
  on copy-neutral control cells, so aneuploid cells would corrupt the dropout-size
  distribution'
- When the amplification chemistry has already been characterized and a bin size fixed
  — re-running ADO calibration on unrelated cells adds no information
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: HiScanner
  role: Implements the `--step ado` two-state HMM that decodes allelic dropout events
    and derives the optimal bin size; its pipeline options also enforce RDR/BAF QC
    downstream
  repo: https://github.com/parklab/HiScanner
- name: SHAPEIT2
  role: Phases gHETs against the 1000 Genomes Phase 3 panel; only SHAPEIT2-phaseable
    gHETs with bulk VAF 0.1–0.9 are retained as HMM input
- name: BICseq2-norm
  role: Performs GC correction and binning at the ADO-derived optimal bin size on
    single-cell and bulk data
- name: GATK HaplotypeCaller
  role: Calls germline SNPs on bulk data (Best Practices, default parameters) that
    supply the gHET sites for BAF-based dropout detection
- name: SCAN2
  role: Must be run separately before HiScanner (snp/phase steps) so phased results
    exist for the ADO step
  repo: https://github.com/parklab/SCAN2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:5ad0461a43e125a9b41a02cd7823277254b69e4a5de6fb01396ea1c1464555c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - BICseq2-norm
  - GATK HaplotypeCaller
  - HiScanner
  - SCAN2
  - SHAPEIT2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-cnv-analysis-quality-control

## Summary

Quality-control workflow for single-cell CNV analysis that calibrates assay-specific allelic dropout (ADO) via a two-state HMM on phased BAF at germline heterozygous SNPs (gHETs), deriving the smallest permissible bin size that balances CNA detection resolution against amplification noise before running the HiScanner copy-number caller.

## When to use

Use when preparing single-cell WGS (e.g., PTA- or MDA-amplified) data for CNA calling and an assay-specific minimum bin size must be chosen from measured allelic dropout, or when verifying that single-cell CNA calls pass QC thresholds (mappability, MAPQ, VAF filters).

## When NOT to use

- When the goal is CNA calling in bulk tumor data — tools like ASCAT and Battenberg already handle BAF there and no single-cell ADO calibration applies
- When input cells are expected aneuploid (e.g., tumor cells): the HMM must be trained on copy-neutral control cells, so aneuploid cells would corrupt the dropout-size distribution
- When the amplification chemistry has already been characterized and a bin size fixed — re-running ADO calibration on unrelated cells adds no information

## Inputs

- Phased gHET VCF from SHAPEIT2 (phased_hets.vcf) called by GATK HaplotypeCaller on matched bulk WGS
- Single-cell BAM files with binarized BAF at gHET sites, restricted to uniquely mappable reads with MAPQ ≥ 30
- Control copy-neutral single cells per assay (PTA: 1278-Oligo-5; MDA: 1278_ct_p1G9) for HMM training

## Outputs

- Sample-specific ADO event length distribution (ado/dropout_size_distribution.tsv)
- Assay-specific smallest permissible bin size (95th percentile of dropout sizes; ~100 kb for PTA, ~10^5 bp-scale for MDA)
- Per-cell ADO/non-ADO state decodings at gHET sites

## How to apply

First, filter germline heterozygous SNPs: retain only gHETs phaseable by SHAPEIT2 with bulk variant allele frequency between 0.1 and 0.9, and binarize each cell's BAF at these sites. Then fit a two-state Gaussian HMM ('ADO' vs 'non-ADO') with Baum-Welch EM on control cells known to be copy-neutral (e.g., 1278-Oligo-5 for PTA; 1278_ct_p1G9 for MDA) using `hiscanner run --step ado`. Apply the fitted HMM to held-out single cells with Viterbi/posterior decoding, group consecutive ADO-state sites into dropout events, and compute the genomic-length distribution of these events. The 95th percentile of the dropout-size distribution sets the smallest permissible bin size (~100 kb expected for PTA; ~10^5 bp-scale events for MDA), which feeds downstream GC-corrected binning (BICseq2-norm) and joint segmentation. Because ADO is an amplification artifact that confounds CNA detection even at high depth, this calibration is assay-specific and must be re-run per amplification chemistry.

## Related tools

- **HiScanner** (Implements the `--step ado` two-state HMM that decodes allelic dropout events and derives the optimal bin size; its pipeline options also enforce RDR/BAF QC downstream) — https://github.com/parklab/HiScanner
- **SHAPEIT2** (Phases gHETs against the 1000 Genomes Phase 3 panel; only SHAPEIT2-phaseable gHETs with bulk VAF 0.1–0.9 are retained as HMM input)
- **BICseq2-norm** (Performs GC correction and binning at the ADO-derived optimal bin size on single-cell and bulk data)
- **GATK HaplotypeCaller** (Calls germline SNPs on bulk data (Best Practices, default parameters) that supply the gHET sites for BAF-based dropout detection)
- **SCAN2** (Must be run separately before HiScanner (snp/phase steps) so phased results exist for the ADO step) — https://github.com/parklab/SCAN2

## Examples

```
hiscanner run --step ado
```

## Evaluation signals

- ado/ results directory exists and contains dropout_size_distribution.tsv
- 95th-percentile dropout size is consistent with reported values (~100 kb for PTA; ~10^5 bp-scale for MDA)
- Retained gHETs are SHAPEIT2-phaseable with bulk VAF in [0.1, 0.9]; all reads used are uniquely mappable with MAPQ ≥ 30
- HMM converges via Baum-Welch EM on control cells without segmentation artifacts; decoded dropout events form a coherent length distribution
- Downstream binned RDR (BICseq2-norm) shows no strong GC bias at the chosen bin size, indicating the bin size is compatible with noise

## Limitations

- Only uniquely mappable reads with MAPQ ≥ 30 and autosomal chromosomes are considered, so sex chromosomes are excluded from this QC path
- ADO-derived bin size is assay-specific (PTA vs MDA differ by orders of magnitude) and cannot be transferred between amplification chemistries
- Single-cell CNA calls overlapping bulk CNA calls are removed and aneuploid cells excluded from CNA-length/prevalence analyses, limiting which events the QC'd caller can report

## Evidence

- [discussion] Two-state HMM training rationale: "we designed a HMM with two distinct states—“ADO” and “non-ADO”—and used Gaussian distribution probabilities to describe them"
- [discussion] gHET retention filter: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [methods] ADO step command: "hiscanner run --step ado      # ADO analysis to identify optimal bin size"
- [intro] Rationale for assay-specific bin size: "It also considers assay-specific ADO patterns to find an optimal bin size, a nuanced yet crucial parameter in CNA analysis"
- [readme] HiScanner demo project initialization: [evidence span withheld — anchor status: external_doc]
