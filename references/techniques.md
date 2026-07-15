# Technique Taxonomy

Every technique below carries a class. Recommend only what the user can actually execute in their environment:

- **Single-prompt (SP)**: works pasted into one chat message.
- **Multi-turn (MT)**: works as a manual sequence of chat messages.
- **Orchestration (OR)**: needs application scaffolding (sampling loops, retrieval infrastructure, tool runtimes, training). When a user's problem calls for one of these, say so plainly: "this wants an app, not a prompt edit," name the pattern, and offer the nearest SP/MT analog.

When recommending, give 1 to 3 candidates, name each, state the cost, and show a one-line application sketch. Naming the technique is deliberate: it builds the user's vocabulary.

## Single-prompt techniques

**Zero-shot** ([source](https://www.promptingguide.ai/techniques/zeroshot)). Direct instruction, no demonstrations; instruction-tuned models handle many tasks this way. Default starting point. When it fails, escalate to few-shot.

**Few-shot** ([source](https://www.promptingguide.ai/techniques/fewshot)). Demonstrations in the prompt condition the output (in-context learning). Best for format, tone, and pattern tasks. Cost: token spend per example. Quality rules: 3 to 5 examples, relevant, diverse, wrapped in `<example>` tags ([Anthropic](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)); demonstration format and label space drive performance even more than per-example label correctness ([Min et al. via promptingguide](https://www.promptingguide.ai/techniques/fewshot)). Known limit: does not rescue multi-step reasoning; use a reasoning scaffold instead.

**Chain-of-thought, zero-shot form** ([source](https://www.promptingguide.ai/techniques/cot)). Append "Let's think step by step" (Kojima et al.), or the APE-discovered stronger variant "Let's work this out in a step by step way to be sure we have the right answer." ([source](https://www.promptingguide.ai/techniques/ape)). Cost: longer, slower outputs. Emergent with sufficiently large models; weak lever on small local models (see model-targets.md).

**Chain-of-thought, few-shot form** ([source](https://www.promptingguide.ai/techniques/cot)). Demonstrations that show worked reasoning, not just answers. One good worked example can suffice. On models with native reasoning modes, prefer general instructions ("consider this thoroughly") over prescriptive step lists, and add a closing self-check ("Before you finish, verify your answer against [criteria]") ([Anthropic](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).

**Meta prompting** ([source](https://www.promptingguide.ai/techniques/meta-prompting)). Structure-oriented: abstract templates and syntax patterns instead of content examples. More token-efficient than few-shot; a form of zero-shot. Caveat: assumes the model already knows the task domain; degrades on novel tasks. Fit: math, coding, theory tasks where the solution shape is known.

**Tree-of-thought prompting (Hulbert single-prompt variant)** ([source](https://www.promptingguide.ai/techniques/tot)). Simulated multi-expert deliberation in one prompt: "Imagine three different experts are answering this question..." Cheap approximation of the full ToT framework. Fit: problems benefiting from explored alternatives without infrastructure.

**Role prompting** ([Anthropic](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). One sentence of persona focuses behavior and tone.

**Structure and delimiters** ([Anthropic](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices); [promptingguide](https://www.promptingguide.ai/introduction/tips)). XML tags for Claude; any consistent separator generically. See rubric dimension 7.

**Output format specification** ([Anthropic](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). Positive instructions, format-indicator tags, prompt-style matching. See rubric dimension 3.

**Quote grounding** ([Anthropic](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). For long documents: instruct the model to extract relevant quotes before performing the task. Cuts noise; pairs with data-at-top ordering.

## Multi-turn techniques (manual in chat)

**Prompt chaining** ([source](https://www.promptingguide.ai/techniques/prompt_chaining)). Split the task into subtasks; feed each response into the next prompt. Gains transparency, controllability, and debuggability. Canonical pattern: extract quotes from a document in turn one, answer from quotes in turn two.

**Self-correction loop** ([Anthropic: Chain complex prompts](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). Generate a draft, review it against stated criteria, refine from the review. Three turns. This is the practical chat-sized analog of the Reflexion pattern (below).

**Generated knowledge** ([source](https://www.promptingguide.ai/techniques/knowledge)). Turn one: have the model generate relevant facts. Turn two: feed those facts plus the question. Fit: commonsense or knowledge-adjacent questions where the model errs when answering cold.

**Manual self-consistency** (approximation; [source](https://www.promptingguide.ai/techniques/consistency)). Regenerate the same reasoning prompt several times and compare answers. Crude majority vote by hand. Cost: N times the calls. Flag it as an approximation of the real technique, which belongs to orchestration.

**Multimodal reasoning sequence** (analog of [Multimodal CoT](https://www.promptingguide.ai/techniques/multimodalcot)). With image input: ask for a grounded description or rationale first, then the answer that uses it.

## Orchestration techniques (redirect)

Name these when the user's underlying need outgrows a chat prompt. Offer the nearest analog from above.

**Self-consistency** ([source](https://www.promptingguide.ai/techniques/consistency)). Sample multiple diverse reasoning paths via few-shot CoT, select the most consistent answer. Cost multiplies per sample. Analog: manual self-consistency.

**Tree of Thoughts framework** ([source](https://www.promptingguide.ai/techniques/tot)). Thought tree plus self-evaluation plus BFS/DFS search with lookahead and backtracking. Analog: Hulbert single-prompt variant.

**RAG** ([source](https://www.promptingguide.ai/techniques/rag)). Retrieval component feeds documents into the generation context; mitigates hallucination on knowledge-intensive tasks and sidesteps stale parametric knowledge. Chat analogs: paste the source documents yourself (with delimiters and data-at-top ordering), or use the product's built-in search/file features.

**ReAct** ([source](https://www.promptingguide.ai/techniques/react)). Interleaved reasoning traces and tool actions (thought, action, observation). Reduces the fact-hallucination CoT suffers alone, but depends on retrieval quality. Modern tool-enabled chat products implement this natively; the user's lever is asking the assistant to search or verify, not restructuring the prompt.

**PAL / program-aided reasoning** ([source](https://www.promptingguide.ai/techniques/pal)). Model writes code as the reasoning step; a runtime executes it. Analog in chat: use a code-execution-enabled assistant and ask for computed, not estimated, answers.

**Reflexion** ([source](https://www.promptingguide.ai/techniques/reflexion)). Actor, evaluator, and self-reflection memory across episodes; verbal reinforcement for agents that learn from trial and error. Analog: the self-correction loop.

**ART** ([source](https://www.promptingguide.ai/techniques/art)). Task-library demonstration selection with generation paused at tool calls. Agent-framework territory.

**APE** ([source](https://www.promptingguide.ai/techniques/ape)). LLM-driven candidate-instruction generation, execution, scoring, and selection. Prompt optimization as black-box search. Relevant to users building evaluated prompt pipelines, not chat.

**Active-Prompt** ([source](https://www.promptingguide.ai/techniques/activeprompt)). Uncertainty-ranked exemplar selection with human annotation of the most uncertain items. Dataset-scale exemplar engineering.

**Directional Stimulus Prompting** ([source](https://www.promptingguide.ai/techniques/dsp)). A trained policy LM generates hints that steer a frozen model. Analog: hand the model your own hint keywords inline.

**Auto-CoT** ([source](https://www.promptingguide.ai/techniques/cot)). Automated construction of diverse CoT demonstrations via clustering and zero-shot CoT.

**Graph prompting** ([source](https://www.promptingguide.ai/techniques/graph)). GraphPrompt framework for graph-task performance; the guide entry is a stub. Mention only if the user asks.

## Decision table

| Task class | First line | Escalation | Cost note |
|---|---|---|---|
| Generation (posts, docs, creative) | Clarity + output spec + role | Few-shot for voice/format lock | Examples cost tokens |
| Extraction / transformation | Output spec ("Desired format:") + delimiters | Few-shot, 3-5 diverse examples | Blocker-level without format spec |
| Classification | Zero-shot with label set stated | Few-shot; keep label space and format consistent | Format beats label correctness |
| Reasoning / analysis | Zero-shot CoT or general "reason thoroughly" + self-check | Few-shot CoT; manual self-consistency; ToT-prompting for branchy problems | CoT lengthens output; self-consistency multiplies calls |
| Knowledge-dependent QA | Supply sources inline (delimited, data at top) + quote grounding | Generated knowledge; product search (ReAct territory) | Unsourced recall risks hallucination |
| Exploration / ideation | Constraints + quantity ask ("give 10, varied by X") | ToT-prompting; chaining (diverge then converge) | Unbounded asks sprawl |
| Multi-document work | Data top, query end, XML per document, quote grounding | Prompt chaining (extract then synthesize) | Ordering worth up to 30% on long inputs |

## Anti-recommendation rules

- Do not stack techniques for their own sake; each recommendation must map to a diagnosed finding.
- Do not recommend heavy CoT scaffolds to strong reasoning-mode models; recommend general thinking instructions and a self-check instead ([Anthropic](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).
- Do not recommend "think step by step" as a cure-all on small local models; see model-targets.md.
- Do not present an orchestration technique as a prompt fix. Redirect and give the analog.
