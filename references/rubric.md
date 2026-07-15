# Diagnostic Rubric

Score the user's prompt against each dimension below. Report only dimensions with findings. Every finding must quote the offending span from the user's prompt (or name the absence), name the dimension, and name the principle violated. Severity levels:

- **Blocker**: the task will likely fail or produce the wrong artifact.
- **Degrader**: the task will complete but with avoidable quality loss.
- **Polish**: marginal gain; mention only if few higher-severity findings exist.

## 1. Task clarity and specificity

Check: Is the instruction explicit about what to produce? Vague verbs ("help with", "look at", "improve"), undefined scope, and unstated quality expectations force the model to guess. Models respond well to clear, explicit instructions; "above and beyond" behavior must be requested explicitly rather than inferred ([Anthropic: Be clear and direct](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). Apply the golden rule: if a colleague with minimal context would be confused by the prompt, the model will be too.

Symptoms: "make it better", "do something with this data", missing success criteria, no verb-forward command. Specific, direct instructions outperform clever or imprecise descriptions ([promptingguide: General Tips](https://www.promptingguide.ai/introduction/tips)).

Severity: usually Blocker.

## 2. Context and motivation

Check: Does the prompt supply the background the task needs, and the *why* behind constraints? Explaining why a behavior matters helps the model generalize and deliver targeted responses ([Anthropic: Add context](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). Relevance matters more than volume: unnecessary detail dilutes the signal ([promptingguide: General Tips](https://www.promptingguide.ai/introduction/tips)).

Symptoms: audience unstated, domain assumptions unshared, constraint given without rationale, references to material the model cannot see ("the doc", "our standard").

Severity: Blocker when the task depends on missing facts; Degrader otherwise.

## 3. Output specification

Check: Format, length, structure, and audience stated? Three levers work reliably: tell the model what to do instead of what not to do; use format indicator tags; match prompt style to desired output style ([Anthropic: Control the format](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). A "Desired format:" block is a proven pattern for extraction tasks ([promptingguide: General Tips](https://www.promptingguide.ai/introduction/tips)).

Symptoms: no format stated for structured output, only negative instructions ("no markdown", "don't be verbose"), no length target where length matters.

Severity: Blocker for extraction/classification/transformation; Degrader for open generation.

## 4. Constraints and exclusions

Check: Scope boundaries present where the task invites sprawl? State what is in scope and what is explicitly out. For agentic or code tasks, unconstrained prompts invite over-engineering; explicit minimal-scope language counters it ([Anthropic: Overeagerness](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)).

Symptoms: open-ended asks with no boundary ("write about X"), no exclusion list where the topic has obvious adjacent territory, no handling instruction for edge cases ("if no relevant quotes, say so").

Severity: Degrader; Blocker when sprawl defeats the purpose.

## 5. Examples

Check: Would 3 to 5 examples materially improve consistency? Examples are among the most reliable levers for format, tone, and structure. Good examples are relevant (mirror the real use case), diverse (cover edge cases without teaching accidental patterns), and structured (wrapped in `<example>` tags) ([Anthropic: Use examples effectively](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). Research finding worth teaching: the label space and format of demonstrations matter even when individual labels are wrong ([promptingguide: Few-shot](https://www.promptingguide.ai/techniques/fewshot), citing Min et al. 2022). Examples do not fix multi-step reasoning failures; that calls for a reasoning scaffold instead ([promptingguide: Few-shot](https://www.promptingguide.ai/techniques/fewshot)).

Symptoms: format-critical task with zero examples; examples present but homogeneous, off-domain, or unfenced from instructions.

Severity: Degrader in general; Blocker for strict-format extraction on smaller models.

## 6. Reasoning scaffold

Check: Multi-step analytical task with no room or direction to reason? Chain-of-thought (intermediate reasoning steps) lifts arithmetic, commonsense, and symbolic reasoning; zero-shot CoT is as simple as appending "Let's think step by step" ([promptingguide: CoT](https://www.promptingguide.ai/techniques/cot)). Calibrate to the model: current reasoning-capable models often do better with general instructions ("consider this thoroughly") than hand-written step lists, and a self-check appended at the end ("Before you finish, verify your answer against [criteria]") reliably catches errors; when reasoning modes are off, structure manual CoT with `<thinking>` and `<answer>` tags ([Anthropic: Thinking capabilities](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). CoT is an emergent ability of sufficiently large models; see model-targets.md before recommending it for small local models ([promptingguide: CoT](https://www.promptingguide.ai/techniques/cot)).

Symptoms: math/logic/analysis prompt demanding a bare answer, prescriptive micro-step lists given to a strong reasoning model, no verification ask on correctness-critical output.

Severity: Blocker for reasoning-heavy tasks; Degrader elsewhere.

## 7. Structure and delimiters

Check: Are instructions, context, data, and examples separated? Mixed content invites misinterpretation. For Claude, wrap each content type in its own XML tag with consistent, descriptive names, nesting where hierarchy exists ([Anthropic: XML tags](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). Model-agnostic: any clear separator (e.g. `###`, fenced blocks) beats none ([promptingguide: General Tips](https://www.promptingguide.ai/introduction/tips)).

Symptoms: pasted document flows straight into instructions, examples indistinguishable from the live task, variable input not marked.

Severity: Degrader; Blocker for long or multi-document prompts.

## 8. Information ordering

Check: For long inputs (roughly 20k+ tokens), longform data belongs at the top of the prompt, above query, instructions, and examples; queries at the end improved response quality by up to 30% in Anthropic's tests. For long-document tasks, asking the model to quote relevant parts first grounds the answer ([Anthropic: Long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). For short prompts, placing instructions at the beginning with a separator is the common recommendation ([promptingguide: General Tips](https://www.promptingguide.ai/introduction/tips)).

Symptoms: question buried above a wall of pasted text, instructions sandwiched mid-document.

Severity: Degrader; rises with input length.

## 9. Role and framing

Check: Would a role improve tone and focus? Even a single role sentence in the system position focuses behavior for the use case ([Anthropic: Give Claude a role](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). In consumer chat, role framing at the top of the message (or in project/custom instructions where the product offers them) serves the same function.

Symptoms: expert-audience task with no persona or register set, tone drifting because none was requested.

Severity: Polish; Degrader for tone-sensitive deliverables.

## 10. Model fit

Check: Does the prompt match the declared target model's conventions and capability class? Load `references/model-targets.md` and apply the matching section. If the user has not named a target, ask once, offering "unknown / generic" as an out.

Severity: varies; template and capability mismatches on local models are Blockers.
