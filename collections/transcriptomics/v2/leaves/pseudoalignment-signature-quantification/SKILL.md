---
name: pseudoalignment-signature-quantification
description: Use when you need alignment-free quantification of transcript splicing
  units or detection/annotation of novel splicing events from RNA-seq reads, and prefer
  pseudoalignment (kallisto equivalence classes) over STAR-style align-and-count pipelines.
when_to_use_negative:
- You already have a genome alignment/BAM or need base-level alignment output rather
  than pseudoalignment-based counts
- Your annotation lacks splice-site boundaries needed for subexon segmentation (processGTF.sh
  requires a spliced GTF annotation)
- Target reads are from an organism or read length for which no fortuna index can
  be built (only H. sapiens 75/100/150 bp GENCODE v42 indices are pre-built; other
  species 'will follow')
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3320
tools:
- name: fortuna
  role: 'Main pipeline: preprocesses GTF (processGTF.sh), builds fragment index (--index),
    pseudoaligns reads and outputs signature counts and novel event annotations (--quant)'
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Bundled dependency performing the pseudoalignment of reads to transcript fragments
    and producing equivalence classes from which counts are derived
  repo: https://github.com/pachterlab/kallisto
- name: htslib
  role: Bundled dependency used by fortuna for reading/writing high-throughput sequencing
    file formats
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:de3cebbe3eb8cb9797fb46071efe4423ff72c83aeb12d96b6e8a4a296e039fa1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - STAR
  - Whippet
  - fortuna
  - htslib
  - kallisto
  - samtools
  - sigcount
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# pseudoalignment-signature-quantification

## Summary

Quantify the most elementary splicing units (signature counts) from RNA-seq FASTQs by pseudoaligning reads with kallisto against an index of annotated and novel transcript fragments built by fortuna, avoiding costly genome alignment while still detecting novel splicing events.

## When to use

Use when you need alignment-free quantification of transcript splicing units or detection/annotation of novel splicing events from RNA-seq reads, and prefer pseudoalignment (kallisto equivalence classes) over STAR-style align-and-count pipelines.

## When NOT to use

- You already have a genome alignment/BAM or need base-level alignment output rather than pseudoalignment-based counts
- Your annotation lacks splice-site boundaries needed for subexon segmentation (processGTF.sh requires a spliced GTF annotation)
- Target reads are from an organism or read length for which no fortuna index can be built (only H. sapiens 75/100/150 bp GENCODE v42 indices are pre-built; other species 'will follow')

## Inputs

- Genome sequence in FASTA format
- Transcript annotation in GTF format (e.g. RefSeq release 109 or GENCODE release 42 GRCh38.p13)
- RNA-seq reads in FASTQ format (paired-end or single-end; simulated 75/100 bp or real datasets such as the ~291M-read ASD sample 29)

## Outputs

- cnt.tsv — tab-separated signature counts, e.g. '223832|223833|223836|	2274' with subexon IDs from the segmented GTF NodeId field
- alt.tsv — novel splicing events with chromosome, coordinates, affected transcript IDs, event type (e.g. ES exon skipping), and supporting read count
- annotation.rg.gtf — segmented GTF of minimal disjoint subexon segments (from processGTF.sh)
- fortuna kallisto index per read length (fortuna --index)

## How to apply

Preprocess the transcript annotation GTF with fortuna's bin/processGTF.sh to compute minimal disjoint subexon segments bounded by splice sites, producing a segmented .rg.gtf (subexon IDs come from its NodeId field). Run fortuna --index with the genome FASTA, the segmented GTF, and the target read length (e.g. -rl 75 or -rl 100) to 'guess' novel transcripts from annotated splice sites, create fragments of all unspliced exonic segment combinations, and build a kallisto index. Then run fortuna --quant on the FASTQ (e.g. with -miss 3 and multi-threading, four threads scales to ~300M reads in ~15 min) to pseudoalign reads to fragments and derive signature counts from kallisto's equivalence classes. Optionally refine by incorporating novel splice sites identified by any spliced aligner from reads that remained unmapped. Outputs are cnt.tsv (signature counts as pipe-separated subexon ID lists) and alt.tsv (novel events typed as ES etc., with supporting read counts), which can be used directly for alternative splicing analysis or aggregated to larger units such as those used by DEXSeq.

## Related tools

- **fortuna** (Main pipeline: preprocesses GTF (processGTF.sh), builds fragment index (--index), pseudoaligns reads and outputs signature counts and novel event annotations (--quant)) — https://github.com/canzarlab/fortuna
- **kallisto** (Bundled dependency performing the pseudoalignment of reads to transcript fragments and producing equivalence classes from which counts are derived) — https://github.com/pachterlab/kallisto
- **htslib** (Bundled dependency used by fortuna for reading/writing high-throughput sequencing file formats) — https://github.com/samtools/htslib

## Examples

```
bash bin/processGTF.sh annotation.gtf && ./fortuna --index -rl 75 genome.fa annotation.rg.gtf && ./fortuna --quant -thr 4 -miss 3 reads.fq
```

## Evaluation signals

- On the STAT1 example dataset (~240,000 reads from sample 4, patient id 11352.p1), fortuna maps over 99% of reads and finds slightly over 1000 reads supporting the novel exon skipping at chr2:190984393-190986854 (line 'chr2 190984393 190986854 ... ES 1256' in alt.tsv)
- cnt.tsv lines are pipe-separated subexon ID signatures followed by a tab and integer count, with IDs matching the NodeId field of the input sample.rg.gtf
- On simulated 75/100 bp benchmark data, fortuna maps over 99% of simulated reads and reports hits-per-read around 1.02/1.04/1.16
- Scaling check: fortuna --quant with four threads (-thr 4) counts all ~291M reads of ASD sample 29 in ~15 min, over 2x faster than Whippet and ~7x faster than STAR 1-pass + sigcount (159 min)
- Peak memory should fall near reported ranges: ~4 GB (kallisto), up to ~24 GB (fortuna), versus ~29 GB for STAR

## Limitations

- Requires C++11 and CMake >= 2.8.12 to compile; depends on zlib and autoconf 2.69 (Mac OS users may need brew install autoconf@2.69)
- Peak memory is higher than Whippet (up to ~24 GB vs ~2 GB), trading memory for speed
- Pre-built indices exist only for H. sapiens GENCODE release 42 (GRCh38.p13) at 75/100/150 bp read lengths; other species must be indexed separately
- Novel splice-site incorporation from unmapped reads requires a separate spliced aligner run; fortuna alone does not discover novel splice sites from unmapped reads

## Evidence

- [intro] fortuna is an alignment-free method to quantify anambiguous building blocks of transcripts and to detect and annotate novel splicing events: "fortuna is an alignment-free method to quantify anambiguous building blocks of transcripts and to detect and annotate novel splicing events"
- [abstract] pseudoalign reads and derive signature counts: "It then pseudoaligns reads to fragments using kallisto and efficiently derives counts of the most elementary splicing units from kallisto's equivalence classes."
- [methods] index build requires FASTA + segmented GTF + read length: "Given a genome sequence in FASTA format, a preprocessed transcript annotation (GTF), and a target read length, it builds an index of"
- [readme] GTF preprocessing computes minimal disjoint segments: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites"
- [readme] example output expectation on STAT1 dataset: "fortuna will map over 99% of those reads and find slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854."
- [abstract] throughput claim: "fortuna was around 7× faster than traditional align and count approaches, and was able to analyze almost 300 million reads in just 15 min when using four threads."
