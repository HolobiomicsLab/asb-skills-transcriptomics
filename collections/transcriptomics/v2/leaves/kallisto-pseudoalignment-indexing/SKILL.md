---
name: kallisto-pseudoalignment-indexing
description: Use when quantifying reads against annotated plus novel transcript fragments
  without an aligner, deriving signature counts from kallisto equivalence classes,
  or detecting/quantifying novel splicing events alignment-free.
when_to_use_negative:
- The task requires aligned reads or per-base alignment quality (e.g. variant calling,
  base-level coverage) — pseudoalignment only determines read-to-fragment compatibility.
- Novel splice sites rather than novel combinations of annotated splice sites are
  the target and no spliced aligner output is available to supply them.
- Read length differs from the index's -rl setting — rebuild the index for the correct
  read length.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3170
tools:
- name: kallisto
  role: Pseudoaligns reads to the fragment set and supplies equivalence classes from
    which fortuna derives signature counts
  repo: https://github.com/pachterlab/kallisto
- name: fortuna
  role: Builds the fragment set and kallisto index (--index) and pseudoaligns reads
    (--quant), then annotates novel splicing events (--alt)
  repo: https://github.com/canzarlab/fortuna
- name: htslib
  role: Bundled dependency of fortuna for reading/writing high-throughput sequencing
    file formats
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:75bdacb22de1d000ce433ee6ba693a05cca2249c9c326dddc330fc3a05201db5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pachterlab/kallisto
  tools:
  - fortuna
  - htslib
  - kallisto
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# kallisto-pseudoalignment-indexing

## Summary

Builds a kallisto index from a set of annotated and guessed-novel transcript fragments and pseudoaligns RNA-seq reads to those fragments, yielding counts of elementary splicing units (signature counts) without full alignment — the core quantification engine fortuna relies on.

## When to use

Use when quantifying reads against annotated plus novel transcript fragments without an aligner, deriving signature counts from kallisto equivalence classes, or detecting/quantifying novel splicing events alignment-free.

## When NOT to use

- The task requires aligned reads or per-base alignment quality (e.g. variant calling, base-level coverage) — pseudoalignment only determines read-to-fragment compatibility.
- Novel splice sites rather than novel combinations of annotated splice sites are the target and no spliced aligner output is available to supply them.
- Read length differs from the index's -rl setting — rebuild the index for the correct read length.

## Inputs

- Genome sequence in FASTA format
- Preprocessed/segmented transcript annotation (annotation.rg.gtf, produced by bin/processGTF.sh from a GTF)
- Target read length (via -rl, matching the sequencing library's longest read length)
- RNA-seq reads in FASTQ format (left and right mate files for paired-end data)

## Outputs

- kallisto index of annotated and guessed novel transcript fragments
- cnt.tsv: signature counts as pipe-separated subexon NodeIds with per-signature counts
- alt.tsv: novel splicing events with coordinates, gene/transcript IDs, event type (ES, IR, AA, AD, AP, IE, XX), and read support
- Merged paired-end output via fortuna --pairs

## How to apply

First preprocess the GTF annotation (fortuna's bin/processGTF.sh) into minimal disjoint subexon segments bounded by splice sites, then run fortuna '--index' with the genome FASTA and segmented GTF, setting -rl to the longest expected read length and controlling novel-fragment generation via -lgo and the -Mc cap of 25000 fragments per gene. Run fortuna '--quant' on each mate's FASTQ file to pseudoalign reads to the fragment index; kallisto's pseudoalignment determines read-to-fragment compatibility without alignment, and fortuna derives signature counts from kallisto's equivalence classes (cnt.tsv lists signatures as pipe-separated subexon NodeIds with counts). For paired-end data, merge left/right per-tissue outputs with fortuna '--pairs' (-lf, -rf, -out), stripping mate-name suffixes (e.g. /1, /2) via -lp/-rp. Optionally incorporate novel splice sites from reads that remained unmapped in the pseudoalignment step, identified by any spliced aligner.

## Related tools

- **kallisto** (Pseudoaligns reads to the fragment set and supplies equivalence classes from which fortuna derives signature counts) — https://github.com/pachterlab/kallisto
- **fortuna** (Builds the fragment set and kallisto index (--index) and pseudoaligns reads (--quant), then annotates novel splicing events (--alt)) — https://github.com/canzarlab/fortuna
- **htslib** (Bundled dependency of fortuna for reading/writing high-throughput sequencing file formats) — https://github.com/samtools/htslib

## Examples

```
bash processGTF.sh /path/to/annotation.gtf && ./fortuna --index -g genome.fa -a annotation.rg.gtf -rl 100 -out idx
```

## Evaluation signals

- On the STAT1 example dataset (~240,000 reads from patient 11352.p1), fortuna should map over 99% of reads and report slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854 in alt.tsv (event type ES, gene Locus_18066, 1256 reads).
- cnt.tsv signature IDs must correspond to NodeId values present in the preprocessed GTF (e.g. '223832|223833|223836|  2274'); verify the counts file schema before downstream aggregation.
- alt.tsv entries should carry chromosome, start/end coordinates, affected transcript IDs, gene_id, event type, and read support count — check the grep result against the documented format.
- The number of novel fragments stays within the -Mc cap of 25000 fragments per gene, and -rl matches the library's longest read length.

## Limitations

- fortuna 'guesses' novel combinations of annotated splice sites, so novel events outside the space of annotated splice sites require optional incorporation of novel splice sites from unmapped reads via a spliced aligner.
- Novel-fragment generation must be bounded (-lgo, -Mc cap of 25000 fragments per gene) to keep the index tractable.
- The method was benchmarked as ~7x faster than traditional align-and-count approaches (almost 300 million reads in 15 min with four threads), but counts derive from pseudoalignment equivalence classes, not full alignments.
- Mac OS lacks grep -P support for querying alt.tsv; the tab escape may need adjustment depending on grep implementation.

## Evidence

- [intro] Build a kallisto index from the fragment set (C): "From this set of fragments we build a kallisto [2] index (C) and use it to efficiently pseudoalign reads to fragments (D)"
- [abstract] fortuna pseudoaligns reads and derives signature counts from equivalence classes: "It then pseudoaligns reads to fragments using kallisto and efficiently derives counts of the most elementary splicing units from kallisto's equivalence classes."
- [methods] Indexing command and parameters: "Steps B and C are executed using fortuna's "--index" command. Given a genome sequence in FASTA format, a preprocessed transcript annotation (GTF), and a target read length, it builds an index of"
- [readme] Read length index availability for GENCODE: "you can download here the fortuna index for different read lengths Other species will follow."
- [readme] Example dataset expected output: "fortuna will map over 99% of those reads and find slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854."
- [other] Paired-end merging with --pairs: [evidence span withheld — anchor status: not_found]
