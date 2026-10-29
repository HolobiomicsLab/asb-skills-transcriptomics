---
name: splici-reference-generation
description: Use when starting from a raw genome FASTA plus matching GTF annotation
  and needing an expanded (splici) reference and mapper index for alevin-fry USA-mode
  single-cell quantification.
when_to_use_negative:
- The expanded reference and piscem index already exist in the output directory —
  re-running would rebuild the index needlessly
- Quantification is to be run on reads already mapped (RAD files) — this skill covers
  only reference construction and indexing, not `simpleaf quant`
- Only a transcriptome FASTA (no genome FASTA + GTF) is available — roers requires
  a genome FASTA and a GTF/GFF3 annotation as input
edam_operation: http://edamontology.org/operation_0327
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: simpleaf
  role: Wraps the whole expanded-reference construction and indexing phase behind
    the `simpleaf index` sub-command; passes --use-piscem state to downstream quant
    calls
  repo: https://github.com/COMBINE-lab/simpleaf
- name: roers
  role: Rust library invoked internally by simpleaf to build the spliced+intronic
    (splici) expanded reference FASTA and t2g mapping from genome FASTA and GTF
  repo: https://github.com/COMBINE-lab/roers
- name: piscem
  role: Builds the index over the expanded reference (`piscem build`) and serves as
    the RNA back-end mapper for subsequent mapping
  repo: https://github.com/COMBINE-lab/piscem
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:292a6cee454b6777fb942eec1e9b6a750cfcd15bf6e44eb9a2e31cbb9e29ef4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/simpleaf
  tools:
  - piscem
  - roers
  - simpleaf
  - wget
  edam_operation: http://edamontology.org/operation_0327
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# splici-reference-generation

## Summary

Builds a spliced+intronic (splici) expanded transcriptome reference from a reference genome FASTA and GTF for alevin-fry/simpleaf single-cell quantification, then indexes it with piscem. Encapsulated by simpleaf as a single `index` command, with roers performing the splici construction.

## When to use

Use when starting from a raw genome FASTA plus matching GTF annotation and needing an expanded (splici) reference and mapper index for alevin-fry USA-mode single-cell quantification.

## When NOT to use

- The expanded reference and piscem index already exist in the output directory — re-running would rebuild the index needlessly
- Quantification is to be run on reads already mapped (RAD files) — this skill covers only reference construction and indexing, not `simpleaf quant`
- Only a transcriptome FASTA (no genome FASTA + GTF) is available — roers requires a genome FASTA and a GTF/GFF3 annotation as input

## Inputs

- Reference genome FASTA (e.g., from Ensembl or GENCODE)
- Matching GTF/GFF3 annotation file

## Outputs

- ref/ subdirectory containing the splici reference FASTA and a 3-column transcript-to-gene (t2g) file
- index/ subdirectory containing the piscem index plus a copy of the 3-column transcript-to-gene file

## How to apply

Run `simpleaf index` with --fasta and --gtf pointing at the downloaded reference genome FASTA and GTF (e.g., Ensembl or GENCODE); simpleaf internally calls roers to build the spliced+intronic (splici) expanded reference using default k-mer length 31 and minimizer length 19, then invokes `piscem build` to index the resulting reference. Before running, set ALEVIN_FRY_HOME and run `simpleaf set-paths` so simpleaf can locate the piscem and alevin-fry executables. If --use-piscem is passed to `simpleaf index`, subsequent `simpleaf quant` calls against this index automatically use piscem as the mapper with appropriate parameters. The splici reference is the augmentation type used for USA-mode quantification; a spliced+unspliced (spliceu) reference is the alternative, and roers also supports gene-body and transcript-body augmentations via -a.

## Related tools

- **simpleaf** (Wraps the whole expanded-reference construction and indexing phase behind the `simpleaf index` sub-command; passes --use-piscem state to downstream quant calls) — https://github.com/COMBINE-lab/simpleaf
- **roers** (Rust library invoked internally by simpleaf to build the spliced+intronic (splici) expanded reference FASTA and t2g mapping from genome FASTA and GTF) — https://github.com/COMBINE-lab/roers
- **piscem** (Builds the index over the expanded reference (`piscem build`) and serves as the RNA back-end mapper for subsequent mapping) — https://github.com/COMBINE-lab/piscem

## Examples

```
export ALEVIN_FRY_HOME=~/.alevin_fry; simpleaf set-paths; simpleaf index --fasta genome.fa --gtf genes.gtf --use-piscem -o splici_index
```

## Evaluation signals

- The output directory contains a `ref` subdirectory holding the splici reference FASTA and a 3-column transcript-to-gene (t2g) file
- The output directory contains an `index` subdirectory holding the piscem index together with the copy of the 3-column t2g file
- `simpleaf index` completes without error (the run succeeds only if this and the expected output files all exist)
- With defaults, the splici reference was built using k-mer length 31 and minimizer length 19, as consumed by piscem build
- Later `simpleaf quant` runs against this index automatically select piscem as the mapper when --use-piscem was set at index time

## Limitations

- Building a piscem index can fail if the user file handle limit is too low; piscem README recommends `ulimit -n 2048` before running
- K-mer size for the underlying colored compacted de Bruijn graph must be odd and <= 31, and the minimizer size must be < k
- On Apple silicon, using efficiency cores in `piscem build` has been observed to severely degrade performance; set -t to the number of high-performance cores

## Evidence

- [intro] simpleaf encapsulates expanded-reference construction into a single command: "`simpleaf` encapsulates the process of creating an expanded reference for quantification into a single command (`index`)"
- [methods] simpleaf index takes genome FASTA and GTF and builds splici or spliceu references: "it will take a reference genome FASTA and GTF as input, from which it can build a spliced+intronic (splici) reference or a spliced+unspliced (spliceu) reference"
- [methods] after expanded reference construction the reference is indexed with piscem build: "after the expanded reference is constructed, the resulting reference will be indexed with ``piscem build``"
- [readme] roers README: spliced + intronic is the splici reference used for USA-mode quantification: [evidence span withheld — anchor status: external_doc]
- [methods] set-paths stores executable paths in ALEVIN_FRY_HOME configuration: "The ``set-paths`` command is used to set the paths to the relevant executables and store them in a configuration file in the ``ALEVIN_FRY_HOME`` directory"
