---
license: mit
base_model: TinyLlama/TinyLlama-1.1B-Chat-v1.0
task: text-generation
tags: [lora, tinyllama, domain-name-generation, safety-refusals]
library_name: peft
---

# Domain Name Generator — TinyLlama Baseline (LoRA)
Adapters only (PEFT/LoRA). Load on top of `TinyLlama/TinyLlama-1.1B-Chat-v1.0`.
Trained on synthetic v1 dataset (seed=42). JSON-only IO with safety refusals.
