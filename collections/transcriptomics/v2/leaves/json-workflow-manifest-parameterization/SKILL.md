---
name: json-workflow-manifest-parameterization
description: Use when a user needs to run a configurable multi-step alevin-fry single-cell
  pipeline end-to-end but wants a templated, parameterized workflow manifest rather
  than hand-composing simpleaf/alevin-fry/piscem commands.
when_to_use_negative:
- The user already has a pre-built piscem index and only needs a single quantification
  step — a direct 'simpleaf quant' (with -u unfiltered permit list or -x explicit
  permit list) is simpler than a workflow manifest.
- The user is not using the alevin-fry ecosystem (e.g. bulk RNA-seq or scATAC-seq
  processing outside alevin-fry/piscem map-scrna) — this workflow template machinery
  composes alevin-fry single-cell commands specifically.
- The user needs a fully custom pipeline whose steps diverge from the published self-documented
  templates rather than filling in sample-specific fields.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0085
tools:
- name: simpleaf
  role: Provides the workflow subcommands (list, get) and the workflow executor that
    validates the JSON manifest, plans the command queue, and executes the composed
    commands with logging
  repo: https://github.com/COMBINE-lab/simpleaf
- name: alevin-fry
  role: Backend suite whose commands (generate-permit-list, collate, quant) are composed
    by the instantiated workflow manifest
  repo: https://github.com/COMBINE-lab/alevin-fry
- name: piscem
  role: RNA back-end mapper/indexer; the workflow's expanded-reference indexing (piscem
    build) and read mapping (map-scrna) steps are executed by piscem
  repo: https://github.com/COMBINE-lab/piscem
- name: wget
  role: Required dependency used by simpleaf (e.g. to fetch resources such as published
    workflow templates)
  repo: https://www.gnu.org/software/wget/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:355a4b4b57e37bd05014fa4655fb8f05c504027fa0fb620a60770b7c157dc761
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/simpleaf
  tools:
  - alevin-fry
  - piscem
  - simpleaf
  - wget
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# json-workflow-manifest-parameterization

## Summary

Instantiate a reusable, self-documented simpleaf workflow template (fetched from its published template registry) by filling in sample-specific parameters into a JSON workflow manifest, then execute it with the simpleaf workflow executor to compose a full alevin-fry single-cell pipeline (expanded-reference indexing, mapping, permit-list generation, collate, quant).

## When to use

Use when a user needs to run a configurable multi-step alevin-fry single-cell pipeline end-to-end but wants a templated, parameterized workflow manifest rather than hand-composing simpleaf/alevin-fry/piscem commands.

## When NOT to use

- The user already has a pre-built piscem index and only needs a single quantification step — a direct 'simpleaf quant' (with -u unfiltered permit list or -x explicit permit list) is simpler than a workflow manifest.
- The user is not using the alevin-fry ecosystem (e.g. bulk RNA-seq or scATAC-seq processing outside alevin-fry/piscem map-scrna) — this workflow template machinery composes alevin-fry single-cell commands specifically.
- The user needs a fully custom pipeline whose steps diverge from the published self-documented templates rather than filling in sample-specific fields.

## Inputs

- Published simpleaf workflow template folder ('<workflow_name>_template') obtained via 'simpleaf workflow get'
- User-defined sample parameters: input read FASTQs, reference genome FASTA + GTF (or pre-built reference), and output directory paths
- Instantiated JSON workflow manifest (workflow.json) after template filling
- Installed simpleaf with its required dependencies: alevin-fry, piscem, and wget (paths configurable via 'simpleaf set-paths' stored in ALEVIN_FRY_HOME)

## Outputs

- Filled workflow template files and an instantiated JSON workflow manifest (workflow.json)
- Executor logs from validation, command-queue planning, and execution
- Populated output directory containing the composed pipeline results: expanded reference (splici/spliceu) indexing, piscem mapping, permit-list generation, alevin-fry collate, and alevin-fry quant outputs

