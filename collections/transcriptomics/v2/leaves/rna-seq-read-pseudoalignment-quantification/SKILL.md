---
name: rna-seq-read-pseudoalignment-quantification
description: Use when you need to quantify reads supporting annotated and novel splicing
  events from short-read RNA-seq FASTQ files faster than a spliced aligner (fortuna
  was ~7x faster and mapped reads with mismatches across novel junctions more accurately).
when_to_use_negative:
- When the analysis requires per-read genomic alignments or a spliced BAM — pseudoalignment
  yields equivalence-class-based signature counts, not alignments
- When reads cannot be pseudoaligned because no pre-built fortuna index exists for
  that read length or species (pre-built indexes are human GENCODE r42 at 75/100/150
  bp; other species 'will follow')
- When the only goal is quantifying already-known, fully annotated transcripts without
  detecting novel splicing events — plain kallisto on a transcriptome index is simpler
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3174
tools:
- name: fortuna
  role: Guesses novel transcript fragments from annotated splice sites, preprocesses
    the GTF (processGTF.sh), builds the kallisto index (--index), runs pseudoalignment
    (--quant), and annotates novel events; --refine incorporates spliced-aligner splice
    sites
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Pseudoaligns reads to the fortuna fragment index and supplies equivalence
    classes from which signature counts are derived
  repo: https://github.com/pachterlab/kallisto
- name: htslib
  role: Dependency bundled with fortuna for reading the spliced-aligner BAM during
    the --refine step
  repo: https://github.com/samtools/htslib
- name: STAR
  role: Spliced aligner producing a genome BAM whose novel splice sites from unmapped
    reads can be incorporated by 'fortuna --refine'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:f62bad198adec7f383f8df11794bb1c55ded9afc9e50ce1c27307111883f68f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - STAR
  - fortuna
  - htslib
  - kallisto
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# rna-seq-read-pseudoalignment-quantification

## Summary

Alignment-free quantification of RNA-seq reads by pseudoaligning them with kallisto to fragments of annotated and 'guessed' novel transcripts, producing elementary signature counts and annotated novel splicing events (ES, IR, AA, AD, AP, IE, XX) without a traditional align-and-count step.

## When to use

Use when you need to quantify reads supporting annotated and novel splicing events from short-read RNA-seq FASTQ files faster than a spliced aligner (fortuna was ~7x faster and mapped reads with mismatches across novel junctions more accurately).

## When NOT to use

- When the analysis requires per-read genomic alignments or a spliced BAM — pseudoalignment yields equivalence-class-based signature counts, not alignments
- When reads cannot be pseudoaligned because no pre-built fortuna index exists for that read length or species (pre-built indexes are human GENCODE r42 at 75/100/150 bp; other species 'will follow')
- When the only goal is quantifying already-known, fully annotated transcripts without detecting novel splicing events — plain kallisto on a transcriptome index is simpler

## Inputs

- Genome sequence in FASTA format
- Transcript annotation in GTF format (e.g. GENCODE human release 42, GRCh38.p13)
- Short-read RNA-seq FASTQ files with a known target read length (75/100/150 bp)
- Optionally, a genome BAM of spliced alignments from a spliced aligner (e.g. STAR 1-pass)

## Outputs

- .rg.gtf segmented annotation with subexon NodeIds (from processGTF.sh)
- kallisto index of annotated and guessed novel transcript fragments
- cnt.tsv — signature counts as pipe-separated subexon IDs with counts (e.g. '223832|223833|223836|   2274')
- alt.tsv — annotated novel splicing events (ES, IR, AA, AD, AP, IE, XX) with per-event supporting read counts
- cnt.refined.tsv / alt.refined.tsv when running '--refine' with a spliced-aligner BAM

## How to apply

