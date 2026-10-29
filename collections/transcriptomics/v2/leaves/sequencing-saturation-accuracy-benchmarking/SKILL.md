---
name: sequencing-saturation-accuracy-benchmarking
description: Use when you need to quantify how deeply the gRNA library was sequenced
  in a CROP-seq/single-cell CRISPR screen and determine the read depth (e.g. 34%,
  59%, 92% saturation) required for reliable gRNA-to-cell assignment.
when_to_use_negative:
- You want to detect CRISPR-Cas9 editing effects in the transcriptome data — that
  requires GiRAFR's separate editing-effect analysis mode, not the gRNA_mutation downsampling
  benchmark.
- You want to identify mutant gRNAs in the full library at a single operating point
  without assessing depth-dependence — the standard GiRAFR gRNA_mutation pipeline
  without downsampling suffices.
- The question is which mutation types or locations are tolerable vs damaging — the
  article explicitly declares this fine-grained appraisal beyond its scope.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: GiRAFR
  role: 'gRNA_mutation pipeline: bam filtration, consensus sequence construction,
    variant calling, and gRNA-to-cell assignment at each saturation level'
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: samtools
  role: random subsampling of the gRNA alignment BAM to fixed saturation fractions
    and splitting of the alignment file
- name: pysam
  role: Python library underlying GiRAFR's BAM reading/manipulation
  repo: https://github.com/pysam-developers/pysam
- name: Python
  role: implementation language for downsampling driver, metric computation, and compilation
    of saturation_annotation_metrics.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:16415f14ab6191407dc0bb82f201f87549647a438e306e92293c177ae66ababa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - GiRAFR
  - Python
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sequencing-saturation-accuracy-benchmarking

## Summary

Benchmarks how gRNA library sequencing saturation affects single-cell CRISPR screen annotation quality by downsampling the gRNA alignment BAM to fixed fractions, re-running the GiRAFR gRNA_mutation pipeline per level, and plotting annotation accuracy against unique gRNA UMIs recovered per cell.

## When to use

Use when you need to quantify how deeply the gRNA library was sequenced in a CROP-seq/single-cell CRISPR screen and determine the read depth (e.g. 34%, 59%, 92% saturation) required for reliable gRNA-to-cell assignment.

## When NOT to use

- You want to detect CRISPR-Cas9 editing effects in the transcriptome data — that requires GiRAFR's separate editing-effect analysis mode, not the gRNA_mutation downsampling benchmark.
- You want to identify mutant gRNAs in the full library at a single operating point without assessing depth-dependence — the standard GiRAFR gRNA_mutation pipeline without downsampling suffices.
- The question is which mutation types or locations are tolerable vs damaging — the article explicitly declares this fine-grained appraisal beyond its scope.

## Inputs

- gRNA library alignment BAM file from a single-cell CRISPR screen (CROP-seq)
- oligo_pool_plasmid.fa reference of designed gRNA sequences
- oligo_pool_plasmid_structure.gtf annotating mutation positions in the gRNA cassette
- filtered cell barcode list
- GiRAFR ConfigFile (target genome, gRNA library parameters)

## Outputs

- one downsampled BAM per saturation level (34%, 59%, 92%)
- per-level GiRAFR outputs: gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam, consensus BAM, cells.gRNA.txt, gRNA.umi.threshold.txt
- saturation_annotation_metrics.csv with annotation accuracy and median/average UMIs per cell at each saturation level
- saturation-accuracy curve (annotation accuracy vs unique gRNA molecules per cell)

## How to apply

Downsample the gRNA library alignment BAM to defined saturation levels (e.g. 34%, 59%, 92% of original reads) using random subsampling with samtools view, producing one downsampled BAM per level. For each level, run the GiRAFR gRNA_mutation pipeline: filter the BAM to yield gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam, build UMI-cell barcode consensus sequences with consensus_sequence.generate_consensus_sequence_gRNA (min_reads=1), call mutations against oligo_pool_plasmid.fa annotated by oligo_pool_plasmid_structure.gtf, and assign gRNAs to cells with assign_gRNA.assign_gRNA_to_cell in auto UMI-threshold mode (per-gRNA two-component Gaussian mixture fit over pooled intact and mutated UMIs). From cells.gRNA.txt and gRNA.umi.threshold.txt, compute annotation accuracy and unique gRNA UMIs per cell at each saturation, then compile into saturation_annotation_metrics.csv. The rationale is that saturation directly limits UMI recovery per cell, and per-cell UMI count drives assignment accuracy: in the in-house CROP-seq experiment, 34%/59%/92% saturation yielded ~5/~16/~31 unique gRNA molecules per cell and ~80%/~95%/~98% correct annotation.

## Related tools

- **GiRAFR** (gRNA_mutation pipeline: bam filtration, consensus sequence construction, variant calling, and gRNA-to-cell assignment at each saturation level) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **samtools** (random subsampling of the gRNA alignment BAM to fixed saturation fractions and splitting of the alignment file)
- **pysam** (Python library underlying GiRAFR's BAM reading/manipulation) — https://github.com/pysam-developers/pysam
- **Python** (implementation language for downsampling driver, metric computation, and compilation of saturation_annotation_metrics.csv)

## Examples

```
samtools view -s 0.34 -b gRNA.aligned.bam > gRNA.downsampled_34pct.bam; python -c "import consensus_sequence; consensus_sequence.generate_consensus_sequence_gRNA(bam_in, barcodes)"
```

## Evaluation signals

- Reproduced annotation accuracies at 34%, 59%, and 92% saturation approximate ~80%, ~95%, and ~98% (Figure 3b of the paper).
- Median unique gRNA UMIs per cell at 34%, 59%, and 92% saturation approximate ~5, ~16, and ~31.
- Each saturation level yields a filtered BAM named gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam.
- accuracy and UMI-per-cell metrics increase monotonically with saturation across the three levels.
- All three levels are compiled into a single saturation_annotation_metrics.csv with matching schema.

## Limitations

- Requires gRNA library read depths sufficient for the chosen saturation levels; the discussion notes specific defined read depths need to be attained for accurate gRNA assignment and mutation detection.
- Results are experiment-specific: multiplicity of infection, number of designed gRNAs, introduced mutations, and cells per perturbation all influence assignment accuracy and may require re-benchmarking.
- Inter-experiment differences in average gRNA UMIs may stem from low gRNA expression or low detection rates, complicating interpretation of the accuracy-vs-saturation curve.
- Subsampling is from one library; ambient RNA and other batch effects are not modelled by the downsampling procedure itself.

## Evidence

- [other] Downsampling the in-house CROP-seq experiment showed that at sequencing saturations of 34%, 59%, and 92%, an average of 5, 16, and 31 unique gRNA molecules were recovered per cell, yielding correct gRNA-to-cell annotation at approximately 80%, 95%, and 98%: [evidence span withheld — anchor status: not_found]
- [methods] step 1: gRNA bam file filtration: "step 1: gRNA bam file filtration."
- [methods] Step2: Construct consensus sequence for each UMI-Cell barcode combination: "Step2: Construct consensus sequence for each UMI-Cell barcode combination"
- [methods] auto UMI-threshold mode: "assign_gRNA.assign_gRNA_to_cell()"
- [discussion] specific defined read depths need to be attained for accurate gRNA assignment and mutation detection: "For example, specific defined read depths need to be attained for accurate gRNA assignment and mutation detection."
