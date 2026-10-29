---
name: fastq-preprocessing-and-downsampling
description: Use when starting from published scRNA-seq resources (e.g. Cell Ranger
  1.1.0 BAMs from Chromium, or oversized FASTQs from ICELL8) that must be converted,
  reduced to chromosome 21, or downsampled to a fixed read count before pipeline benchmarking.
when_to_use_negative:
- Input FASTQs are already size-appropriate for full analysis and no downsampling
  or platform-specific reference restriction (e.g. Chr21) is intended
- Data are 5' scRNA-seq technologies other than Chromium, which UniverSC does not
  support at the time of writing
- The downstream goal is full-genome analysis rather than the reduced chromosome 21
  benchmark, since Chr21 filtering exists only to reduce output data size
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: bamtofastq
  role: Converts Cell Ranger 1.1.0 BAM output back to FASTQ for re-running pipelines;
    requires --cr11 for older 10x pipelines
  repo: https://github.com/10XGenomics/bamtofastq
- name: seqtk
  role: Downsamples the ICELL8 dataset to 250 K reads with the same random seed per
    read
- name: Docker
  role: Containerised environment providing consistent execution of the pipeline stack
  repo: https://github.com/minoda-lab/universc
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34681-z
    title: A flexible cross-platform single-cell data processing pipeline
  source_content_hash: sha256:8922219c24d02937ee858c90f99b2fda5736aa8dacf1f5b49e6d54c27a3be069
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/bamtofastq
  tools:
  - Docker
  - UniverSC
  - bamtofastq
  - dropSeqPipe
  - seqtk
  - zUMIs
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# fastq-preprocessing-and-downsampling

## Summary

Prepares published single-cell RNA-seq datasets for cross-platform processing by converting 10x Cell Ranger BAM output back to FASTQ with bamtofastq, filtering reads to chromosome 21, and downsampling large FASTQs (e.g. ICELL8) with seqtk, enabling fair UniverSC-vs-native-pipeline benchmarks on the GRCh38 (hg38) Chr21 reference.

## When to use

Use when starting from published scRNA-seq resources (e.g. Cell Ranger 1.1.0 BAMs from Chromium, or oversized FASTQs from ICELL8) that must be converted, reduced to chromosome 21, or downsampled to a fixed read count before pipeline benchmarking.

## When NOT to use

- Input FASTQs are already size-appropriate for full analysis and no downsampling or platform-specific reference restriction (e.g. Chr21) is intended
- Data are 5' scRNA-seq technologies other than Chromium, which UniverSC does not support at the time of writing
- The downstream goal is full-genome analysis rather than the reduced chromosome 21 benchmark, since Chr21 filtering exists only to reduce output data size

## Inputs

- Cell Ranger 1.1.0 BAM output from a published Chromium human cell line dataset
- FASTQ files from ICELL8 and other scRNA-seq platforms (Drop-seq, SmartSeq3)
- GRCh38 (hg38) chromosome 21 reference

## Outputs

- FASTQ files reconstructed from the 10x BAM (mkfastq-compatible directory layout)
- Chromosome 21-filtered FASTQ reads
- Downsampled ICELL8 FASTQ subset (250 K reads, fixed random seed)

## How to apply

For the Chromium dataset, convert the Cell Ranger 1.1.0 BAM output back to FASTQ using the 10x Genomics bamtofastq tool, supplying the --cr11 flag since Cell Ranger v1.0-1.1 BAMs predate automatic sequence-layout headers. Reduce output size by keeping only reads that mapped to chromosome 21, matching the GRCh38 (hg38) Chr21 reference used for all datasets. For ICELL8, downsample to 250 K reads with seqtk sample, using the same random seed for each read so the subsampling is reproducible. Run these steps before handing the FASTQs to UniverSC or the native pipelines (Cell Ranger, dropSeqPipe, CogentAP, zUMIs) so both arms of any comparison consume identical reads.

## Related tools

- **bamtofastq** (Converts Cell Ranger 1.1.0 BAM output back to FASTQ for re-running pipelines; requires --cr11 for older 10x pipelines) — https://github.com/10XGenomics/bamtofastq
- **seqtk** (Downsamples the ICELL8 dataset to 250 K reads with the same random seed per read)
- **Docker** (Containerised environment providing consistent execution of the pipeline stack) — https://github.com/minoda-lab/universc

## Examples

```
bamtofastq --cr11 chromium_cellranger110.bam chr21_fastq/
```

## Evaluation signals

- The bamtofastq output directory mirrors the mkfastq-style layout and the FASTQs contain the same sequence set as originally input to Cell Ranger (order not preserved)
- Only reads mapped to chromosome 21 remain in the filtered FASTQs
- Downsampled ICELL8 FASTQ contains 250,000 reads, reproducible when re-run with the same seqtk seed
- Downstream Pearson correlation validation still passes: Chromium pair r = 1 vs Cell Ranger 3.0.2 and Drop-seq, ICELL8, SmartSeq3 pairs r >= 0.94 (Fig. 2 left-column expectation)

## Limitations

- bamtofastq does not preserve read order, only the sequence content
- Older 10x BAMs (Cell Ranger <1.2, e.g. 1.1.0) require explicit --cr11 flags; only BAMs with sequence-layout headers convert automatically
- Multi-Gem Group BAMs from Cell Ranger 1.2 and earlier lack Read Group tags, so GEM groups cannot be distinguished
- 5' scRNA-seq technologies other than Chromium are not supported by UniverSC at the time of writing

## Evidence

- [methods] the 10x Genomics bamtofastq tool (https://github.com/10XGenomics/bamtofastq) was used to convert Cell Ranger 1.1.0 output: "the 10x Genomics bamtofastq tool (https://github.com/10XGenomics/bamtofastq) was used to convert Cell Ranger 1.1.0 output"
- [methods] Only the reads that mapped to chromosome 21 were kept to reduce output data size.: "Only the reads that mapped to chromosome 21 were kept to reduce output data size."
- [methods] The ICELL8 dataset was further down sampled to 250 K reads using seqtk17 (sample with the same random seed for each read).: "The ICELL8 dataset was further down sampled to 250 K reads using seqtk17 (sample with the same random seed for each read)."
- [readme] The FASTQ files emitted by the tool should contain the same set of sequences that were input to the original pipeline run, although the order will not be preserved.: [evidence span withheld — anchor status: external_doc]
- [readme] Older 10x pipelines require arguments to indicate which pipeline created the BAM.: [evidence span withheld — anchor status: external_doc]
