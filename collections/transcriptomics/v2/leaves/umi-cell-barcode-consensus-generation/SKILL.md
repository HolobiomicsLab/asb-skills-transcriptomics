---
name: umi-cell-barcode-consensus-generation
description: Use when single-cell CRISPR screen gRNA library BAM alignments must be
  deduplicated into per-UMI per-cell consensus gRNA sequences before variant calling,
  e.g. to reproduce the 28.4% mutant molecule fraction.
when_to_use_negative:
- Input BAM has not been through utils.gRNA_bam_filter (secondary alignments and off-cassette
  reads still present) — run the filtration step first
- The goal is assigning gRNAs to cells (assign_gRNA.assign_gRNA_to_cell with min_umi=3)
  or detecting Cas9 editing effects — these are separate GiRAFR steps/modes
- Data are bulk or UMI-less; grouping by UMI + cell barcode combination requires droplet-based
  single-cell gRNA library reads
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GiRAFR
  role: Provides consensus_sequence.generate_consensus_sequence_gRNA, the module that
    groups reads by UMI-cell barcode and emits consensus.sequence.gRNA.txt
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: pysam
  role: Reads the filtered gRNA BAM and barcode list while building per-UMI consensus
    sequences
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: Backs the upstream gRNA_bam_filter step that produces the input BAM for consensus
    generation
- name: Python
  role: Language in which GiRAFR and this consensus module are implemented
provenance:
  source_task_ids:
  - task_002
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:d9dd0e7f474e41b30d86d79893d1b3fd63bfaa82cfafd437c4c9a2fc2c82fbde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - GiRAFR
  - Python
  - pysam
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# umi-cell-barcode-consensus-generation

## Summary

GiRAFR's Step-2 module that collapses the filtered gRNA BAM (gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam) into one most-read-supported consensus sequence per UMI-cell barcode combination, discarding UMIs below min_reads so sequencing errors never enter downstream mutation calling.

## When to use

Use when single-cell CRISPR screen gRNA library BAM alignments must be deduplicated into per-UMI per-cell consensus gRNA sequences before variant calling, e.g. to reproduce the 28.4% mutant molecule fraction.

## When NOT to use

- Input BAM has not been through utils.gRNA_bam_filter (secondary alignments and off-cassette reads still present) — run the filtration step first
- The goal is assigning gRNAs to cells (assign_gRNA.assign_gRNA_to_cell with min_umi=3) or detecting Cas9 editing effects — these are separate GiRAFR steps/modes
- Data are bulk or UMI-less; grouping by UMI + cell barcode combination requires droplet-based single-cell gRNA library reads

## Inputs

- Filtered gRNA library BAM: gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam (from utils.gRNA_bam_filter, samtools-backed)
- Filtered (unzipped) CellRanger-corrected cell barcode list from the expression library
- GiRAFR ConfigFile specifying min_reads (integer, default 1; set to 2 for error filtering)

## Outputs

- consensus.sequence.gRNA.txt (one consensus sequence per UMI-cell barcode combination)
- consensus.seqeunce.gRNA.all_umi.txt
- consensus.bam
- Non-consensus.bam

## How to apply

After utils.gRNA_bam_filter has removed secondary alignments and kept only reads mapped to gRNA cassette chromosomes, run consensus_sequence.generate_consensus_sequence_gRNA on that BAM together with the filtered (unzipped) CellRanger-corrected cell barcode list from the expression library. Using pysam, keep only reads whose cell barcodes appear in the barcode list, then group reads by UMI + cell barcode combination and take the most read-supported sequence as that UMI's consensus. Set min_reads=2 (tool default is 1, configurable in ConfigFile) so UMIs supported by fewer than 2 reads are discarded as sequencing errors — this is what reduces raw mutant molecules to the reported 28.4% fraction in the in-house CROP-seq library. Rationale: per-UMI consensus calling prevents PCR duplicates and sequencing errors from being scored as distinct gRNA variants, which matters because Cell Ranger otherwise labels single-mutation gRNAs as intact.

## Related tools

- **GiRAFR** (Provides consensus_sequence.generate_consensus_sequence_gRNA, the module that groups reads by UMI-cell barcode and emits consensus.sequence.gRNA.txt) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **pysam** (Reads the filtered gRNA BAM and barcode list while building per-UMI consensus sequences) — https://github.com/pysam-developers/pysam
- **samtools** (Backs the upstream gRNA_bam_filter step that produces the input BAM for consensus generation)
- **Python** (Language in which GiRAFR and this consensus module are implemented)

## Examples

```
consensus_sequence.generate_consensus_sequence_gRNA(bam_in='gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam', barcodes=filtered_barcodes)
```

## Evaluation signals

- All four expected outputs exist: consensus.sequence.gRNA.txt, consensus.seqeunce.gRNA.all_umi.txt, consensus.bam, Non-consensus.bam
- Every UMI-cell barcode combination in consensus.sequence.gRNA.txt is represented by its most read-supported sequence, and no retained UMI has fewer than min_reads (2) supporting reads
- Only reads whose cell barcodes appear in the filtered CellRanger-corrected barcode list contribute to consensus sequences
- Feeding the output into variant.call_gRNA_variant on the in-house CROP-seq library reproduces the reported 28.4% mutant gRNA molecule fraction
- consensus.bam read content is consistent with the retained UMI-cell barcode combinations in consensus.sequence.gRNA.txt

## Limitations

- min_reads defaults to 1, which retains UMIs supported by a single read and thus sequencing errors; it must be raised (e.g. to 2) to reproduce the paper's error-filtered statistics
- Specific defined read depths need to be attained for accurate gRNA assignment and mutation detection; low gRNA UMI counts impair downstream accuracy
- Consensus calling alone does not call mutations — variant.call_gRNA_variant against oligo_pool_plasmid.fa and oligo_pool_plasmid_structure.gtf is a required downstream step

## Evidence

- [methods] Step2: Construct consensus sequence for each UMI-Cell barcode combination: "Step2: Construct consensus sequence for each UMI-Cell barcode combination"
- [methods] consensus_sequence.generate_consensus_sequence_gRNA(bam_in, barcodes): "consensus_sequence.generate_consensus_sequence_gRNA(bam_in, barcodes)"
- [methods] min_reads default and UMI filtering rule: "Integer number. By default = 1. UMIs with less than min_reads will be filtered out."
- [methods] generate filtered bam file in output folder: "generate filtered bam file in output folder named as ``gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam``"
- [readme] GiRAFR README abstract on mutant gRNAs and error attribution: "We demonstrate that the current default tool fails to detect mutant gRNAs. We therefore developed GiRAFR, a pysam-based software tool to characterize intact and mutant gRNAs."
