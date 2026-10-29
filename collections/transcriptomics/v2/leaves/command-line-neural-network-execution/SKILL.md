---
name: command-line-neural-network-execution
description: Use when fixed-length biological sequences or scRNA-seq samples need
  to be encoded into matrices (dict encoding) and trained/evaluated with a Keras deep
  learning model via a standalone CLI without writing custom training code.
when_to_use_negative:
- Input samples are not fixed-length and cannot be encoded at the --spcLen sample
  length with dict encoding
- The task requires visualizing hidden-layer representations (use layerUMAP instead
  of plain running.py)
- The user needs a web/noncoding interface (autoBioSeqpy is a command-line/Jupyter
  tool, not a webserver)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: autoBioSeqpy
  role: Provides the running.py and predicting.py standalone scripts that encode sequences
    into matrices, train Keras models, and save model/weights/parameters/predictions
  repo: https://github.com/jingry/autoBioSeqpy
- name: Keras
  role: Backend deep learning framework in which the model definition loaded via --modelLoadFile
    is executed, and whose model/weight files are saved
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:72c86c38da4ad831b01dca04221564bbab444ace6c4e4f432e4722b6b8928322
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jingry/autoBioSeqpy
  repo_url: https://github.com/jingry/autoBioSeqpy
  tools:
  - Keras
  - autoBioSeqpy
  - layerUMAP
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# command-line-neural-network-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and evaluate deep learning classifiers (e.g. CNN, BiLSTM, RFCN) for biological sequence and scRNA-seq cell-type classification entirely from the command line using autoBioSeqpy's running.py/predicting.py scripts, with Keras model, weights, parameters, predictions, and figures saved to an output folder.

## When to use

Use when fixed-length biological sequences or scRNA-seq samples need to be encoded into matrices (dict encoding) and trained/evaluated with a Keras deep learning model via a standalone CLI without writing custom training code.

## When NOT to use

- Input samples are not fixed-length and cannot be encoded at the --spcLen sample length with dict encoding
- The task requires visualizing hidden-layer representations (use layerUMAP instead of plain running.py)
- The user needs a web/noncoding interface (autoBioSeqpy is a command-line/Jupyter tool, not a webserver)

## Inputs

- Training and test data text files (e.g. train_pos.txt/train_neg.txt) with fixed-length samples
- Class label lists passed via --dataTrainLabel / --dataTestLabel (e.g. 1 0)
- Keras model definition file specified via --modelLoadFile (e.g. protein_CNN_model.py, protein_biLSTM_model.py)
- Training parameters text file (parameters.txt) for --paraFile-based invocation

## Outputs

- Keras model JSON (--modelSaveName tmpMod.) and weights (--weightSaveName tmpWeight.bin)
- Training parameters file (tmpOut/parameters.txt)
- Per-sample predictions and figures in the output folder (--savePrediction 1 --saveFig 1)
- Test-set predictions via predicting.py (optionally written to --predictionSavePath)

## How to apply

Install the dependencies (re, numpy, importlib, matplotlib, sklearn, keras) via anaconda3 or pip, then set the autoBioSeqpy folder as the working path or append it via sys.path. Run 'python running.py' with --dataType, --dataEncodingType dict, --dataTrainFilePaths/--dataTrainLabel and --dataTestFilePaths/--dataTestLabel, --modelLoadFile pointing to a model definition file (e.g. protein_CNN_model.py or an RFCN model), --spcLen for the fixed sample length, --batch_size 60, --epochs 20, --shuffleDataTrain 1, and --noGPU 0 to enable GPU. Save artifacts with --outSaveFolderPath tmpOut, --modelSaveName tmpMod., --weightSaveName tmpWeight.bin, and --paraSaveName parameters.txt, enabling --savePrediction 1 and --saveFig 1. For inference on new data with a trained model, call 'python predicting.py --paraFile tmpOut/parameters.txt' with --dataTestFilePaths and optionally --predictionSavePath; long parameter lists can be written to a text file and passed via --paraFile.

## Related tools

- **autoBioSeqpy** (Provides the running.py and predicting.py standalone scripts that encode sequences into matrices, train Keras models, and save model/weights/parameters/predictions) — https://github.com/jingry/autoBioSeqpy
- **Keras** (Backend deep learning framework in which the model definition loaded via --modelLoadFile is executed, and whose model/weight files are saved)

## Examples

```
python running.py --dataType protein --dataEncodingType dict --dataTrainFilePaths examples/typeIII_secreted_effectors_prediction/data/train_pos.txt examples/typeIII_secreted_effectors_prediction/data/train_neg.txt --dataTrainLabel 1 0 --dataTestFilePaths examples/typeIII_secreted_effectors_prediction/data/test_pos.txt examples/typeIII_secreted_effectors_prediction/data/test_neg.txt --dataTestLabel 1 0 --modelLoadFile examples/typeIII_secreted_effectors_prediction/model/protein_CNN_model.py --verbose 1 --outSaveFolderPath tmpOut --savePrediction 1 --saveFig 1 --batch_size 60 --epochs 20 --spcLen 100 --shuffleDataTrain 1 --modelSaveName tmpMod. --weightSaveName tmpWeight.bin --noGPU 0 --paraSaveName parameters.txt
```

## Evaluation signals

- running.py --help runs without error, confirming the dependency environment (numpy, matplotlib, sklearn, keras) is resolved
- The output folder (e.g. tmpOut) contains tmpMod., tmpWeight.bin, parameters.txt, prediction files, and figures
- Re-running the training with --shuffleDataTrain 1 and the same parameters (batch_size 60, epochs 20) yields comparable test accuracy
- predicting.py --paraFile tmpOut/parameters.txt loads the saved model/weights without retraining and reproduces consistent predictions on held-out test files
- On the scRNA-seq benchmarks, RFCN overall accuracy should approach the reported 98.09% (Zheng sorted), 99.78% (Pancreas), and 100% (CellBench)

## Limitations

- All dependent modules must be resolved (anaconda3 recommended; Python 2 is not suggested) before the CLI works
- Runtime is approximate and hardware-dependent; GPU usage is controlled only by the --noGPU flag (0 enables GPU)
- Too many parameters for the command line; users must rely on --paraFile text files for complex configurations

## Evidence

- [other] Train the RFCN architecture by running 'python running.py' with the model definition loaded via --modelLoadFile, batch_size 60, epochs 20: [evidence span withheld — anchor status: not_found]
- [readme] All modules are included in anaconda3, users could resolve module dependencies by installing anaconda3 (2 is not suggested): "Since all modules are included in **anaconda3**, users could resolve module dependencies by installing anaconda3 (2 is not suggested) on their official website"
