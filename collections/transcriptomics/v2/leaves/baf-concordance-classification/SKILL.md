---
name: baf-concordance-classification
description: Use when you have single-cell CNA calls from a BAF-agnostic (read-depth-only)
  caller and phased bulk germline heterozygous SNPs, and you need to quantify how
  many calls are supported by local allelic (BAF) evidence per cell type.
when_to_use_negative:
- When the CNA caller already models BAF and phasing directly (e.g., HiScanner, CHISEL,
  Alleloscope) — post-hoc BAF validation is redundant and the calls are already allele-specific.
- When no matched bulk WGS is available to phase gHETs and filter on bulk VAF (0.1–0.9),
  since local BAF cannot be anchored to a haplotype-phased heterozygous locus.
- When the target is calling CNVs from scratch rather than auditing the reliability
  of existing single-cell CNA calls.
edam_operation: http://edamontology.org/operation_0452
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3678
tools:
- name: Ginkgo
  role: Read-depth-only single-cell CNA caller whose per-cell calls are the object
    of BAF-concordance validation; command-line version obtained from robertaboukhalil/ginkgo
  repo: https://github.com/robertaboukhalil/ginkgo
- name: HiScanner
  role: BAF-aware single-cell CNA caller whose combined read-depth + BAF + phasing
    approach motivates the concordance criterion; context for interpreting discordant
    read-depth-only calls
  repo: https://github.com/parklab/HiScanner
- name: SHAPEIT2
  role: Phases bulk gHETs using the 1000 Genomes Phase 3 reference panel so local
    BAF can be computed on phased heterozygous sites
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:7f180bf7bb152d7faaa448b5883650cffa37582df8aa019a53a6ee8f32ebbc7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robertaboukhalil/ginkgo
  tools:
  - Ginkgo
  - HiScanner
  - SCAN2
  - SHAPEIT2
  edam_operation: http://edamontology.org/operation_0452
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# baf-concordance-classification

## Summary

Validates single-cell read-depth-based CNA calls (e.g., from Ginkgo) against local B-allele frequency (BAF) evidence computed from phased gHETs in the same cell's BAM, then quantifies BAF-concordance/discordance fractions per cell type. This tests whether read-depth-only calls are supported by allelic evidence, as read-depth signals alone can be confounded by single-cell whole-genome amplification artifacts.

## When to use

Use when you have single-cell CNA calls from a BAF-agnostic (read-depth-only) caller and phased bulk germline heterozygous SNPs, and you need to quantify how many calls are supported by local allelic (BAF) evidence per cell type.

## When NOT to use

- When the CNA caller already models BAF and phasing directly (e.g., HiScanner, CHISEL, Alleloscope) — post-hoc BAF validation is redundant and the calls are already allele-specific.
- When no matched bulk WGS is available to phase gHETs and filter on bulk VAF (0.1–0.9), since local BAF cannot be anchored to a haplotype-phased heterozygous locus.
- When the target is calling CNVs from scratch rather than auditing the reliability of existing single-cell CNA calls.

## Inputs

- Single-cell BAM files (PTA-amplified scWGS, e.g., 65 brain cells from 11 neurotypical brains)
- Bulk-derived phased heterozygous germline SNP (gHET) calls per individual, phased with SHAPEIT2
- Bulk CNA call set for overlap filtering
- Ginkgo per-cell CNA calls (segmented read-depth ratio states)

## Outputs

- Per-call table ginkgo_baf_concordance.tsv with per-region local BAF and concordance/discordance labels
- Stratified discordance fractions per cell type and per call-length bin (e.g., Fig. 4d-j style summary)

## How to apply

Run the read-depth caller (Ginkgo, downloaded from the robertaboukhalil/ginkgo repository command-line version) with default settings on each single-cell BAM (65 PTA-amplified brain cells from 11 neurotypical brains) to emit per-cell CNA segments. For each called region, gather phased gHETs inside and flanking the segment — retaining only gHETs phaseable by SHAPEIT2 with bulk VAF between 0.1 and 0.9 — and compute the local BAF from the single-cell BAM, restricted to autosomes. Classify each call as BAF-concordant only if the local BAF deviates from 0.5 in the direction implied by the read-depth copy-number state; remove any single-cell calls overlapping bulk CNA calls. Aggregate discordance fractions per cell type (oligodendrocytes vs neurons) and per call-length bin, and write the per-call table and stratified summary fractions to ginkgo_baf_concordance.tsv.

## Related tools

- **Ginkgo** (Read-depth-only single-cell CNA caller whose per-cell calls are the object of BAF-concordance validation; command-line version obtained from robertaboukhalil/ginkgo) — https://github.com/robertaboukhalil/ginkgo
- **HiScanner** (BAF-aware single-cell CNA caller whose combined read-depth + BAF + phasing approach motivates the concordance criterion; context for interpreting discordant read-depth-only calls) — https://github.com/parklab/HiScanner
- **SHAPEIT2** (Phases bulk gHETs using the 1000 Genomes Phase 3 reference panel so local BAF can be computed on phased heterozygous sites)

## Examples

```
git clone https://github.com/robertaboukhalil/ginkgo && cd ginkgo && make
```

## Evaluation signals

- Aggregate discordance fractions must reproduce the reported values: 97% (581/599) of oligodendrocyte calls and 66% (1410/2121) of neuronal calls BAF-discordant.
- Total per-cell-type call counts (599 oligodendrocyte, 2121 neuronal) must match before/after the stated filters.
- Concordance directionality check: a call is concordant only when local BAF deviates from 0.5 in the direction implied by the read-depth copy-number state (e.g., BAF shifted toward 0 for amplification of one allele).
- Confirm no retained call overlaps a bulk CNA and no non-autosomal gHETs were used.
- All gHETs used in local BAF computation must satisfy the bulk VAF 0.1–0.9 filter.

## Limitations

- Ginkgo's read-depth-only calls are confounded by PTA amplification artifacts, which cause large read-depth fluctuations even at high sequencing depth — hence most oligodendrocyte calls (97%) were BAF-discordant.
- Calls overlapping bulk CNA calls must be removed, which shrinks the audited call set and can shift the discordance fraction.
- Restricted to autosomes; sex chromosomes excluded for cross-sample (both sexes) consistency, and gHETs outside the 0.1–0.9 bulk VAF range carry no usable allelic evidence.

## Evidence

- [discussion] gHET bulk VAF filter: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [discussion] bulk CNA overlap removal: "Single-cell CNA calls that overlapped with bulk CNA calls were removed"
- [discussion] autosome restriction: "To maintain consistency across samples from both sexes, our analysis was restricted to autosomal chromosomes"
- [discussion] Ginkgo provenance: "Ginkgo’s command line version was downloaded from https://github.com/robertaboukhalil/ginkgo"
- [readme] Ginkgo tool description: [evidence span withheld — anchor status: external_doc]
