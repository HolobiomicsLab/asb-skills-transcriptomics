---
name: workflow-template-retrieval-from-registry
description: Use when a user needs to reproduce or run a configurable alevin-fry single-cell
  processing pipeline end-to-end and prefers a curated, versioned workflow template
  over hand-composing simpleaf/alevin-fry/piscem commands.
when_to_use_negative:
- The analysis is a single simpleaf command (index or quant) on one sample — encapsulated
  commands are sufficient and a workflow template adds overhead.
- The input is already an instantiated workflow.json ready for the executor — retrieval
  and editing steps should be skipped.
- The desired pipeline requires steps not represented in any published template and
  cannot be expressed by editing sample-specific fields in an existing template.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simpleaf
  role: provides the workflow registry commands (list, get) and the workflow executor
    that validates the JSON manifest, plans the command queue, and runs the composed
    commands
  repo: https://github.com/COMBINE-lab/simpleaf
- name: alevin-fry
  role: underlying single-cell processing suite whose commands (generate-permit-list,
    collate, quant) the retrieved workflow template composes
  repo: https://github.com/COMBINE-lab/alevin-fry
- name: piscem
  role: RNA back-end mapper and indexer invoked by the composed pipeline for index
    construction (piscem build) and read mapping (map-scrna)
  repo: https://github.com/COMBINE-lab/piscem
- name: wget
  role: required dependency of simpleaf used when fetching workflow templates and
    resources
  repo: https://www.gnu.org/software/wget/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:2601b4176c27c9f0de2e9b1525ef1be4dff32c7f54e600e863f6717776c871f6
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# workflow-template-retrieval-from-registry

## Summary

Retrieves and instantiates self-documented workflow templates from the simpleaf workflow registry so that a complex, multi-step alevin-fry single-cell pipeline (expanded-reference indexing, mapping, permit-list generation, collate, quant) can be run as a validated JSON workflow manifest rather than assembled command-by-command.

## When to use

Use when a user needs to reproduce or run a configurable alevin-fry single-cell processing pipeline end-to-end and prefers a curated, versioned workflow template over hand-composing simpleaf/alevin-fry/piscem commands.

## When NOT to use

- The analysis is a single simpleaf command (index or quant) on one sample — encapsulated commands are sufficient and a workflow template adds overhead.
- The input is already an instantiated workflow.json ready for the executor — retrieval and editing steps should be skipped.
- The desired pipeline requires steps not represented in any published template and cannot be expressed by editing sample-specific fields in an existing template.

## Inputs

- self-documented workflow template folder ('<workflow_name>_template') fetched from the simpleaf workflow registry
- user-defined sample parameters: input sequencing reads, reference genome FASTA and GTF, and output paths
- instantiated JSON workflow manifest (workflow.json)

## Outputs

- workflow.json manifest consumed by the simpleaf workflow executor
- executor logs of the planned command queue and composed command execution
- populated output directory containing expanded-reference indexing, mapping (RAD), permit-list generation, collate, and quant results

## How to apply

First list the available workflows with 'simpleaf workflow list', then fetch the chosen published template with 'simpleaf workflow get --name <workflow> --output <dir>', which packs all related files into a folder named '<workflow_name>_template'. Edit the template files to fill in the user-defined sample parameters — input reads, reference (e.g. a genome FASTA and GTF from which a splici/spliceu expanded reference is built), and output paths. Instantiate the filled template as a JSON workflow manifest (workflow.json) and hand it to the simpleaf workflow executor, which validates the manifest, plans the command queue, and executes the composed commands with logging. Judge success by an error-free end-to-end run and a designated output directory populated with the results of the composed pipeline.

## Related tools

- **simpleaf** (provides the workflow registry commands (list, get) and the workflow executor that validates the JSON manifest, plans the command queue, and runs the composed commands) — https://github.com/COMBINE-lab/simpleaf
- **alevin-fry** (underlying single-cell processing suite whose commands (generate-permit-list, collate, quant) the retrieved workflow template composes) — https://github.com/COMBINE-lab/alevin-fry
- **piscem** (RNA back-end mapper and indexer invoked by the composed pipeline for index construction (piscem build) and read mapping (map-scrna)) — https://github.com/COMBINE-lab/piscem
- **wget** (required dependency of simpleaf used when fetching workflow templates and resources) — https://www.gnu.org/software/wget/

## Examples

```
simpleaf workflow list; simpleaf workflow get --name <workflow> --output <dir>; simpleaf workflow run --runcfg workflow.json
```

## Evaluation signals

- The 'simpleaf workflow get' command creates a folder named '<workflow_name>_template' containing all related template files.
- The edited template instantiates into a valid workflow.json that passes the simpleaf workflow executor's validation without error.
- The executor logs show a planned command queue covering expanded-reference indexing, mapping, permit-list generation, collate, and quant.
- The designated output directory is populated with the results of the composed pipeline after an error-free end-to-end run.

## Limitations

- Templates must be obtained and filled per-sample: the registry supplies only the self-documented template, and the user must supply sample-specific reads, references, and output paths before instantiation.
- The executor requires a valid JSON workflow manifest; invalid or incomplete edits to template files will fail validation before any commands are planned.
- The workflow composes commands from the alevin-fry ecosystem (simpleaf, piscem, alevin-fry), so failures in underlying steps (e.g. piscem index builds) surface through the executor run.

## Evidence

- [other] The workflow module operates via workflow templates: users obtain an existing (self-documented) template, fill in minimal sample-specific information to instantiate it, and run the instantiated template with the simpleaf workflow executor.: [evidence span withheld — anchor status: not_found]
- [other] List available workflows with 'simpleaf workflow list' and fetch the published template with 'simpleaf workflow get --name <workflow> --output <dir>', which packs all related files into a folder named '<workflow_name>_template'.: [evidence span withheld — anchor status: not_found]
- [other] Run the workflow with the simpleaf workflow executor, which validates the workflow manifest, plans the command queue, and executes the composed commands with logging.: [evidence span withheld — anchor status: not_found]
- [abstract] We introduce simpleaf, a program that simplifies the processing of single-cell data using tools from the alevin-fry ecosystem, and adds new functionality and capabilities, while retaining the flexibility and performance of the underlying tools.: "We introduce simpleaf, a program that simplifies the processing of single-cell data using tools from the alevin-fry ecosystem"
- [intro] `simpleaf` uses [`piscem`](https://github.com/COMBINE-lab/piscem) as its RNA back-end mapper and indexer.: "`simpleaf` uses [`piscem`](https://github.com/COMBINE-lab/piscem) as its RNA back-end mapper and indexer."
