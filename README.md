# phi3-qlora-text-to-sql
Fine-tuning Phi-3-mini for text-to-SQL using QLoRA on a free T4 GPU.
Fine-tuned Microsoft’s Phi-3-mini (3.8B parameters) to generate SQL queries from natural language questions and table schemas, using QLoRA on a single free-tier T4 GPU.

What I did:

-Loaded the base model in 4-bit NF4 precision with double quantization, cutting memory roughly 4x so it fit on consumer-grade hardware.
-Trained LoRA adapters on the attention and MLP layers instead of updating all weights, using Hugging Face transformers, peft, bitsandbytes, and trl.
-Trained on 2,000 examples from the sql-create-context dataset and evaluated on 40 held-out queries.
-Found and fixed a silent misconfiguration: Phi-3 fuses its q/k/v projections into one layer, so my Llama-style target module names meant only one layer was being trained. I diagnosed it by inspecting the model’s actual layer names, then corrected the config and retrained.
-Fixed several other issues along the way: GPU precision limits (fp16 vs bf16), library version conflicts, a generation bug caused by dropout staying active at inference, and checkpoint reloading.

-Results: 75% SQL structure and keyword accuracy and 86% average output similarity on held-out queries. Strict exact-match was 0%, because the model misreproduced arbitrary table and column identifiers (like table_name_97) while getting the query logic right.
