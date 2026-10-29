---
name: piscem-index-building
description: Use when you need to create a searchable reference index for alevin-fry/scRNA-seq
  quantification from a reference genome FASTA plus GTF annotation, e.g. preparing
  a splici reference for later `simpleaf quant` runs.
when_to_use_negative:
- You already have a pre-built reference index or a ready-made transcriptome-only
  reference and do not need expanded (splici/spliceu) construction from genome FASTA
  + GTF
- You are quantifying against a probe-based target set and need probe-set CSV to FASTA
  conversion plus probe_t2g.tsv generation instead of a sequence-based splici index
- The task is bulk RNA-seq abundance estimation with piscem-infer, which uses map-bulk
  against its own index rather than the single-cell map-scrna workflow
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: simpleaf
  role: Exposes the indexing phase as the `simpleaf index` sub-command; orchestrates
    roers for expanded-reference construction and invokes piscem build
  repo: https://github.com/COMBINE-lab/simpleaf
- name: piscem
  role: RNA back-end mapper and indexer; `piscem build` indexes the expanded reference
    produced by roers
  repo: https://github.com/COMBINE-lab/piscem
- name: roers
  role: Used as a library directly from simpleaf to construct the spliced+intronic
    (splici) expanded reference from the genome FASTA and GTF
  repo: https://github.com/COMBINE-lab/roers
- name: alevin-fry
  role: Downstream quantification ecosystem that consumes the index via piscem map-scrna;
    simpleaf requires it and registers it via set-paths
  repo: https://github.com/COMBINE-lab/alevin-fry
- name: wget
  role: Required dependency of simpleaf (used to fetch reference resources such as
    FASTA/GTF files)
  repo: https://www.gnu.org/software/wget/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:f014b32ef3ede89ad3b87a8751c6b36580be6cacc46f005e6b2f39ebd786509a
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# piscem-index-building

## Summary

Build a piscem index over an expanded (splici) reference for alevin-fry single-cell quantification, as orchestrated by the `simpleaf index` command: roers constructs the spliced+intronic reference from a genome FASTA and GTF, then `piscem build` indexes it.

## When to use

Use when you need to create a searchable reference index for alevin-fry/scRNA-seq quantification from a reference genome FASTA plus GTF annotation, e.g. preparing a splici reference for later `simpleaf quant` runs.

## When NOT to use

- You already have a pre-built reference index or a ready-made transcriptome-only reference and do not need expanded (splici/spliceu) construction from genome FASTA + GTF
- You are quantifying against a probe-based target set and need probe-set CSV to FASTA conversion plus probe_t2g.tsv generation instead of a sequence-based splici index
- The task is bulk RNA-seq abundance estimation with piscem-infer, which uses map-bulk against its own index rather than the single-cell map-scrna workflow

## Inputs

- Reference genome FASTA (e.g. from Ensembl or GENCODE)
- Matching GTF (or GFF3) annotation file
- Optional probe-set CSV for probe-based indexing
- Locally installed piscem and alevin-fry executables registered via ALEVIN_FRY_HOME / simpleaf set-paths

## Outputs

- Expanded reference FASTA (splici: spliced+intronic, or spliceu: spliced+unspliced) in a `ref` subdirectory
- 3-column transcript-to-gene (t2g) mapping file (copied into the `index` subdirectory)
- piscem index built by `piscem build` in an `index` subdirectory

## How to apply

First ensure piscem and alevin-fry are discoverable by setting ALEVIN_FRY_HOME and running `simpleaf set-paths`, which stores executable paths in a configuration file in the ALEVIN_FRY_HOME directory. Run `simpleaf index` with --fasta and --gtf pointing at the reference genome FASTA and matching GTF; by default this builds an expanded reference of type spliced+intronic (splici) using roers (invoked as a library by simpleaf) with default k-mer length 31 and minimizer length 19. After the expanded reference is constructed, simpleaf invokes `piscem build` on it, producing the piscem index; when `--use-piscem` is set at index time, subsequent `simpleaf quant` runs automatically use piscem (with `piscem map-scrna`) as the mapper against this index. roers optionally supports --dedup to deduplicate identical sequences while building the expanded reference. Verify the output directory contains a `ref` subdirectory holding the splici reference FASTA plus the 3-column transcript-to-gene (t2g) file, and an `index` subdirectory holding the piscem index and a copy of the t2g file.

## Related tools

- **simpleaf** (Exposes the indexing phase as the `simpleaf index` sub-command; orchestrates roers for expanded-reference construction and invokes piscem build) — https://github.com/COMBINE-lab/simpleaf
- **piscem** (RNA back-end mapper and indexer; `piscem build` indexes the expanded reference produced by roers) — https://github.com/COMBINE-lab/piscem
- **roers** (Used as a library directly from simpleaf to construct the spliced+intronic (splici) expanded reference from the genome FASTA and GTF) — https://github.com/COMBINE-lab/roers
- **alevin-fry** (Downstream quantification ecosystem that consumes the index via piscem map-scrna; simpleaf requires it and registers it via set-paths) — https://github.com/COMBINE-lab/alevin-fry
- **wget** (Required dependency of simpleaf (used to fetch reference resources such as FASTA/GTF files)) — https://www.gnu.org/software/wget/

## Examples

```
simpleaf index --use-piscem --fasta ref.fa --gtf annotation.gtf -r splici -k 31 -m 19 -o index_dir
```

## Evaluation signals

- `simpleaf index` completes without error
- The output directory contains a `ref` subdirectory holding the splici reference FASTA
- A 3-column transcript-to-gene (t2g) file exists and is copied alongside the piscem index in the `index` subdirectory
- piscem index files exist in the `index` subdirectory and can subsequently be mapped against by `piscem map-scrna` during `simpleaf quant`
- k-mer length (default 31, must be odd and <= 31) is greater than the minimizer length (default 19, must be < k), consistent with piscem build constraints

## Limitations

- If you have difficulty building a piscem index, the piscem README advises first increasing the file handle limit (e.g. `ulimit -n 2048`) before filing a GitHub issue
- piscem build's -k parameter must be odd and <= 31, and -m (minimizer size) must be < k
- On Apple silicon, running `piscem build` with -t greater than the number of high-performance cores has been observed to severely degrade performance
- piscem build's --work-dir (temporary cDBG files, default ./workdir.noindex) and --tmp-dir (SSHash external minimizer-sort scratch) can matter on HPC systems with small /tmp or cgroup memory limits

## Evidence

- [intro] simpleaf encapsulates the process of creating an expanded reference for quantification into a single command (`index`): [evidence span withheld — anchor status: not_found]
- [methods] it will take a reference genome FASTA and GTF as input, from which it can build a spliced+intronic (splici) reference or a spliced+unspliced (spliceu) reference: [evidence span withheld — anchor status: not_found]
- [methods] after the expanded reference is constructed, the resulting reference will be indexed with ``piscem build``: "index construction with piscem build"
- [readme] The `simpleaf` program can be installed from source, from [crates.io](https://crates.io/crates/simpleaf), or via [bioconda](https://bioconda.github.io/recipes/simpleaf/README.html).: [evidence span withheld — anchor status: external_doc]
- [readme] If you have difficulty building a `piscem` index, before filing an issue on GitHub, please make sure you try: [evidence span withheld — anchor status: external_doc]
