# Model Targets

Apply the section matching the user's declared target. If the target was not declared and the user chose "unknown / generic," apply only the Universal section.

## Universal (any model)

These hold everywhere: clarity and specificity; relevant context; explicit output format; separation of instructions, context, examples, and data with consistent delimiters; instructions placed prominently (top for short prompts); iterate rather than over-engineer the first draft ([promptingguide: General Tips](https://www.promptingguide.ai/introduction/tips)).

## Claude (claude.ai, Claude Desktop, Claude apps)

Source for this section: [Anthropic prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices).

- **XML tags are the native delimiter.** Wrap instructions, context, input, and examples in their own tags; use consistent descriptive names; nest where hierarchy exists (`<documents>` containing `<document index="n">`).
- **Instruction following is literal.** Current Claude models do what the prompt says, closely. Request "above and beyond" quality explicitly; request actions explicitly ("make these edits," not "can you suggest changes") when action is wanted.
- **Long inputs: data at top, query at end.** Anthropic reports up to 30% quality improvement from end-placed queries on complex multi-document inputs. Add quote grounding ("first extract the relevant quotes") for long-document tasks.
- **Format control:** positive instructions over prohibitions; format-indicator tags; match the prompt's own style to the desired output style (a markdown-heavy prompt begets markdown-heavy output).
- **Examples:** 3 to 5, relevant, diverse, in `<example>`/`<examples>` tags.
- **Reasoning:** prefer general guidance ("consider this thoroughly," "reason through the tradeoffs") over prescriptive step lists; append a self-check for correctness-critical work. If steering away from reasoning modes, note that some models are sensitive to the literal word "think"; alternatives include "consider" and "evaluate."
- **Role:** one sentence of persona at the top of the message, or in project/custom instructions where the product provides them.

## Other frontier chat models (ChatGPT, Gemini, and peers)

The Universal section carries most of the weight. Delimiters: `###` separators or fenced blocks are the widely cited generic pattern ([promptingguide: General Tips](https://www.promptingguide.ai/introduction/tips)). Structural conventions and reasoning-mode behaviors vary by vendor and change between releases; do not assert vendor-specific behaviors from memory. Point the user to the vendor's current prompting guide and to promptingguide.ai's model pages ([index](https://www.promptingguide.ai/models)) for specifics.

## Local and open-weight models (Ollama, llama.cpp, LM Studio; small models generally)

Two failure planes exist here, and the second masquerades as the first.

### Serving-layer checks before any prompt rewrite

The chat template is the silent killer. Local runtimes must format the conversation with the model's own template; a missing or broken one corrupts output in ways users blame on their prompt.

- GGUF publishers warn that omitting the correct template flag produces wrong output outright (example: "please only use --jinja otherwise the output will be wrong!", [unsloth GLM-4.6 discussion](https://huggingface.co/unsloth/GLM-4.6-GGUF/discussions/2)).
- Wrong or fallback templates yield garbage tokens even when the model loads and runs ([goose Gemma-4 issue](https://github.com/aaif-goose/goose/issues/9110)); simplified stand-in templates can degrade output badly ([EXAONE 4.0 discussion](https://huggingface.co/LGAI-EXAONE/EXAONE-4.0-32B-GGUF/discussions/1)).
- Symptom cluster pointing at the template, not the prompt: literal `<tool_call>` XML in replies, leaked `</think>` tags, empty tool arguments, repetition loops. Fixes: add the runtime's Jinja flag, use the model card's recommended flags, or serve a community-corrected template file ([troubleshooting reference](https://netclaw.dev/troubleshooting/llama-cpp/)). Some GGUFs ship broken embedded templates; check the model's Hugging Face discussions.
- Structured output is quantization-sensitive: sub-4-bit quants can malform tool calls and strict formats even with a correct template ([same reference](https://netclaw.dev/troubleshooting/llama-cpp/)).

Diagnostic rule: if the user reports garbled tags, format collapse, or nonsense from a local model, direct them to verify template and quantization before touching prompt wording.

### Prompt-side adjustments for small models

- **Do not lean on chain-of-thought.** CoT is an emergent ability of sufficiently large models ([promptingguide: CoT](https://www.promptingguide.ai/techniques/cot)); "think step by step" underdelivers as parameter count drops.
- **Lean on demonstration instead.** Few-shot format and label-space conditioning is the reliable lever ([promptingguide: Few-shot](https://www.promptingguide.ai/techniques/fewshot)); show the exact output shape wanted.
- **Anchor the output hard.** Explicit "Desired format:" blocks, terminal cues, and short leash on length. Small models drift without anchors.
- **Decompose.** Prefer several simple prompts chained by hand over one complex prompt ([promptingguide: Prompt Chaining](https://www.promptingguide.ai/techniques/prompt_chaining)); start simple and add elements iteratively ([promptingguide: General Tips](https://www.promptingguide.ai/introduction/tips)).
- **Mind the context budget.** Long few-shot blocks plus long inputs collide with small context windows; check the model card rather than assuming a limit.
- **Sampling settings interact with adherence.** Direct determinism-sensitive users to the guide's settings page ([promptingguide: LLM Settings](https://www.promptingguide.ai/introduction/settings)) rather than asserting parameter values.
