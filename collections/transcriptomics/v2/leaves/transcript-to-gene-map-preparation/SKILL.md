---
name: transcript-to-gene-map-preparation
description: Reference for preparing a spliced+intronic (splici) reference and its
  3-column transcript-to-gene mapping from a reference genome FASTA and GTF before
  single-cell quantification with alevin-fry/simpleaf.
when_to_use_negative:
- A piscem index and t2g mapping already exist for the reference (proceed directly
  to simpleaf quant instead of re-indexing)
- The task is bulk RNA-seq quantification against an unmodified transcriptome, where
  an expanded splici reference and gene-level t2g are not the target representation
- Inputs are raw sequencing reads rather than a reference genome FASTA plus GTF —
  this skill operates on reference/annotation files, not FASTQ
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: simpleaf
  role: Exposes the index sub-command that orchestrates expanded-reference (splici)
    construction and t2g generation, then calls piscem build
  repo: https://github.com/COMBINE-lab/simpleaf
- name: roers
  role: Rust library invoked by simpleaf to build the expanded splici reference FASTA
    and the transcript-to-gene mapping from genome FASTA + GTF
  repo: https://github.com/COMBINE-lab/roers
- name: piscem
  role: Indexes the constructed expanded reference via piscem build; -k 31 and minimizer
    -m 19 defaults apply, with k odd and <= 31 and m < k
  repo: https://github.com/COMBINE-lab/piscem
- name: alevin-fry
  role: Downstream consumer of the t2g mapping and piscem index during permit-list
    generation, collate, and quant
  repo: https://github.com/COMBINE-lab/alevin-fry
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:8e832c67210f6dcb09b9691ae826c8672401f2b4eacf1e13361812dda05e1e72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/simpleaf
  tools:
  - alevin-fry
  - piscem
  - roers
  - simpleaf
  - wget
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# transcript-to-gene-map-preparation

## Summary

Builds the expanded (splici/spliceu) reference and its transcript-to-gene (t2g) mapping from a genome FASTA plus GTF/GFF3 annotation, as implemented by simpleaf (via roers) in the alevin-fry single-cell pipeline. The t2g file is the artifact downstream mapping and quantification depend on, so its correct construction is a prerequisite for simpleaf quant.

## When to use

Reference for preparing a spliced+intronic (splici) reference and its 3-column transcript-to-gene mapping from a reference genome FASTA and GTF before single-cell quantification with alevin-fry/simpleaf.

## When NOT to use

- A piscem index and t2g mapping already exist for the reference (proceed directly to simpleaf quant instead of re-indexing)
- The task is bulk RNA-seq quantification against an unmodified transcriptome, where an expanded splici reference and gene-level t2g are not the target representation
- Inputs are raw sequencing reads rather than a reference genome FASTA plus GTF — this skill operates on reference/annotation files, not FASTQ

## Inputs

- Reference genome FASTA file
- Matching GTF (or GFF3) gene annotation file

## Outputs

- Splici (spliced+intronic) reference FASTA in the output `ref` subdirectory
- 3-column transcript-to-gene (t2g) mapping file
- Piscem index in the output `index` subdirectory, together with a copy of the t2g file

## How to apply

Download a reference genome FASTA and matching GTF annotation (e.g., from Ensembl or GENCODE), then set ALEVIN_FRY_HOME and run `simpleaf set-paths` so simpleaf can locate the piscem and alevin-fry executables. Run `simpleaf index` with --fasta and --gtf; simpleaf calls roers (used as a library) to build the spliced+intronic (splici) expanded reference with default k-mer length 31 and minimizer length 19, emitting a FASTA of spliced transcripts augmented with merged intronic sequence (read-length-aware flanks) plus the transcript-to-gene mapping. After the expanded reference is constructed, simpleaf invokes `piscem build` to index it, and the index directory receives a copy of the 3-column t2g file. If --use-piscem is set, subsequent simpleaf quant runs automatically use piscem as the mapper; the t2g is what ties mapped transcripts back to genes during USA-mode quantification.

## Related tools

- **simpleaf** (Exposes the index sub-command that orchestrates expanded-reference (splici) construction and t2g generation, then calls piscem build) — https://github.com/COMBINE-lab/simpleaf
- **roers** (Rust library invoked by simpleaf to build the expanded splici reference FASTA and the transcript-to-gene mapping from genome FASTA + GTF) — https://github.com/COMBINE-lab/roers
- **piscem** (Indexes the constructed expanded reference via piscem build; -k 31 and minimizer -m 19 defaults apply, with k odd and <= 31 and m < k) — https://github.com/COMBINE-lab/piscem
- **alevin-fry** (Downstream consumer of the t2g mapping and piscem index during permit-list generation, collate, and quant) — https://github.com/COMBINE-lab/alevin-fry

## Examples

```
simpleaf index --fasta genome.fa --gtf annotation.gtf -o ref_index_out --use-piscem
```

## Evaluation signals

- `simpleaf index` completes without error
- Output directory contains a `ref` subdirectory holding the splici reference FASTA
- Output directory contains an `index` subdirectory holding the piscem index and a copy of the 3-column transcript-to-gene file
- The t2g file has exactly 3 columns mapping transcripts to genes
- Subsequent simpleaf quant executions automatically use piscem as the mapper when the index was built with --use-piscem

## Limitations

- piscem requires the k-mer size (-k) to be odd and <= 31 and the minimizer size (-m) to be < k; the user file handle limit should be set to 2048 (ulimit -n 2048) before running piscem
- On Apple silicon, threading (-t) for piscem build should not exceed the number of high-performance cores, as efficiency cores severely degrade indexing performance
- roers can optionally deduplicate identical sequences (--dedup) when building the expanded reference; if index construction fails, the piscem README advises checking the file handle limit before filing a GitHub issue

## Evidence

- [intro] Research objective: encapsulate the process of creating an expanded reference for quantification into a single command (index): "`simpleaf` encapsulates the process of creating an expanded reference for quantification into a single command (`index`) and the quantification of a sample into a single command (`quant`)"
- [methods] roers builds splici reference from FASTA+GTF: "it will take a reference genome FASTA and GTF as input, from which it can build a spliced+intronic (splici) reference or a spliced+unspliced (spliceu) reference"
- [methods] piscem build indexes the expanded reference: "after the expanded reference is constructed, the resulting reference will be indexed with ``piscem build``"
- [readme] roers output description from README: [evidence span withheld — anchor status: external_doc]
- [readme] intronic augmentation defines splici: [evidence span withheld — anchor status: external_doc]
