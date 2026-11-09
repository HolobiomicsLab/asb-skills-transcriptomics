---
name: simpleaf-cli-configuration
description: Use when you need to run alevin-fry-based single-cell preprocessing through
  simpleaf's CLI — e.g. configuring ALEVIN_FRY_HOME, setting executable paths, or
  building a splici/spliceu reference index from genome FASTA + GTF.
when_to_use_negative:
- You need custom mapper options beyond what simpleaf's quant/index flags expose —
  call piscem map-scrna and alevin-fry directly instead
- The reference is already an expanded (splici/spliceu) FASTA with a t2g file — go
  straight to piscem build rather than re-running simpleaf index
- You are processing bulk RNA-seq or scATAC-seq — simpleaf targets single-cell scRNA-seq
  processing with alevin-fry
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: simpleaf
  role: CLI wrapper that exposes the index and quant sub-commands and calls roers
    and piscem internally
  repo: https://github.com/COMBINE-lab/simpleaf
- name: piscem
  role: RNA back-end mapper and indexer invoked via `piscem build` on the expanded
    reference
  repo: https://github.com/COMBINE-lab/piscem
- name: alevin-fry
  role: downstream single-cell processing suite (permit-list generation, collate,
    quant) driven through simpleaf
  repo: https://github.com/COMBINE-lab/alevin-fry
- name: roers
  role: library used directly from simpleaf to build the expanded splici/spliceu reference
    from FASTA + GTF
  repo: https://github.com/COMBINE-lab/roers
- name: wget
  role: required dependency of simpleaf for fetching reference resources
  repo: https://www.gnu.org/software/wget/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:5ae6eb61fed7b95d719885659d09ab47b99b2da1405903d101616d10edc5096f
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
yes_rate: 0.4166666666666667
n_claims: 12
---

# simpleaf-cli-configuration

## Summary

Configure and drive the simpleaf command-line interface so it orchestrates alevin-fry single-cell processing, including locating dependency executables via set-paths and building expanded-reference indices via the index sub-command. simpleaf encapsulates reference preparation and quantification into single commands while retaining alevin-fry/piscem performance.

## When to use

Use when you need to run alevin-fry-based single-cell preprocessing through simpleaf's CLI — e.g. configuring ALEVIN_FRY_HOME, setting executable paths, or building a splici/spliceu reference index from genome FASTA + GTF.

## When NOT to use

- You need custom mapper options beyond what simpleaf's quant/index flags expose — call piscem map-scrna and alevin-fry directly instead
- The reference is already an expanded (splici/spliceu) FASTA with a t2g file — go straight to piscem build rather than re-running simpleaf index
- You are processing bulk RNA-seq or scATAC-seq — simpleaf targets single-cell scRNA-seq processing with alevin-fry

## Inputs

- Reference genome FASTA file (e.g. from Ensembl or GENCODE)
- Matching GTF annotation file
- Optional probe-set CSV (for probe-based indexing paths)

## Outputs

- Output directory with a `ref` subdirectory containing the splici reference FASTA
- `index` subdirectory containing the piscem index
- 3-column transcript-to-gene (t2g) mapping file copied alongside the index

## How to apply

First install the dependencies (alevin-fry, piscem, wget) and export the ALEVIN_FRY_HOME environment variable, then run `simpleaf set-paths` so simpleaf records the paths to piscem and alevin-fry executables in a configuration file under ALEVIN_FRY_HOME. For indexing, run `simpleaf index` with --fasta and --gtf pointing at a reference genome FASTA and matching GTF; by default this builds the spliced+intronic (splici) expanded reference via roers (used as a library directly from simpleaf), with default k-mer length 31 and minimizer length 19, then invokes `piscem build` on the result. If the --use-piscem flag is set at index time, subsequent `simpleaf quant` calls against this index automatically use piscem as the mapper with appropriate parameters. Validation: the run succeeds only if the output directory contains a `ref` subdirectory with the splici reference FASTA, an `index` subdirectory with the piscem index, and a 3-column transcript-to-gene file.

## Related tools

- **simpleaf** (CLI wrapper that exposes the index and quant sub-commands and calls roers and piscem internally) — https://github.com/COMBINE-lab/simpleaf
- **piscem** (RNA back-end mapper and indexer invoked via `piscem build` on the expanded reference) — https://github.com/COMBINE-lab/piscem
- **alevin-fry** (downstream single-cell processing suite (permit-list generation, collate, quant) driven through simpleaf) — https://github.com/COMBINE-lab/alevin-fry
- **roers** (library used directly from simpleaf to build the expanded splici/spliceu reference from FASTA + GTF) — https://github.com/COMBINE-lab/roers
- **wget** (required dependency of simpleaf for fetching reference resources) — https://www.gnu.org/software/wget/

## Examples

```
ALEVIN_FRY_HOME=~/.alevin_fry simpleaf set-paths; simpleaf index --fasta genome.fa --gtf annotation.gtf --output af_index --use-piscem
```

## Evaluation signals

- `simpleaf index` completes without error
- Output directory contains a `ref` subdirectory holding the splici reference FASTA
- `index` subdirectory holds the piscem index files plus the copy of the 3-column transcript-to-gene file
- The t2g file is 3-column formatted and matches the splici reference transcript names
- With --use-piscem set, subsequent `simpleaf quant` invocations resolve piscem from the ALEVIN_FRY_HOME configuration and map against the index without manual path arguments

## Limitations

- simpleaf requires piscem, alevin-fry, and wget to be installed and discoverable; if piscem index build fails, increase the file handle limit (e.g. `ulimit -n 2048`) before filing a GitHub issue
- piscem k-mer size must be odd and <= 31 and the minimizer size must be < k, constraining index parameters
- On Apple silicon, use a thread count not exceeding the number of high-performance cores during piscem build, as efficiency cores severely degrade build performance

## Evidence

- [methods] set-paths configuration: "The ``set-paths`` command is used to set the paths to the relevant executables and store them in a configuration file in the ``ALEVIN_FRY_HOME`` directory"
- [methods] expanded reference construction and indexing: "it will take a reference genome FASTA and GTF as input, from which it can build a spliced+intronic (splici) reference or a spliced+unspliced (spliceu) reference"
- [methods] piscem build invocation: "after the expanded reference is constructed, the resulting reference will be indexed with ``piscem build``"
- [intro] index encapsulation: "`simpleaf` encapsulates the process of creating an expanded reference for quantification into a single command (`index`)"
- [methods] roers as library: "using `roers <https://github.com/COMBINE-lab/roers>`_  (which is used as a library directly from ``simpleaf``"
- [readme] piscem file handle limit: [evidence span withheld — anchor status: external_doc]
- [readme] roers splici definition: [evidence span withheld — anchor status: external_doc]
