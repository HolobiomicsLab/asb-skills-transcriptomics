---
name: rna-seq-splice-event-quantification
description: Use when you need to detect and count novel alternative splicing events
  (ES, IR, AA, AD, AP, IE) from RNA-seq FASTQs using only a GTF annotation and genome
  FASTA, without a spliced aligner, or when tissue-specific event catalogs (e.g. RPM
  >= 1) are needed.
when_to_use_negative:
- When a full spliced alignment is required (e.g. novel splice-site discovery beyond
  fortuna's guessed fragments) — fortuna only guesses novel combinations of annotated
  splice sites, though novel splice sites from any spliced aligner can optionally
  be incorporated from reads unmapped in step (D).
- When the annotation is not in GTF format or the target read length differs from
  the index read length — the index must be rebuilt with the correct -rl.
- When the analysis only needs per-subexon counts and no novel-event annotation —
  plain DEXSeq or simpler aggregation pipelines may suffice.
edam_operation: http://edamontology.org/operation_0262
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0085
tools:
- name: fortuna
  role: Preprocesses the GTF into subexon segments, builds the kallisto fragment index,
    quantifies signature counts, annotates novel events (alt.tsv), converts counts
    to DEXSeq units, and merges mate-specific outputs
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Performs the pseudoalignment of reads to the fortuna-built fragment index;
    equivalence classes are the basis for signature counts
  repo: https://github.com/pachterlab/kallisto
- name: DEXSeq
  role: Defines the larger per-subexon counting unit that fortuna '--trans' aggregates
    signature counts into
- name: htslib
  role: Bundled dependency of fortuna for HTS file handling during indexing/quantification
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:e016ca3e0a77f6c54854ba6dd0fcfb07c958084fd651e34ee0d7f6f8c2f45de2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - DEXSeq
  - fortuna
  - htslib
  - kallisto
  edam_operation: http://edamontology.org/operation_0262
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# rna-seq-splice-event-quantification

## Summary

Quantify reads supporting annotated and novel splicing events by alignment-free pseudoalignment: fortuna guesses novel transcript fragments from annotated splice sites, builds a kallisto index, and derives signature counts and novel-event annotations from equivalence classes without aligning raw reads.

## When to use

Use when you need to detect and count novel alternative splicing events (ES, IR, AA, AD, AP, IE) from RNA-seq FASTQs using only a GTF annotation and genome FASTA, without a spliced aligner, or when tissue-specific event catalogs (e.g. RPM >= 1) are needed.

## When NOT to use

- When a full spliced alignment is required (e.g. novel splice-site discovery beyond fortuna's guessed fragments) — fortuna only guesses novel combinations of annotated splice sites, though novel splice sites from any spliced aligner can optionally be incorporated from reads unmapped in step (D).
- When the annotation is not in GTF format or the target read length differs from the index read length — the index must be rebuilt with the correct -rl.
- When the analysis only needs per-subexon counts and no novel-event annotation — plain DEXSeq or simpler aggregation pipelines may suffice.

## Inputs

- GTF transcript annotation (preprocessed with bin/processGTF.sh into segmented .rg.gtf)
- Genome sequence in FASTA format
- RNA-seq FASTQ files (left and right mates for paired-end; -rl matched to read length, e.g. 75/100/150bp)

## Outputs

- cnt.tsv: signature counts (subexon-ID signature '|' joined to read count)
- alt.tsv: novel splicing events with chromosome, start/end coordinates, overlapping transcript IDs, gene/locus, event type (ES, IR, AA, AD, AP, IE, XX), and read support
- DEXSeq-style per-subexon counts via fortuna '--trans' aggregation

## How to apply

First preprocess the GTF with bin/processGTF.sh to compute minimal disjoint subexon segments bounded by splice sites (annotation.rg.gtf). Run fortuna '--index' with the genome FASTA, segmented GTF, and -rl set to the longest read length, controlling novel-fragment generation via -lgo and capping fragments per gene with -Mc 25000. Then run fortuna '--quant' on each mate FASTQ (with --p for paired-end intermediates, --cnt for signature counts, --alt for novel-event annotations), merge mates with '--pairs' (-lf/-rf/-out, stripping mate suffixes via -lp/-rp such as /1 and /2). Outputs are cnt.tsv (tab-separated mapping signatures — '|' -joined subexon NodeIds — and read counts) and alt.tsv (novel events with coordinates, gene/locus, event type, and supporting-read count). Signature counts can be aggregated to DEXSeq per-subexon counts via '--trans', since each subexon count equals the sum of signatures containing it.

## Related tools

- **fortuna** (Preprocesses the GTF into subexon segments, builds the kallisto fragment index, quantifies signature counts, annotates novel events (alt.tsv), converts counts to DEXSeq units, and merges mate-specific outputs) — https://github.com/canzarlab/fortuna
- **kallisto** (Performs the pseudoalignment of reads to the fortuna-built fragment index; equivalence classes are the basis for signature counts) — https://github.com/pachterlab/kallisto
- **DEXSeq** (Defines the larger per-subexon counting unit that fortuna '--trans' aggregates signature counts into)
- **htslib** (Bundled dependency of fortuna for HTS file handling during indexing/quantification) — https://github.com/samtools/htslib

## Examples

```
bash processGTF.sh annotation.gtf && ./fortuna --index -g genome.fa -gft annotation.rg.gtf -rl 100 -out idx && ./fortuna --quant -idx idx -r1 L.fq --p --cnt --alt
```

## Evaluation signals

- On the STAT1 example dataset (~240,000 reads from sample 4, patient 11352.p1), fortuna should map over 99% of reads and report slightly over 1000 reads supporting the novel exon skipping at chr2:190984393-190986854 (grep in alt.tsv returns 'Locus_18066 ES 1256').
- cnt.tsv lines must be tab-separated signatures of subexon NodeIds from the processed .rg.gtf joined by '|' with an integer count (e.g. '223832|223833|223836\t2274').
- Hand-check the '--trans' output on the Fig. 2 subexon configuration: every DEXSeq subexon count equals the sum of all signature counts whose '|' -separated signature contains that subexon ID.
- Merged '--pairs' output should agree with per-mate '--quant' runs; mate-name suffixes (e.g. /1, /2) must be stripped via -lp/-rp.
- For Drosophila tissue catalogs, reconstructed distributions should match reported values: ~7.5% shared vs 13%/15%/48% IFM/leg/brain-specific events at RPM >= 1, ~2% of events with >= 150 reads, and > 60% with <= 5 reads.

## Limitations

- Novel-event detection is limited to guessed recombinations of annotated splice sites; novel splice sites must come from an external spliced aligner on reads unmapped in step (D).
- Fortuna requires C++11 and CMake >= 2.8.12; the bundled kallisto/htslib are built from source (zlib and autoconf 2.69 needed), and pre-compiled binaries are only for x86_64 Linux, Mac OS x86, and Mac OS Arm64 (M1/M2).
- The Docker image was built on Linux and tested on Linux and Mac OS, but not on Windows; Mac OS grep lacks -P support for alt.tsv filtering.

## Evidence

- [abstract] fortuna, a method that guesses novel combinations of annotated splice sites to create transcript fragments. It then pseudoaligns reads to fragments using kallisto and efficiently derives counts of the most elementary splicing units from kallisto's equivalence classes.: "fortuna, a method that guesses novel combinations of annotated splice sites to create transcript fragments."
- [intro] Computed counts can be directly used for alternative splicing analysis or added up to larger units such as those used by DEXSeq [3] (E1). In addition, fortuna annotates all novel events (E2): "Computed counts can be directly used for alternative splicing analysis or added up to larger units such as those used by DEXSeq [3] (E1)."
- [methods] A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites"
- [readme] File cnt.tsv contains signature counts in the format documented below. An example line from the file follows. 223832|223833|223836|   2274: "File cnt.tsv contains signature counts in the format documented below."
- [readme] fortuna will map over 99% of those reads and find slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854.: "fortuna will map over 99% of those reads and find slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854."
