---
name: cellranger-grna-assignment-comparison
description: Use when a CROP-seq / single-cell CRISPR screen has both Cell Ranger
  feature-barcode gRNA calls and GiRAFR output, and you need to quantify how many
  cells carry mutant gRNAs or lack any gRNA call under the default pipeline.
when_to_use_negative:
- The goal is downstream differential-expression or perturbation-effect analysis rather
  than auditing gRNA annotation quality — use the assigned perturbation labels directly
  instead
- Input is already a GiRAFR per-cell assignment table with no Cell Ranger annotation
  available, so no head-to-head comparison can be made
- The dataset uses gRNA capture protocols other than CROP-seq-style feature barcoding
  where Cell Ranger feature reference annotation is not the baseline being audited
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: GiRAFR
  role: Mutation-aware gRNA assignment to cells; produces intact and mutant gRNA calls
    per cell barcode for comparison with Cell Ranger
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: CellRanger
  role: Provides the baseline feature-barcoding gRNA annotation (corrected cell barcodes
    and per-cell gRNA calls) that GiRAFR assignments are benchmarked against
- name: pysam
  role: Python library underlying GiRAFR for reading and manipulating the SAM/BAM
    gRNA alignment files
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: Used within utils.gRNA_bam_filter to split/filter the gRNA BAM, removing secondary
    alignments and off-target reads
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:c196d8a1c37da6fbcc89d42b3d079d3e23cdb3eebecc31be6b93a84c07801055
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cellranger-grna-assignment-comparison

## Summary

Benchmark mutation-aware GiRAFR per-cell gRNA assignments against the standard Cell Ranger feature barcoding annotation on a CROP-seq dataset, quantifying cells with mutant gRNAs and cells left without a gRNA call by Cell Ranger. This validates that the default pipeline misses mutant gRNAs and inflates untransformed-cell estimates.

## When to use

Use when a CROP-seq / single-cell CRISPR screen has both Cell Ranger feature-barcode gRNA calls and GiRAFR output, and you need to quantify how many cells carry mutant gRNAs or lack any gRNA call under the default pipeline.

## When NOT to use

- The goal is downstream differential-expression or perturbation-effect analysis rather than auditing gRNA annotation quality — use the assigned perturbation labels directly instead
- Input is already a GiRAFR per-cell assignment table with no Cell Ranger annotation available, so no head-to-head comparison can be made
- The dataset uses gRNA capture protocols other than CROP-seq-style feature barcoding where Cell Ranger feature reference annotation is not the baseline being audited

## Inputs

- gRNA library BAM aligned to a custom gRNA-cassette reference (oligo_pool_plasmid.fa)
- Cell Ranger corrected cell barcodes (unzipped filtered_barcode file)
- Cell Ranger feature-barcoding per-cell gRNA annotation table
- oligo_pool_plasmid_structure.gtf structure annotation
- ConfigFile for girafr gRNA_mutation mode

## Outputs

- gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam (filtered BAM)
- consensus.sequence.gRNA.variant.txt with mutations in CIGAR-like string format
- Per-cell GiRAFR gRNA assignment table (intact vs mutant, UMI counts)
- Comparison counts vs Cell Ranger, e.g. 2317/5744 cells without a Cell Ranger call and 1198/5744 (20.7%) cells with a mutant gRNA

## How to apply

Run GiRAFR gRNA_mutation on the in-house A549 CROP-seq data using a ConfigFile specifying gRNA_bam_file, filtered_barcode (unzipped Cell Ranger corrected cell barcodes), ref_fasta (oligo_pool_plasmid.fa) and structure_gtf (oligo_pool_plasmid_structure.gtf). Filter the gRNA library BAM with utils.gRNA_bam_filter via samtools (drop secondary alignments and reads not mapped to the designed gRNA cassette), build per-UMI consensus sequences with consensus_sequence.generate_consensus_sequence_gRNA (min_reads = 1), and call variants with variant.call_gRNA_variant against the plasmid reference. Assign gRNAs to cells with assign_gRNA.assign_gRNA_to_cell using auto = True (two-component Gaussian mixture per-gRNA UMI threshold), pool = True, and min_umi = 3 as the fallback threshold, then add mutation details with assign_gRNA.add_variant_type and profile_MT_pattern.py. Finally, cross-tabulate GiRAFR assignments (intact vs mutant) against the Cell Ranger annotation per cell barcode, counting cells with no Cell Ranger call and cells whose only call is a mutant gRNA.

## Related tools

- **GiRAFR** (Mutation-aware gRNA assignment to cells; produces intact and mutant gRNA calls per cell barcode for comparison with Cell Ranger) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **CellRanger** (Provides the baseline feature-barcoding gRNA annotation (corrected cell barcodes and per-cell gRNA calls) that GiRAFR assignments are benchmarked against)
- **pysam** (Python library underlying GiRAFR for reading and manipulating the SAM/BAM gRNA alignment files) — https://github.com/pysam-developers/pysam
- **samtools** (Used within utils.gRNA_bam_filter to split/filter the gRNA BAM, removing secondary alignments and off-target reads)

## Examples

```
girafr gRNA_mutation -f ConfigFile
```

## Evaluation signals

- Filtered BAM is produced and named gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam
- Variant file consensus.sequence.gRNA.variant.txt exists with mutations encoded in the CIGAR-like string format annotated against oligo_pool_plasmid_structure.gtf
- Reproduced counts: 2317 of 5744 cells have no gRNA call by Cell Ranger and 1198 of 5744 cells (20.7%) are assigned a mutant gRNA by GiRAFR, including 481 cells previously assigned no gRNA by Cell Ranger
- Gaussian mixture auto-threshold fitting converged per gRNA, with min_umi = 3 applied as the fixed fallback for cells below mixture reliability
- Every cell in the GiRAFR assignment table carries a Cell Ranger corrected barcode, enabling a per-cell cross-tabulation without barcode mismatch

## Limitations

- Up to 22% of cells labeled as singlets by Cell Ranger are in fact multiplets, and gRNAs with a single mutation (1 Hamming distance) are labeled as intact by Cell Ranger yet show attenuated phenotype — so the baseline annotation is itself inaccurate
- Head-to-head comparison between published studies is unfeasible because studies often use ad hoc customized pipelines and fixed thresholds for analysis
- The comparison does not provide a fine-grained appraisal of which gRNA mutation types and locations are tolerable or damaging, which was declared beyond the scope of the study
- Accurate gRNA assignment and mutation detection require specific defined read depths to be attained; inter-experiment differences in average gRNA UMIs may reflect low gRNA expression or low detection rates

## Evidence

- [discussion] Cell Ranger labels mutant gRNAs as intact: "gRNAs with a single mutation (1 Hamming distance) are labeled as intact by CellRanger but similarly show an attenuated phenotype"
- [methods] Assign gRNAs to cells: [evidence span withheld — anchor status: not_found]
- [methods] gRNA bam filtration output file: "generate filtered bam file in output folder named as ``gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam``"
- [readme] GiRAFR purpose: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
- [abstract] Default pipeline misses mutant gRNAs: "During analysis, we observed that the most commonly used method fails to detect mutant gRNAs."
- [discussion] Multiplet underestimation by Cell Ranger: "up to 22% of all cells profiled in earlier studies that are labeled as singlets by Cell Ranger are in fact multiplets"
