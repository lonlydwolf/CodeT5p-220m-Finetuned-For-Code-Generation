---
license: bsd-3-clause
library_name: peft
tags:
  - trl
  - sft
  - generated_from_trainer
base_model: Salesforce/codet5p-220m
datasets:
  - generator
model-index:
  - name: CodeT5p-220M-For-Code-Generation
    results: []
---

<!-- Proposed replacement README.md for https://huggingface.co/LonlyDWolf/CodeT5p-220M-For-Code-Generation. Review before publishing. This local draft does not update the Hub. Existing metadata and license declaration are preserved. -->

# CodeT5+ 220M: historical code-generation experiment

**Completed learning experiment · retired from active development**

This repository contains artifacts from a 2024 experiment exploring LoRA and 4-bit fine-tuning of `Salesforce/codet5p-220m` on Python description/code pairs. The [source notebook and retrospective](https://github.com/lonlydwolf/CodeT5p-220m-Finetuned-For-Code-Generation) document the historical configuration and unresolved limitations.

These are experimental artifacts, not an evaluated code-generation product. No published results establish an improvement over the base model, and no current fresh-environment inference workflow is verified. There is no ongoing support commitment.

## Artifact and review scope

A documentation review on 18 October 2026 inspected public metadata and small configuration files at revision [`141a2d4e5c9d471082d1c7d59e87bd943a8aae6e`](https://huggingface.co/LonlyDWolf/CodeT5p-220M-For-Code-Generation/tree/141a2d4e5c9d471082d1c7d59e87bd943a8aae6e), alongside source commit [`4dedaf2`](https://github.com/lonlydwolf/CodeT5p-220m-Finetuned-For-Code-Generation/commit/4dedaf2b97d6b517a2caac84447450ead1960d3f).

The repository includes `model.safetensors`, `adapter_model.safetensors`, model/adapter configurations and tokenizer assets. The review did not download weights, train, run inference or execute generated code. The existence of these files establishes publication, not exact training lineage or model quality.

## Data and recorded training setup

The notebook uses [`codeparrot/xlcost-text-to-code`](https://huggingface.co/datasets/codeparrot/xlcost-text-to-code), configuration `Python-program-level`, selecting `train[:5%]` and passing the validation split to the trainer. The inherited `generator` dataset metadata above is an automatically generated label, not a separate identified training corpus; the notebook names XLCoST.

XLCoST represents code with markers such as `NEW_LINE`, `INDENT` and `DEDENT`. The notebook does not implement conversion back to executable Python.

Recorded notebook settings, rather than independently verified run measurements:

| Setting                 | Value                                                         |
| ----------------------- | ------------------------------------------------------------- |
| Model class             | `T5ForConditionalGeneration`                                  |
| Training budget         | 100 optimizer steps                                           |
| Training batch size     | 2 per device                                                  |
| Gradient accumulation   | 8                                                             |
| Learning rate           | `0.0002`                                                      |
| Scheduler/warmup        | Constant / `0.03`                                             |
| Optimizer setting       | `paged_adamw_32bit`                                           |
| LoRA rank/alpha/dropout | 8 / 16 / 0.1                                                  |
| LoRA targets            | `q`, `k`, `v`, `o`                                            |
| PEFT task setting       | `CAUSAL_LM`                                                   |
| Quantization            | 4-bit NF4; float16 compute                                    |
| Trainer/input           | TRL `SFTTrainer`; packed description-plus-reference-code text |

The original generated card additionally recorded seed 42, evaluation batch size 8 and total training batch size 16. These inherited values are retained as historical metadata about the reported setup, not evidence of a reproduced run.

## Evaluation and intended use

No baseline comparison, held-out generation metrics, functional correctness results, latency measurements or peak-memory measurements are available in the source notebook or this card. The notebook has no saved execution outputs and does not record an evaluation schedule or an explicit evaluation call. Providing a validation dataset does not demonstrate that evaluation occurred.

Use these artifacts as a historical learning reference. Their quality and suitability for downstream use remain unestablished. Missing execution records do not prove that training failed or never occurred.

## Known methodology and loading limitations

- **Task/model mismatch needs investigation.** CodeT5+ 220M is an encoder-decoder model, while the notebook uses a causal-LM PEFT setting and packs descriptions together with reference answers. With the contemporaneous TRL behavior inspected during review, the full packed text becomes both encoder input and labels, exposing reference answers during training. The exact historical TRL version is unknown; this is a methodology concern, not proof of a failed run. The inspected sources were [TRL v0.8.1 packing](https://github.com/huggingface/trl/blob/v0.8.1/trl/trainer/utils.py#L448-L486) and [PEFT v0.10.0 task mapping](https://github.com/huggingface/peft/blob/v0.10.0/src/peft/mapping.py#L60-L66).
- **This is not established as an unquantized FP32 export.** The notebook's reload/merge cell still supplies a quantization configuration. Published `config.json` retains 4-bit bitsandbytes settings, even though it also declares a float32 dtype.
- **Artifact lineage and loading require validation.** Model and adapter files coexist. The final notebook cell constructs another trainer and pushes without another training call. A confirmed loading path, compatibility requirements and inference results are not provided.
- **The environment is incomplete.** Most notebook installations are unpinned; paths are Kaggle-specific. The intermediate `LonlyDWolf/output` reference was not publicly accessible without authentication during review. No conclusion about its deletion is implied.

## Historical framework versions

The original generated model card recorded:

- PEFT 0.10.0
- Transformers 4.39.2
- PyTorch 2.1.2
- Datasets 2.16.0
- Tokenizers 0.15.2

The source notebook metadata identifies Python 3.10.13. These are not a complete tested environment: historical TRL, bitsandbytes, CUDA and hardware versions remain unestablished. No ready-to-run quickstart is claimed.

## License and provenance

The existing model-repository declaration, **BSD-3-Clause**, is retained. Consult the [Salesforce base-model card](https://huggingface.co/Salesforce/codet5p-220m) and [XLCoST dataset card](https://huggingface.co/datasets/codeparrot/xlcost-text-to-code) for their respective provenance and terms. This documentation update does not assign a license to the separate notebook repository or change existing declarations.
