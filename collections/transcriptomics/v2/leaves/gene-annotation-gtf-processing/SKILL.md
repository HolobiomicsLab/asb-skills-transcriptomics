---
name: gene-annotation-gtf-processing
description: 'Use when preparing a reference for splicing-status prediction of scRNA-seq
  reads: you have a gene annotation GTF and matching genome FASTA and need an augmented
  spliced+unspliced transcriptome (spliceu.gtf, spliceu.fa, t2g_3col.tsv).'
when_to_use_negative:
- You only need a standard spliced transcriptome index (e.g. Salmon/kallisto quantification),
  not a spliced+unspliced augmented reference for splicing-status prediction.
- Your GTF and genome FASTA come from different genome builds — the unspliced genomic-span
  transcripts require consistent coordinates.
- You already have a completed spliceu index directory containing spliceu.gtf, spliceu.fa,
  and t2g_3col.tsv.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: R (build_spliceu.R)
  role: Runs the preprocessing script that converts the GTF and genome FASTA into
    the spliceu spliced+unspliced augmented reference
  repo: https://github.com/COMBINE-lab/forseti
- name: conda
  role: Creates and activates the isolated build_spliceu_r environment from forseti/envs/build_spliceu_r.yml
    for index building
  repo: https://github.com/COMBINE-lab/forseti
- name: Forseti
  role: Downstream consumer of the spliceu reference for predicting splicing status
    of scRNA-seq reads
  repo: https://github.com/COMBINE-lab/forseti
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:44588ff98346d750da5d6da7e0342a3c13953d19e8836eafbd387a1cfd970563
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - R
  - R (build_spliceu.R)
  - conda
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gene-annotation-gtf-processing

## Summary

Constructs the Forseti spliceu augmented gene annotation reference, in which every transcript is represented twice — as a spliced exon concatenation and as an unspliced genomic span — from a standard GTF plus genome FASTA, so downstream alignments gain read compatibility to both splicing states.

## When to use

Use when preparing a reference for splicing-status prediction of scRNA-seq reads: you have a gene annotation GTF and matching genome FASTA and need an augmented spliced+unspliced transcriptome (spliceu.gtf, spliceu.fa, t2g_3col.tsv).

## When NOT to use

- You only need a standard spliced transcriptome index (e.g. Salmon/kallisto quantification), not a spliced+unspliced augmented reference for splicing-status prediction.
- Your GTF and genome FASTA come from different genome builds — the unspliced genomic-span transcripts require consistent coordinates.
- You already have a completed spliceu index directory containing spliceu.gtf, spliceu.fa, and t2g_3col.tsv.

## Inputs

- Genome FASTA (e.g. refdata-gex-GRCh38-2020-A fasta/genome.fa)
- Gene annotation GTF (e.g. refdata-gex-GRCh38-2020-A genes/genes.gtf)
- Thread count for the build (e.g. 12)

## Outputs

- spliceu.gtf — augmented gene annotation GTF with spliced and unspliced transcript entries
- spliceu.fa — spliced+unspliced transcriptome FASTA
- t2g_3col.tsv — transcript-to-gene mapping TSV
- Co-located copies of genome.fa and genes.gtf in the index directory

## How to apply

Obtain a standard 10x-style reference such as refdata-gex-GRCh38-2020-A, which supplies genome.fa under fasta/ and genes.gtf under genes/. Create the dedicated conda environment build_spliceu_r from forseti/envs/build_spliceu_r.yml to isolate R dependencies and avoid R/Python conflicts. Run the R preprocessing script build_spliceu.R with the genome FASTA, the GTF, an output index directory, and a thread count (e.g. 12); the script emits the three spliceu artifacts: spliceu.gtf (augmented gene annotation), spliceu.fa (transcriptome FASTA), and t2g_3col.tsv (transcript-to-gene mapping). Move genome.fa and genes.gtf into the index directory so all reference files co-locate for the subsequent STAR genomeGenerate step, then conda deactivate before creating the separate forseti_test environment. Rationale: ambiguous ('A') reads must align to both spliced and unspliced versions of each transcript for the prediction model to be able to classify their splicing status.

## Related tools

- **R (build_spliceu.R)** (Runs the preprocessing script that converts the GTF and genome FASTA into the spliceu spliced+unspliced augmented reference) — https://github.com/COMBINE-lab/forseti
- **conda** (Creates and activates the isolated build_spliceu_r environment from forseti/envs/build_spliceu_r.yml for index building) — https://github.com/COMBINE-lab/forseti
- **Forseti** (Downstream consumer of the spliceu reference for predicting splicing status of scRNA-seq reads) — https://github.com/COMBINE-lab/forseti

## Examples

```
Rscript forseti/preprocess/build_spliceu.R $REF_DIR/fasta/genome.fa $REF_DIR/genes/genes.gtf $IDX_DIR 12; mv $REF_DIR/fasta/genome.fa $IDX_DIR/genome.fa
```

## Evaluation signals

- The index directory contains exactly the three expected spliceu artifacts: spliceu.gtf, spliceu.fa, and t2g_3col.tsv.
- build_spliceu.R completes without error and the conda environment build_spliceu_r was used.
- genome.fa and genes.gtf have been moved into the index directory alongside the generated outputs.
- Each transcript in spliceu.fa has both a spliced representation (concatenated exons) and an unspliced representation (full genomic span of all its exons).
- The t2g_3col.tsv mapping covers both spliced and unspliced transcript IDs and maps them to the correct gene IDs.

## Limitations

- Separate conda environments are needed because combining R and Python in one environment causes conflicts — the build_spliceu_r env must be deactivated before creating/activating the forseti_test env.
- The spliceu reference only augments the annotation; it does not itself resolve fundamentally ambiguous reads (e.g. polyA-tail priming with read2 in the terminal exon), which Forseti as a mechanistic model cannot distinguish.
- Tutorial references and parameters (e.g. 10x refdata-gex-GRCh38-2020-A, 12 threads) assume a 10x-style human reference layout with fasta/genome.fa and genes/genes.gtf paths.

## Evidence

- [methods] The spliceu reference contains both a spliced and an unspliced representation for each transcript annotation: spliced transcripts are defined as the concatenation of the corresponding exons, and each unspliced transcript is defined as the underlying genomic span of all exons of the transcript.: [evidence span withheld — anchor status: not_found]
- [readme] The R script build_spliceu.R will build a spliceu (spliced + unspliced), an augmented transcriptome reference consisting of a augmented gene annotation GTF file, a transcriptome FASTA file, and a transcript-to-gene mapping TSV file.: "The R script `build_spliceu.R` will build a _spliceu_ (spliced + unspliced), an augmented transcriptome reference consisting of a augmented gene"
- [methods] Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts.: "Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts."
- [readme] To handle the potential conflicts for environments having both R and python, we have two seperate envs: build_spliceu_r for building spliceu index.: "To handle the potential conflicts for environments having both R and python, we have two seperate envs:
* _build_spliceu_r_ for building spliceu index."
