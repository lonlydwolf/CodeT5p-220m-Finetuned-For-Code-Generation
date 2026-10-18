# CodeT5+ experiment: technical retrospective

This completed learning experiment explored quantized model loading, LoRA configuration, dataset preparation and model publication through Hugging Face tools. The notebook and published files are useful records of that exploration. They do not establish a measured improvement in code generation or a reproducible application.

This retrospective distinguishes the historical configuration from findings of a later documentation review. It does not reconstruct undocumented personal motivations, contributions or training results.

## What was inspected

On 18 October 2026, the review examined repository commit [`4dedaf2`](https://github.com/lonlydwolf/CodeT5p-220m-Finetuned-For-Code-Generation/commit/4dedaf2b97d6b517a2caac84447450ead1960d3f), the notebook, and small public metadata/configuration files at [Hub revision `141a2d4`](https://huggingface.co/LonlyDWolf/CodeT5p-220M-For-Code-Generation/tree/141a2d4e5c9d471082d1c7d59e87bd943a8aae6e). It did not download weights, train a model, run inference or execute generated code.

The notebook selects the first 5% of XLCoST's Python-program training split and configures 100 optimizer steps. It passes a validation dataset, but records no evaluation schedule, explicit evaluation call, metric output, baseline comparison or test-set result. All notebook code-cell outputs are empty. The Hub files confirm publication, not the exact process that produced every artifact.

## Lessons from the available evidence

### Match the objective to the architecture

[Salesforce describes CodeT5+ as an encoder-decoder family](https://huggingface.co/Salesforce/codet5p-220m). The notebook loads `T5ForConditionalGeneration`, but configures PEFT as `CAUSAL_LM` and concatenates the description and reference code before packing. [PEFT 0.10.0 distinguishes causal and sequence-to-sequence wrappers](https://github.com/huggingface/peft/blob/v0.10.0/src/peft/mapping.py#L60-L66).

In the [contemporaneous TRL implementation inspected](https://github.com/huggingface/trl/blob/v0.8.1/trl/trainer/utils.py#L448-L486), packed input sequences are also used as labels. Applied to this encoder-decoder setup, that would let the encoder see the reference answer while the decoder reconstructs the sequence. It does not match generating code from a description alone.

The precise historical TRL version is not recorded, so this is a source-backed concern rather than proof of how every published weight was trained. The lesson is to inspect a processed batch and the loss objective before relying on a trainer abstraction or interpreting its loss.

### Separate a training configuration from an evaluated result

The use of LoRA and 4-bit NF4 describes a method. Without measurements, it does not show the achieved memory reduction, speed or generation quality. Passing a validation split also does not show that evaluation ran. The appropriate conclusion is that improvement remains unmeasured, not that the model must be ineffective.

Dataset representation matters too: [XLCoST uses code serialization markers](https://huggingface.co/datasets/codeparrot/xlcost-text-to-code), and the notebook supplies no conversion to executable Python. Text similarity, syntax validity and functional correctness would answer different questions about its output.

### Treat artifact publication as another boundary to verify

The notebook's merge path still passes the 4-bit configuration. The [published model config](https://huggingface.co/LonlyDWolf/CodeT5p-220M-For-Code-Generation/blob/141a2d4e5c9d471082d1c7d59e87bd943a8aae6e/config.json) retains bitsandbytes quantization settings, and the Hub contains both model and adapter files. A `float32` dtype field does not make the artifact unquantized. The earlier README's plain-FP32 export description was therefore too strong.

The final notebook cell creates another trainer and pushes without a further training call. That makes exact artifact lineage worth documenting, rather than inferring it from filenames. A verified load-and-generate check would still be needed before offering a quickstart.

### Preserve enough information to reproduce the experiment

The model card records several library versions, but the notebook installs mostly unpinned packages and does not preserve the full environment, dataset revision, hardware configuration or training logs. Its authentication setup and intermediate Hub reference also prevent treating it as a confirmed fresh-kernel workflow. [Recorded versions and current limitations](../README.md#historical-environment) are preserved without claiming they form a tested installation recipe.

## What a future experiment should establish

1. Use description-only encoder inputs and separate code labels for a sequence-to-sequence task, or deliberately choose a decoder-only architecture for causal-LM training. Inspect a tiny batch to confirm the intended behavior.
2. Record dataset/model revisions, sampling rules, seeds, dependency versions and preprocessing, including code-marker conversion where relevant.
3. Compare the base and adapted models on the same held-out examples and decoding settings. Preserve failures and distinguish syntax checks from functional tests; isolate any generated-code execution.
4. Save the training/evaluation record and an artifact manifest, then test the documented loading path in a fresh environment.

These are possible future learning tasks, not an active roadmap or changes delivered by this closeout. The historical notebook remains unchanged. A carefully explained negative result would be useful evidence too; additional training is not required simply to retain the project as a learning record.
