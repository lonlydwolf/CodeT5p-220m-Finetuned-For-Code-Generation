# CodeT5p-220M Fine-Tuned for Code Generation 🚀

This repository contains the Jupyter Notebook and training pipeline for
fine-tuning **Salesforce's `codet5p-220m`** model to generate **Python
code from natural language descriptions**.

The project leverages **Quantized LoRA (QLoRA)** and
**Parameter-Efficient Fine-Tuning (PEFT)** to efficiently train the
model on consumer-grade hardware while maintaining high performance.

The pipeline is built using the **Hugging Face ecosystem**
(`transformers`, `peft`, `trl`, and `datasets`).

------------------------------------------------------------------------

# 🌟 Project Highlights

-   **Base Model:** `Salesforce/codet5p-220m`
-   **Dataset:** `codeparrot/xlcost-text-to-code` (Python program-level
    split)
-   **Training Method:** Supervised Fine-Tuning (SFT) using
    `trl.SFTTrainer`
-   **Efficiency:** 4-bit quantization via `bitsandbytes` (NF4 format)
    and LoRA targeting attention weights
-   **Tracking:** Integration with **Weights & Biases (W&B)** for
    experiment tracking
-   **Final Model:** Available on Hugging Face\
    `LonlyDWolf/CodeT5p-220M-For-Code-Generation`

------------------------------------------------------------------------

# 🛠️ Requirements & Installation

The notebook is designed to run in a **GPU-accelerated environment**
such as:

-   Kaggle
-   Google Colab
-   Local CUDA setup

Install the required dependencies:

``` bash
pip install -U transformers peft trl accelerate evaluate datasets==2.16 nltk rouge_score
pip install -i https://pypi.org/simple/ bitsandbytes wandb
```

------------------------------------------------------------------------

# 🔑 Environment Setup

This notebook requires authentication with **Hugging Face** and
**Weights & Biases**.

⚠️ **Do not hardcode tokens in the notebook.**

Use environment variables or a `.env` file instead:

``` bash
HF_TOKEN=your_huggingface_token
WANDB_API_KEY=your_wandb_key
```

------------------------------------------------------------------------

# 🧠 Training Details

## Prompt Formatting

The model is trained using a **structured prompt format** to clearly
separate the user instruction from the generated code output.

    ### USER: {natural_language_description}
    ### ASSISTANT: {python_code}

------------------------------------------------------------------------

## LoRA Configuration

We use **Parameter-Efficient Fine-Tuning (PEFT)** to train adapters
instead of the full model weights, drastically reducing memory usage.

  Parameter                  Value
  -------------------------- ------------------------
  Rank (`r`)                 8
  Alpha (`lora_alpha`)       16
  Target Modules             `["q", "k", "v", "o"]`
  Dropout (`lora_dropout`)   0.1
  Bias                       none
  Task Type                  `CAUSAL_LM`

------------------------------------------------------------------------

## Quantization Configuration

To fit the model into limited VRAM, the base model is loaded in **4-bit
precision**.

  Setting         Value
  --------------- -----------------
  Load in 4-bit   True
  Compute Dtype   `torch.float16`
  Quant Type      `nf4`

------------------------------------------------------------------------

# 📦 Model Export & Merging Workflow

The notebook includes the **complete end-to-end workflow**:

1.  **Training** -- Supervised fine-tuning of the LoRA adapter weights.
2.  **Saving** -- Exporting the adapter model to a local directory or
    Hugging Face Hub.
3.  **Reloading** -- Loading the base `codet5p` model in **FP32
    (unquantized)**.
4.  **Merging** -- Injecting LoRA weights into the base model using
    `merge_and_unload()`.
5.  **Publishing** -- Pushing the final standalone merged model to the
    **Hugging Face Hub**.

------------------------------------------------------------------------

# 💻 Quickstart: Using the Fine-Tuned Model

You can use the merged model directly with the **Hugging Face
`transformers` library**.

``` python
import torch
from transformers import AutoTokenizer, T5ForConditionalGeneration

tokenizer = AutoTokenizer.from_pretrained("Salesforce/codet5p-220m")
model = T5ForConditionalGeneration.from_pretrained(
    "LonlyDWolf/CodeT5p-220M-For-Code-Generation",
    torch_dtype=torch.float32,
    device_map="auto"
)

instruction = "Write a Python function to calculate the factorial of a given number."
prompt = f"### USER: {instruction}\n### ASSISTANT: "

inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
outputs = model.generate(**inputs, max_length=256)

generated_code = tokenizer.decode(outputs[0], skip_special_tokens=True)
print(generated_code)
```

------------------------------------------------------------------------

# 📝 License

This project is **open-source**.

Please ensure compliance with the licenses of:

-   `Salesforce/codet5p-220m`
-   `codeparrot/xlcost-text-to-code`

when using or distributing this code.
