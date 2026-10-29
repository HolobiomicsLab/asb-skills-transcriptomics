---
name: grna-cell-assignment-umi-thresholding
description: Use when you have a filtered gRNA BAM (UMI + cell barcode aware) from
  a droplet CRISPR screen (e.g. CROP-seq) and need per-cell gRNA calls that include
  mutant gRNAs missed by Cell Ranger feature barcoding.
when_to_use_negative:
- The goal is detecting CRISPR-Cas9-induced DNA editing in transcriptome data — that
  is GiRAFR's separate editing_effect analysis mode, not gRNA-to-cell assignment.
- You only need gRNA library quality control/mutation profiling without per-cell annotation.
- The input is already a per-cell gRNA annotation table (e.g. Cell Ranger feature
  barcoding output) — the point here is to replace/augment that assignment.
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GiRAFR
  role: python package providing assign_gRNA.assign_gRNA_to_cell (auto Gaussian-mixture
    UMI thresholding, pool=True, min_umi=3 fallback), assign_gRNA.add_variant_type,
    and the gRNA_mutation pipeline
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: pysam
  role: underlying BAM/UMI parsing library GiRAFR is built on for consensus generation
    and variant calling
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: used by utils.gRNA_bam_filter to remove secondary and non-gRNA-cassette alignments
    before assignment
- name: CellRanger
  role: source of corrected cell barcodes (filtered_barcode) and the baseline annotation
    to compare against
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:e9cc5e0559368862beb6abacc3c960d3494396317e0a5c5b9f2e4923d0404589
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - CellRanger
  - GiRAFR
  - Python
  - pysam
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# grna-cell-assignment-umi-thresholding

## Summary

Mutation-aware assignment of intact and mutant gRNAs to individual cells from a single-cell CRISPR screen, using per-gRNA UMI thresholds fitted with a two-component Gaussian mixture instead of a fixed cutoff, as implemented in GiRAFR's assign_gRNA.assign_gRNA_to_cell.

## When to use

Use when you have a filtered gRNA BAM (UMI + cell barcode aware) from a droplet CRISPR screen (e.g. CROP-seq) and need per-cell gRNA calls that include mutant gRNAs missed by Cell Ranger feature barcoding.

## When NOT to use

- The goal is detecting CRISPR-Cas9-induced DNA editing in transcriptome data — that is GiRAFR's separate editing_effect analysis mode, not gRNA-to-cell assignment.
- You only need gRNA library quality control/mutation profiling without per-cell annotation.
- The input is already a per-cell gRNA annotation table (e.g. Cell Ranger feature barcoding output) — the point here is to replace/augment that assignment.

## Inputs

- Filtered gRNA library BAM (gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam) with UMIs and cell barcodes
- Unzipped Cell Ranger corrected cell barcode list (filtered_barcode)
- Consensus sequence file per UMI-cell barcode (from consensus_sequence.generate_consensus_sequence_gRNA)
- gRNA variant file (consensus.sequence.gRNA.variant.txt) with CIGAR-like mutation strings
- gRNA reference FASTA (oligo_pool_plasmid.fa) and structure GTF (oligo_pool_plasmid_structure.gtf)

## Outputs

- cells.gRNA.txt: per-cell gRNA assignments (intact and mutant)
- gRNA.umi.threshold.txt: per-gRNA fitted UMI thresholds
- gRNA annotation accuracy and median UMIs per cell (for QC metrics)

## How to apply

Run the upstream GiRAFR gRNA_mutation pipeline first: filter the gRNA library BAM with utils.gRNA_bam_filter (remove secondary alignments and reads not mapped to the designed gRNA cassette, yielding gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam), build a consensus sequence per UMI-cell barcode combination with consensus_sequence.generate_consensus_sequence_gRNA (min_reads = 1), and call variants against oligo_pool_plasmid.fa with variant.call_gRNA_variant using the structure GTF. Then invoke assign_gRNA.assign_gRNA_to_cell with auto = True so a two-component Gaussian mixture is fitted per gRNA on UMI counts, pooling intact and mutated UMIs per gRNA (pool = True), and fall back to the fixed threshold min_umi = 3 (cells with UMIs >= min_umi are assigned a gRNA). The rationale is that Cell Ranger feature barcoding misses mutant gRNAs, inflating the untransformed-cell count; pooling intact and mutated UMIs preserves threshold robustness for gRNAs with many mutants. Optionally annotate assigned cells with assign_gRNA.add_variant_type and profile_MT_pattern.py.

## Related tools

- **GiRAFR** (python package providing assign_gRNA.assign_gRNA_to_cell (auto Gaussian-mixture UMI thresholding, pool=True, min_umi=3 fallback), assign_gRNA.add_variant_type, and the gRNA_mutation pipeline) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **pysam** (underlying BAM/UMI parsing library GiRAFR is built on for consensus generation and variant calling) — https://github.com/pysam-developers/pysam
- **samtools** (used by utils.gRNA_bam_filter to remove secondary and non-gRNA-cassette alignments before assignment)
- **CellRanger** (source of corrected cell barcodes (filtered_barcode) and the baseline annotation to compare against)

## Examples

```
girafr gRNA_mutation -f ConfigFile  # ConfigFile sets filtered_barcode, ref_fasta=oligo_pool_plasmid.fa, structure_gtf=oligo_pool_plasmid_structure.gtf; then assign_gRNA.assign_gRNA_to_cell(auto=True, pool=True, min_umi=3)
```

## Evaluation signals

- gRNA.umi.threshold.txt contains a fitted (Gaussian-mixture) threshold for each gRNA rather than a single global cutoff.
- Cells with UMI counts >= min_umi = 3 are assigned a gRNA; cells below threshold are left unassigned.
- On the A549 CROP-seq benchmark, GiRAFR assigns a mutant gRNA to 1198/5744 cells (20.7%), including 481 cells with no Cell Ranger gRNA call; Cell Ranger leaves 2317/5744 cells without a call.
- Downsampling check: at 34%, 59%, and 92% gRNA-library saturation, annotation accuracy should approach ~80%, ~95%, and ~98% with ~5, ~16, and ~31 unique gRNA molecules per cell (Figure 3b reproduction).
- Mutant gRNA assignments (via assign_gRNA.add_variant_type) appear in cells.gRNA.txt, not just intact-gRNA calls.

## Limitations

- Fixed thresholds (min_umi = 3 fallback) may misassign cells when gRNA UMI distributions vary between experiments; ad hoc customized pipelines and fixed thresholds make head-to-head comparison between studies unfeasible.
- Accurate gRNA assignment requires specific defined read depths — accuracy depends on gRNA library sequencing saturation (80% accuracy at only 34% saturation).
- Inter-experiment differences in average gRNA UMIs (low expression vs low detection) are undetermined and can harm singlet/multiplet discrimination and ambient-vs-endogenous assignment.
- Which gRNA mutation types and locations are tolerable or damaging was not appraised (declared beyond the scope of the study).

## Evidence

- [methods] Cells with UMIs >= min_umi will be assigned with gRNA.: "Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA."
- [methods] Step 4 assigns gRNAs to cells in the GiRAFR pipeline.: "Step4: Assign gRNAs to cells"
- [readme] GiRAFR assigns gRNAs to cells in a sensitive, mutation-aware manner.: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
- [abstract] Failure to detect mutant gRNAs leads to inflated untransformed-cell estimates.: "failure to detect and annotate them leads to an inflated estimate of the number of untransformed cells as well as an underestimated multiplet frequency"
