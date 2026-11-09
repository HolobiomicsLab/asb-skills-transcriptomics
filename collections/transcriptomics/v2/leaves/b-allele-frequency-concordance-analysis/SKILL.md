---
name: b-allele-frequency-concordance-analysis
description: Use when validating single-cell CNA calls (RDR-based segmentation) by
  requiring concordant BAF shifts at phased gHETs, or when comparing allele-aware
  vs. BAF-agnostic CNA callers.
when_to_use_negative:
- Data are low-coverage (<0.1x per cell) with too few heterozygous SNPs per bin for
  stable BAF estimation, where BAF-agnostic RDR-only callers like Ginkgo or SCYN are
  the intended mode
- No matched bulk sample or reference panel is available for phasing gHETs, so BAF
  cannot be reliably computed
- Calls on sex chromosomes — SCAN2/HiScanner analyses are restricted to diploid autosomes
  1-22
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HiScanner
  role: Jointly models RDR and phased BAF per cell to infer ploidy and allele-specific
    copy-number states; provides the BAF concordance output
  repo: github.com/parklab/HiScanner
- name: SCAN2
  role: Run separately before HiScanner; its SNP/genotype output feeds the phasing
    step that enables pBAF computation
  repo: github.com/parklab/SCAN2
- name: CHISEL
  role: BAF-aware single-cell comparator whose likelihood method HiScanner extends;
    also used as a competing caller (100 kb and 5 Mb bin settings)
  repo: github.com/raphael-group/chisel
- name: Ginkgo
  role: BAF-agnostic RDR-based comparator run on the same single-cell BAM for benchmarking
    breakpoint density
  repo: github.com/robertaboukhalil/ginkgo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:bbd357f4156abf77e2630a3425f5c315dff5058b3e27f7946d686a769d1da12b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHISEL
  - Ginkgo
  - HiScanner
  - SCAN2
  - SCOPE
  - SCYN
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# B-allele frequency concordance analysis

## Summary

Assess copy-number calls by checking whether read-depth-based losses and gains are corroborated by shifts in B-allele frequency (BAF) at phased germline heterozygous SNPs, as done in HiScanner for allele-specific CNA validation.

## When to use

Use when validating single-cell CNA calls (RDR-based segmentation) by requiring concordant BAF shifts at phased gHETs, or when comparing allele-aware vs. BAF-agnostic CNA callers.

## When NOT to use

- Data are low-coverage (<0.1x per cell) with too few heterozygous SNPs per bin for stable BAF estimation, where BAF-agnostic RDR-only callers like Ginkgo or SCYN are the intended mode
- No matched bulk sample or reference panel is available for phasing gHETs, so BAF cannot be reliably computed
- Calls on sex chromosomes — SCAN2/HiScanner analyses are restricted to diploid autosomes 1-22

## Inputs

- Single-cell scWGS BAM (e.g., PTA-amplified neuron 5823PFC-B at ~29x)
- Matched bulk whole-genome sequencing BAM for germline SNP calling
- Phased gHET list from GATK HaplotypeCaller + SHAPEIT2 (1000 Genomes Phase 3 panel)
- Binned per-cell RDR track (e.g., BICseq2-norm GC-corrected bins)
- Mappability track (hg19CRG.100bp or hg38 150mer, figshare DOI 10.6084/m9.figshare.28370357.v1)

## Outputs

- Bin-wise phased BAF (pBAF) profile per cell
- Allele-specific integer copy-number calls per bin (CN states a, b, and ploidy)
- Set of RDR-based loss calls with concordant BAF shifts, with length distribution statistics
- Comparison table of breakpoint density per caller (e.g., CHISEL 10,322 breakpoints; 0.38 per 100 kb)

## How to apply

First call germline heterozygous SNPs on the matched bulk WGS with GATK HaplotypeCaller, phase them with SHAPEIT2 against the 1000 Genomes Phase 3 panel, and retain only gHETs phasable by SHAPEIT2 with bulk VAF between 0.1 and 0.9. Compute bin-wise phased BAF (pBAF) restricted to autosomal chromosomes and uniquely mappable reads with mapping quality ≥ 30. Model RDR and BAF jointly per cell — HiScanner extends CHISEL's likelihood-based method with a GMM on 'balanced' bins to infer ploidy and integer allele-specific copy states — so a called loss should show the expected BAF departure from 0.5 in the predicted direction. When benchmarking callers, tabulate the length distribution of loss calls that carry concordant BAF evidence; in the aneuploid neuron 5823PFC-B comparison, HiScanner's loss calls showed the expected BAF shifts while CHISEL over-segmented (10,322 breakpoints).

## Related tools

- **HiScanner** (Jointly models RDR and phased BAF per cell to infer ploidy and allele-specific copy-number states; provides the BAF concordance output) — github.com/parklab/HiScanner
- **SCAN2** (Run separately before HiScanner; its SNP/genotype output feeds the phasing step that enables pBAF computation) — github.com/parklab/SCAN2
- **CHISEL** (BAF-aware single-cell comparator whose likelihood method HiScanner extends; also used as a competing caller (100 kb and 5 Mb bin settings)) — github.com/raphael-group/chisel
- **Ginkgo** (BAF-agnostic RDR-based comparator run on the same single-cell BAM for benchmarking breakpoint density) — github.com/robertaboukhalil/ginkgo

## Examples

```
hiscanner run --step phase  # process SCAN2 results to obtain phased gHETs and bin-wise pBAF for RDR+BAF joint CNA calling
```

## Evaluation signals

- Called loss regions show a directional BAF shift away from 0.5 at phased gHETs; concordant HiScanner loss calls exhibit the expected BAF shifts (Fig. 3 validation criterion)
- All retained gHETs have bulk VAF within 0.1–0.9 and are phasable by SHAPEIT2
- Only uniquely mappable reads with MAPQ ≥ 30 (SCOPE comparison used MAPQ 40) contribute to bin counts
- Breakpoint density table per method reproduces the ordering where CHISEL is most fragmented (10,322 breakpoints, 0.38 per 100 kb) and HiScanner has fewer spurious breakpoints
- Analysis is restricted to autosomes; centromeric-region calls are stable across callers

## Limitations

- CHISEL showed a tendency for high-ploidy predictions and overfitting, plus strong GC bias in its 100 kb-bin RDR due to lack of GC normalization, complicating direct BAF-based comparison
- HiScanner's model assumes independence between single-cell genomes, which is violated for cells sharing clonal origins
- Phasing depends on matched bulk data and the 1000 Genomes panel; only autosomal chromosomes are supported

## Evidence

- [intro] Reference-based phasing of gHETs and BAF estimation complement RDR: "HiScanner leverages reference-based phasing and local sequence bias to efficiently estimate BAF to complement read depth-based CNA identification"
- [intro] Joint RDR+BAF modeling yields integer allele-specific states: "By simultaneously modeling RDR and BAF for individual cells, HiScanner infers cell ploidy and integer allele-specific copy number states"
- [discussion] gHET retention filter for BAF analysis: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [discussion] Read quality filter applied before BAF/RDR computation: "Only uniquely mappable reads with a minimum mapping quality of 30 were considered"
- [other] Concordant loss calls with BAF evidence as a Fig. 3 validation signal: [evidence span withheld — anchor status: not_found]
- [readme] Allele- and haplotype-specific copy number output files: [evidence span withheld — anchor status: external_doc]
