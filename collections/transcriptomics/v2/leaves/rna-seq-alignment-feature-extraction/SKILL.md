---
name: rna-seq-alignment-feature-extraction
description: Use when you have coordinate-sorted RNA-seq BAM alignments plus RefSeq
  GTF annotation and need to classify candidate terminal exons (skipped vs composite
  IPA) using read-based features rather than sequence motifs.
when_to_use_negative:
- The goal is only APA quantification of annotated 3'UTR sites — tools like MISO,
  QAPA, LABRAT or DaPars already handle this without novel IPA discovery
- 'The input BAM has too few supporting reads: candidates with fewer than 5 uniquely
  mapped spliced reads (skipped) or fewer than 10 spanning reads (composite) are not
  considered'
- Only unaligned FASTQ data is available and genome alignment is out of scope for
  the current task — this skill operates on sorted BAM alignments, not raw reads
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: InPACT
  role: 'Main tool: constructs putative intronic terminal exons from the BAM and trains
    random-forest classifiers on alignment features to call skipped vs composite IPA
    sites'
  repo: https://github.com/YY-TMU/InPACT
- name: featureCount
  role: Computes per-region RNA-seq alignment features; candidate regions with enough
    reads are retained as determined by featureCount
- name: SAMtools
  role: 'Preprocessing: filters out reads flagged as secondary alignment (and removes
    PCR duplicates with samtools rmdup)'
- name: Salmon
  role: 'Downstream quantification: indexes and quantifies assembled novel transcripts
    so IPA usage (PAU estimate) can be calculated'
  repo: https://github.com/COMBINE-lab/salmon
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:7e395a1638583cc34a7bb0f58d171adc1af24728be8b32171c9f0d93c6bb54fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - HISAT2
  - InPACT
  - SAMtools
  - Salmon
  - featureCount
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# rna-seq-alignment-feature-extraction

## Summary

Extracts per-region RNA-seq alignment features (with featureCount) from a sorted coordinate BAM and trains ensemble random-forest classifiers to classify putative skipped and composite intronic terminal exons, enabling InPACT to identify previously unannotated intronic polyadenylation (IPA) sites from conventional RNA-seq.

## When to use

Use when you have coordinate-sorted RNA-seq BAM alignments plus RefSeq GTF annotation and need to classify candidate terminal exons (skipped vs composite IPA) using read-based features rather than sequence motifs.

## When NOT to use

- The goal is only APA quantification of annotated 3'UTR sites — tools like MISO, QAPA, LABRAT or DaPars already handle this without novel IPA discovery
- The input BAM has too few supporting reads: candidates with fewer than 5 uniquely mapped spliced reads (skipped) or fewer than 10 spanning reads (composite) are not considered
- Only unaligned FASTQ data is available and genome alignment is out of scope for the current task — this skill operates on sorted BAM alignments, not raw reads

## Inputs

- Coordinate-sorted RNA-seq BAM file (e.g. HISAT2-aligned reads against GRCh38, secondary alignments removed with SAMtools)
- RefSeq GTF annotation (e.g. hg38.109.20211119.ncbiRefSeq.gtf.gz from UCSC goldenPath)
- Candidate IPA site SAF file (regions from 20 nt upstream of potential polyA sites: gene, chromosome, start, end, strand)
- Reference genome FASTA (for downstream transcript assembly)

## Outputs

- Per-region alignment feature table (relative region length, normalized expression, coefficients of variation, entropy efficiency, spliced/unspliced counts at 5'/3' ends)
- predict.result.txt — classified terminal exons per IPA site with IPA type (Skipped or composite), gene, upstream coordinate, and polyA site
- AUROC and other metrics (TPR, TNR, accuracy, precision, F1, MCC) per replicate and class
- merged.gtf with assembled novel IPA isoforms (via InPACT_transcript)

## How to apply

Define three training classes from the RefSeq annotation: terminal exons (annotated last exons with at least five splice-in reads), internal exons (annotated exons between first and last with at least five splice-in reads), and background regions (annotated last exons with fewer than five splice-in reads), then randomly split regions 80:20 into training and testing sets. Compute alignment features per region with featureCount — relative region length, normalized region expression, coefficients of variation, entropy efficiency, and spliced/unspliced read counts around the 5' and 3' ends — after removing secondary alignments with SAMtools (and rRNA reads with Bowtie, PCR duplicates with samtools rmdup). Train separate random-forest ensembles over ten randomized subsamples of the training set: one for skipped terminal exons (requiring >=5 uniquely mapped spliced reads with the 3' end in the candidate region) and one for composite terminal exons (requiring >=10 uniquely mapped unspliced reads crossing the closest upstream splice site). Evaluate each ensemble on the 20% held-out set, computing TPR, TNR, accuracy, precision, F1, MCC and AUROC.

## Related tools

- **InPACT** (Main tool: constructs putative intronic terminal exons from the BAM and trains random-forest classifiers on alignment features to call skipped vs composite IPA sites) — https://github.com/YY-TMU/InPACT
- **featureCount** (Computes per-region RNA-seq alignment features; candidate regions with enough reads are retained as determined by featureCount)
- **SAMtools** (Preprocessing: filters out reads flagged as secondary alignment (and removes PCR duplicates with samtools rmdup))
- **Salmon** (Downstream quantification: indexes and quantifies assembled novel transcripts so IPA usage (PAU estimate) can be calculated) — https://github.com/COMBINE-lab/salmon

## Examples

```
InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5
```

## Evaluation signals

- AUROC per replicate and class must match expected values within a small tolerance: rep1 skipped 0.999 / composite 0.997; rep2 skipped 0.998 / composite 0.998 (auroc_results.csv)
- Held-out metrics (TPR, TNR, accuracy, precision, F1, MCC) computed on the 20% testing split should be consistently high across the ten-subsample ensembles
- Skipped candidates must satisfy the >=5 uniquely mapped spliced reads threshold and composite candidates the >=10 spanning reads threshold before classification
- Training class definitions must hold: terminal exons with >=5 splice-in reads, internal exons with >=5 splice-in reads, background last exons with <5 splice-in reads
- Output predict.result.txt must contain the documented columns (Terminal exon, IPA type, Gene, Upstream coordinate, PolyAsite, IPA usage)

## Limitations

- Default read-count thresholds (5 spliced reads for skipped, 10 spanning reads for composite) mean lowly expressed IPA events may be missed entirely
- Classifiers are ensembles over ten randomized subsamples, so results depend on the random split; no version number is given for InPACT, complicating reproducibility
- Read module depends on adequate sequencing depth for coverage-based features; the paper notes tools relying on read coverage fluctuations necessitate adequate sequencing depth

## Evidence

- [methods] random forest ensembling: "InPACT trains classifiers on ten randomized subsamples of the training set using random forest, and then ensemble the classifiers."
- [methods] terminal exon training class: "annotated unique last exons that have at least five splice-in reads"
- [readme] skipped read threshold: "Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]"
- [readme] composite read threshold: "Minimum number of reads that cross the boundary required to consider the potential composite terminal exon. [default=10]"
- [methods] preprocessing filters: "Reads flagged as secondary alignment were filtered out using SAMtools"