## How to apply

List available workflows with 'simpleaf workflow list' and fetch the published self-documented template with 'simpleaf workflow get --name <workflow> --output <dir>', which packs all related files into a folder named '<workflow_name>_template'. Edit the template files, filling in only the user-defined sample parameters such as input reads, reference (e.g. a genome FASTA and GTF for splici/spliceu expanded-reference construction), and output paths. Instantiate the filled template as a JSON workflow manifest (workflow.json). Run it with the simpleaf workflow executor, which validates the manifest, plans the command queue, and executes the composed commands (expanded-reference indexing via piscem build, piscem map-scrna, alevin-fry generate-permit-list, collate, and quant) with logging. The template approach retains the flexibility of the underlying tools while making alevin-fry 'even simpler', because sample-specific information is the only thing a user must supply.

## Related tools

- **simpleaf** (Provides the workflow subcommands (list, get) and the workflow executor that validates the JSON manifest, plans the command queue, and executes the composed commands with logging) — https://github.com/COMBINE-lab/simpleaf
- **alevin-fry** (Backend suite whose commands (generate-permit-list, collate, quant) are composed by the instantiated workflow manifest) — https://github.com/COMBINE-lab/alevin-fry
- **piscem** (RNA back-end mapper/indexer; the workflow's expanded-reference indexing (piscem build) and read mapping (map-scrna) steps are executed by piscem) — https://github.com/COMBINE-lab/piscem
- **wget** (Required dependency used by simpleaf (e.g. to fetch resources such as published workflow templates)) — https://www.gnu.org/software/wget/

## Examples

```
simpleaf workflow get --name <workflow> --output <dir>   # then edit template files and run the instantiated workflow.json with the simpleaf workflow executor
```

## Evaluation signals

- The simpleaf workflow executor accepts the manifest without validation errors (schema/type check of workflow.json against the template schema).
- The workflow executes end-to-end without error, with executor logging showing the planned command queue was fully executed.
- The designated output directory is populated with the outputs of the composed commands: expanded-reference index, mapped RAD file, permit list, collated data, and quant matrix outputs.
- The instantiated manifest contains only sample-specific substitutions (reads, reference, output paths) relative to the published self-documented template.
- The quant output is consistent with the specified chemistry/geometry (UMI and cell-barcode geometry passed to piscem map-scrna) used in the manifest.

## Limitations

- The template mechanism requires the workflow to already exist in the published template set; novel pipeline structures require editing template files rather than arbitrary reshaping.
- Manifest validation and execution depend on all required executables (alevin-fry, piscem, wget) being installed and their paths configured via 'set-paths' in the ALEVIN_FRY_HOME directory.
- Building a piscem index can fail on resource-constrained systems; users are advised to raise the file handle limit (ulimit -n 2048) before filing GitHub issues.

## Evidence

- [other] The workflow module operates via workflow templates: users obtain an existing (self-documented) template, fill in minimal sample-specific information to instantiate it, and run the instantiated template with the simpleaf workflow executor.: [evidence span withheld — anchor status: not_found]
- [other] List available workflows with 'simpleaf workflow list' and fetch the published template with 'simpleaf workflow get --name <workflow> --output <dir>', which packs all related files into a folder named '<workflow_name>_template'.: [evidence span withheld — anchor status: not_found]
- [readme] simpleaf is a rust framework to make using alevin-fry even simpler.: "`simpleaf` is a `rust` framework to make using `alevin-fry` _even_ simpler."
- [intro] simpleaf encapsulates expanded-reference creation into a single index command and sample quantification into a single quant command.: "creating an expanded reference for quantification into a single command (`index`) and the quantification of a sample into a single command (`quant`)"
- [methods] The set-paths command configures executable paths in the ALEVIN_FRY_HOME configuration directory.: "The ``set-paths`` command is used to set the paths to the relevant executables and store them in a configuration file in the ``ALEVIN_FRY_HOME`` directory"
