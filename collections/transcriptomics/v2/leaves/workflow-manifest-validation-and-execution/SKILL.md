---
name: workflow-manifest-validation-and-execution
description: Use when a multi-step single-cell processing pipeline (splici/spliceu
  reference construction, piscem indexing, mapping, permit-list generation, collate,
  quant) must be declared as a validated JSON manifest and executed end-to-end rather
  than run as ad-hoc commands.
when_to_use_negative:
- The task is a single sample quantification with fixed parameters — the direct `simpleaf
  index` / `simpleaf quant` commands already encapsulate the whole process without
  a template
- The user's pipeline does not fit an existing published simpleaf workflow template,
  since the module operates by obtaining and instantiating existing templates
- Only reference construction or mapping is needed without downstream quantification
  — the full manifest executes the complete composed pipeline including quant
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: simpleaf
  role: Framework whose workflow module lists, fetches, instantiates, validates, and
    executes JSON workflow manifests for composed alevin-fry pipelines
  repo: https://github.com/COMBINE-lab/simpleaf
- name: alevin-fry
  role: Underlying suite whose commands (generate-permit-list, collate, quant) are
    composed and executed by the workflow manifest
  repo: https://github.com/COMBINE-lab/alevin-fry
- name: piscem
  role: RNA back-end mapper/indexer invoked by the composed pipeline for index construction
    (piscem build) and mapping (piscem map-scrna)
  repo: https://github.com/COMBINE-lab/piscem
- name: wget
  role: Required dependency used by simpleaf to fetch workflow template assets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad614
    title: '<tt>simpleaf</tt> : a simple, flexible, and scalable framework for single-cell
      data processing using alevin-fry'
  source_content_hash: sha256:7c53c27a004589556f79634d14c49a175f4865f7edb19c5051955a34b81d6af8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/simpleaf
  tools:
  - alevin-fry
  - piscem
  - simpleaf
  - wget
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# workflow-manifest-validation-and-execution

## Summary

Instantiate and run reproducible alevin-fry single-cell pipelines via the simpleaf workflow module: fetch a self-documented workflow template, fill in sample-specific parameters to produce a JSON workflow manifest, and let the simpleaf workflow executor validate the manifest, plan the command queue, and execute the composed commands (expanded-reference indexing, mapping, permit-list generation, collate, quant).

## When to use

Use when a multi-step single-cell processing pipeline (splici/spliceu reference construction, piscem indexing, mapping, permit-list generation, collate, quant) must be declared as a validated JSON manifest and executed end-to-end rather than run as ad-hoc commands.

## When NOT to use

- The task is a single sample quantification with fixed parameters — the direct `simpleaf index` / `simpleaf quant` commands already encapsulate the whole process without a template
- The user's pipeline does not fit an existing published simpleaf workflow template, since the module operates by obtaining and instantiating existing templates
- Only reference construction or mapping is needed without downstream quantification — the full manifest executes the complete composed pipeline including quant

## Inputs

- simpleaf workflow template folder ('<workflow_name>_template') fetched with `simpleaf workflow get`
- User-filled sample parameters: input read FASTQs, reference genome FASTA and GTF, output directory paths
- Instantiated JSON workflow manifest (workflow.json)
- Installed executables: alevin-fry, piscem, wget (paths configurable via simpleaf set-paths)

## Outputs

- Validated and executed workflow with a planned command queue and execution logs
- Populated output directory containing the results of the composed pipeline: expanded-reference index, mapped RAD output, permit list, collated data, and alevin-fry quantification

## How to apply

First list available workflows with `simpleaf workflow list` and fetch the desired published, self-documented template with `simpleaf workflow get --name <workflow> --output <dir>`, which packs all related files into a folder named '<workflow_name>_template'. Edit the template files to supply user-defined sample parameters such as input reads, reference (FASTA/GTF for the splici/spliceu expanded reference), and output paths. Instantiate the filled template as a JSON workflow manifest (workflow.json). Invoke the simpleaf workflow executor, which validates the workflow manifest, plans the command queue, and executes the composed commands with logging; the composed pipeline spans expanded-reference indexing, mapping, permit-list generation, collate, and quant. The rationale is that the executor enforces manifest validity and command ordering before any command runs, so failures surface at planning time rather than mid-pipeline.

## Related tools

- **simpleaf** (Framework whose workflow module lists, fetches, instantiates, validates, and executes JSON workflow manifests for composed alevin-fry pipelines) — https://github.com/COMBINE-lab/simpleaf
- **alevin-fry** (Underlying suite whose commands (generate-permit-list, collate, quant) are composed and executed by the workflow manifest) — https://github.com/COMBINE-lab/alevin-fry
- **piscem** (RNA back-end mapper/indexer invoked by the composed pipeline for index construction (piscem build) and mapping (piscem map-scrna)) — https://github.com/COMBINE-lab/piscem
- **wget** (Required dependency used by simpleaf to fetch workflow template assets)

## Examples

```
simpleaf workflow list; simpleaf workflow get --name <workflow> --output <dir>
```

## Evaluation signals

- The workflow executor completes manifest validation without error before any command runs
- The workflow executes end-to-end without error through all composed stages (indexing, mapping, permit-list generation, collate, quant)
- The designated output directory is populated with the outputs of the composed commands (expanded-reference index, RAD file, permit list, quant results)
- The executor's command queue plan matches the expected pipeline stage ordering (expanded-reference indexing → mapping → permit-list → collate → quant)
- Execution logs record each composed command and its successful completion

## Limitations

- The workflow module depends on published, self-documented templates; a user cannot run a workflow that does not exist as a template without authoring one
- simpleaf requires alevin-fry, piscem, and wget to be installed and discoverable (paths set via set-paths in the ALEVIN_FRY_HOME configuration file)
- piscem index building may fail if the file handle limit is not raised (ulimit -n 2048 is recommended before running piscem)

## Evidence

- [methods] workflow module operates via templates and executor: [evidence span withheld — anchor status: not_found]
- [results] executor validation and command queue: [evidence span withheld — anchor status: not_found]
- [methods] composed pipeline stages: [evidence span withheld — anchor status: not_found]
- [methods] template fetch command: [evidence span withheld — anchor status: not_found]
- [readme] simpleaf simplifies alevin-fry usage: "`simpleaf` is a `rust` framework to make using `alevin-fry` _even_ simpler."
- [readme] required dependencies: "`simpleaf` requires [`alevin-fry`](https://github.com/COMBINE-lab/alevin-fry), [`piscem`](https://github.com/COMBINE-lab/piscem), and [`wget`](https://www.gnu.org/software/wget/)."
