# CodeT5+ 220M: a code-generation fine-tuning experiment

**Completed learning experiment · retired from active development**

This repository preserves a 2024 notebook exploring LoRA and 4-bit fine-tuning of Salesforce's CodeT5+ 220M on Python description/code pairs. It records an early model-adaptation experiment, with published artifacts and unresolved methodology and reproducibility questions. There is no active development roadmap or ongoing support commitment.

The [historical notebook](fine-tuning-project.ipynb) is preserved unchanged. The [published Hugging Face repository](https://huggingface.co/LonlyDWolf/CodeT5p-220M-For-Code-Generation) contains model and adapter files, but no reported evaluation establishes improved code generation over the base model. This is not presented as a validated code-generation product or a ready-to-run training pipeline.

- [Technical retrospective](docs/TECHNICAL_RETROSPECTIVE.md): what the available evidence supports and what remains unresolved.
- [Proposed Hugging Face model card](docs/MODEL_CARD.md): a local draft for review; adding this file does not update the Hub.

## Recorded experiment configuration

These are settings in the notebook, not independently verified measurements of a completed run.

| Area               | Recorded configuration                                                                                       |
| ------------------ | ------------------------------------------------------------------------------------------------------------ |
| Base model         | `Salesforce/codet5p-220m`, loaded as `T5ForConditionalGeneration`                                            |
| Data               | `codeparrot/xlcost-text-to-code`, `Python-program-level`                                                     |
| Training selection | `train[:5%]`: the first 5% of the training split                                                             |
| Validation         | Validation split passed to the trainer; no evaluation schedule, explicit evaluation call or results recorded |
| Training budget    | 100 optimizer steps; batch size 2; gradient accumulation 8                                                   |
| Learning rate      | `2e-4`, constant schedule, warmup ratio `0.03`                                                               |
| LoRA               | Rank 8, alpha 16, dropout 0.1, targets `q`, `k`, `v`, `o`                                                    |
| PEFT task          | `CAUSAL_LM`; this needs review for the encoder-decoder model                                                 |
| Quantization       | 4-bit NF4, float16 compute via bitsandbytes                                                                  |
| Trainer/input      | TRL `SFTTrainer`, packing enabled; description and reference code concatenated into one string               |

The [dataset card](https://huggingface.co/datasets/codeparrot/xlcost-text-to-code) describes serialized code containing `NEW_LINE`, `INDENT` and `DEDENT` markers. The notebook does not convert that representation back into executable Python.

## Evidence and limitations

A documentation review on **18 October 2026** examined source commit [`4dedaf2`](https://github.com/lonlydwolf/CodeT5p-220m-Finetuned-For-Code-Generation/commit/4dedaf2b97d6b517a2caac84447450ead1960d3f) and [Hub revision `141a2d4`](https://huggingface.co/LonlyDWolf/CodeT5p-220M-For-Code-Generation/tree/141a2d4e5c9d471082d1c7d59e87bd943a8aae6e).

- **Publication is verified; performance is not.** The Hub has model/adapter files and configurations. The notebook has no saved execution outputs, and the published model card has no evaluation results. Missing logs do not prove that training failed or never occurred.
- **The training objective needs investigation.** CodeT5+ 220M is an encoder-decoder model, while the notebook uses a causal-LM task setting and packs description-plus-answer text. With the contemporaneous TRL behavior inspected in the review, that places reference answers in encoder inputs while reproducing the same sequence as labels. The exact historical TRL version is unknown; this is a material methodology concern, not proof of a failed training run.
- **The export is not documented accurately as unquantized FP32.** The reload/merge cell still supplies a quantization configuration, and the published `config.json` retains 4-bit bitsandbytes settings. Model and adapter files coexist; a fresh-environment loading path and exact artifact lineage have not been validated.
- **Reproduction is incomplete.** Most dependencies are unpinned, paths are Kaggle-specific, and authentication setup is incomplete. The intermediate `LonlyDWolf/output` reference was not accessible without authentication during review. Its absence from a public request does not establish deletion.

No training, model-weight download, inference run or generated-code execution was performed in that review. There are no substantiated claims here about model quality, memory savings, latency or improvement over the base model.

## Historical environment

The published model card records the following versions:

| Package      | Recorded version |
| ------------ | ---------------- |
| PEFT         | 0.10.0           |
| Transformers | 4.39.2           |
| PyTorch      | 2.1.2            |
| Datasets     | 2.16.0           |
| Tokenizers   | 0.15.2           |

The notebook metadata identifies Python 3.10.13. These fragments are not a complete tested environment: the TRL, bitsandbytes, CUDA and hardware versions for the historical run are not established. Installing the latest libraries is not a substitute for reconstructing that environment.

No current quickstart is offered. A future fork should first validate preprocessing, task/model compatibility and artifact loading on a small batch, then compare against the base model on held-out data before spending compute on retraining. See the [retrospective](docs/TECHNICAL_RETROSPECTIVE.md#what-a-future-experiment-should-establish).

## License information

This code repository currently has no standalone license file. The [published model card](https://huggingface.co/LonlyDWolf/CodeT5p-220M-For-Code-Generation) declares BSD-3-Clause; the [base-model](https://huggingface.co/Salesforce/codet5p-220m) and [dataset](https://huggingface.co/datasets/codeparrot/xlcost-text-to-code) cards document their own terms. This documentation closeout does not assign a new license or change those declarations.