First preprocess the GENCODE GTF annotation with fortuna's bin/processGTF.sh, which computes minimal disjoint subexon segments bounded by splice sites and writes a segmented .rg.gtf (subexon NodeIds referenced by outputs). Then run 'fortuna --index' with the genome FASTA, the preprocessed GTF, and -rl set to the longest read length (pre-built human GENCODE release 42, GRCh38.p13 indexes exist for 75bp, 100bp, and 150bp reads) to guess novel transcript fragments (Tas) from annotated splice sites via the -lgo strategy and build the kallisto index. Run 'fortuna --quant' on each sample FASTQ to pseudoalign reads to fragments with kallisto and derive signature counts from equivalence classes, writing cnt.tsv and per-event supporting read counts in alt.tsv. Optionally run 'fortuna --refine' with a spliced-aligner BAM (e.g. STAR 1-pass) to incorporate novel splice sites from previously unmapped reads, yielding cnt.refined.tsv and alt.refined.tsv.

## Related tools

- **fortuna** (Guesses novel transcript fragments from annotated splice sites, preprocesses the GTF (processGTF.sh), builds the kallisto index (--index), runs pseudoalignment (--quant), and annotates novel events; --refine incorporates spliced-aligner splice sites) — https://github.com/canzarlab/fortuna
- **kallisto** (Pseudoaligns reads to the fortuna fragment index and supplies equivalence classes from which signature counts are derived) — https://github.com/pachterlab/kallisto
- **htslib** (Dependency bundled with fortuna for reading the spliced-aligner BAM during the --refine step) — https://github.com/samtools/htslib
- **STAR** (Spliced aligner producing a genome BAM whose novel splice sites from unmapped reads can be incorporated by 'fortuna --refine')

## Examples

```
bash bin/processGTF.sh /path/to/annotation.gtf && ./fortuna --index -rl 100 genome.fa annotation.rg.gtf && ./fortuna --quant sample.fastq
```

## Evaluation signals

- On the bundled STAT1 example dataset (~240,000 reads from sample 4, patient id 11352.p1, from Jaganathan et al. 2019), fortuna maps over 99% of reads
- The same example must show slightly over 1000 reads supporting the novel exon skipping at chr2:190984393-190986854 (grep of alt.tsv returns an ES event for gene Locus_18066 with 1256 reads)
- cnt.tsv lines must be pipe-separated subexon IDs (NodeIds from the preprocessed sample.rg.gtf) followed by a count; alt.tsv fields take chromosome, gene_id, and transcript_id from the input GTF
- Every STAR-mapped junction read on the ASD samples should be identically mapped by fortuna, and on the sample 20 skipping event STAR and fortuna agree on 196 reads Whippet left unmapped
- Rebuilt per-event supporting read counts reproduce Supplementary Table S3

## Limitations

- Pseudoalignment quantifies fragments but does not produce per-read genomic alignments; event support relies on the guessed fragment set
- The fortuna index is read-length specific; --rl must match the longest read length, and pre-built indexes cover only human GENCODE r42 at 75/100/150 bp
- Novel splice sites lying outside annotated splice-site combinations are only recovered in the optional --refine step, which requires a spliced-aligner BAM of reads unmapped in step (D)
- Compilation requires C++11 and CMake >=2.8.12; Mac OS users may need autoconf 2.69 installed via brew

## Evidence

- [abstract] It then pseudoaligns reads to fragments using kallisto and efficiently derives counts of the most elementary splicing units from kallisto's equivalence classes.: "It then pseudoaligns reads to fragments using kallisto and efficiently derives counts of the most elementary splicing units from kallisto's equivalence classes."
- [abstract] fortuna was around 7x faster than traditional align and count approaches: "fortuna was around 7× faster than traditional align and count approaches, and was able to analyze almost 300 million reads in just 15 min when using four threads."
- [readme] A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites (segments s<sub>1</sub>, s<sub>2</sub>, s<sub>3</sub>, s<sub>4</sub> in"
- [readme] fortuna will map over 99% of those reads and find slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854: "fortuna will map over 99% of those reads and find slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854."
- [readme] The above example implies that a signature comprised of subexons 223832, 223833, and 223836 has a count of 2274.: "The above example implies that a signature comprised of subexons 223832, 223833, and 223836 has a count of 2274."
- [intro] Optionally, fortuna can further incorporate novel splice sites identified by any spliced aligner from reads that remained unmapped: [evidence span withheld — anchor status: not_found]
