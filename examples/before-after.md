# Worked Examples

Three graded transformations, one per common task class. Each shows the diagnosis shape the skill should produce and a rewrite whose changelog maps edits to rubric dimensions and techniques.

---

## Example 1: Generation (target: Claude)

**Before**

```
Write a blog post about zero trust. Make it good and not too long. Don't use jargon.
```

**Diagnosis (excerpt)**

- [Blocker] Task clarity — "Make it good" — explicit instructions beat inferred quality expectations; the model must guess what "good" means here, so it will guess generically.
- [Blocker] Context and motivation — no audience is stated — without knowing who reads this, register, depth, and examples cannot be chosen deliberately.
- [Degrader] Output specification — "not too long" and "Don't use jargon" — both are negative or vague; positive instructions ("write for X at Y length in Z register") steer more reliably than prohibitions.
- [Polish] Role and framing — no persona set — one role sentence would lock tone.

**After**

```
You are a security consultant writing for small-business owners with no IT staff.

Write a blog post titled "Zero Trust, Explained for People Who Run a Business, Not a Network."

Audience: owners of 5-50 person companies who buy IT services.
Goal: after reading, they can ask their IT provider three informed questions about zero trust.
Length: 700-900 words.
Register: plain professional English; define any necessary technical term in one clause the first time it appears.
Structure: open with a one-paragraph story of a breach that zero trust would have contained, then explain the principle, then close with the three questions they should ask.
```

**Changelog**

- Added role sentence → Role and framing → role prompting.
- Named audience and goal → Context and motivation → clarity principle (golden rule: a cold reader could now execute this).
- Replaced "good / not too long / no jargon" with positive, concrete specs → Output specification → tell the model what to do, not what to avoid.
- Added structure directive → Task clarity → explicit instruction.

---

## Example 2: Extraction (target: unknown / generic)

**Before**

```
Here are some support tickets. Pull out the important stuff.

[500 lines of pasted tickets]

Also tell me which ones are urgent.
```

**Diagnosis (excerpt)**

- [Blocker] Output specification — "Pull out the important stuff" — extraction without a declared schema produces a different shape every run; state the fields.
- [Blocker] Structure and delimiters — pasted tickets flow into instructions with no separator — the model cannot reliably tell data from directive; fence the data.
- [Degrader] Information ordering — the second instruction ("Also tell me which ones are urgent") sits below the data — put data first, all instructions and the query together at the end for long inputs.
- [Degrader] Examples — none, for a format-critical task — one or two demonstrations would lock the schema (few-shot; format consistency matters even more than any single label).

**After**

```
### DATA
[500 lines of pasted tickets]
### END DATA

### TASK
From each ticket in DATA, extract:

Ticket: <id>
Customer: <name or "unknown">
Issue: <one sentence>
Urgency: <high | normal | low>

Urgency rules: "high" if the ticket mentions outage, data loss, or a deadline within 48 hours; otherwise "normal"; "low" for feature requests.

Example output for one ticket:
Ticket: 4412
Customer: Acme Corp
Issue: VPN drops every 30 minutes for all remote staff.
Urgency: high

Output only the extracted records, one blank line between tickets.
```

**Changelog**

- Declared field schema with a "Desired format" block → Output specification → format specification.
- Fenced data with `###` separators, instructions after data → Structure and delimiters; Information ordering → generic delimiter pattern; data-top/query-end ordering.
- Defined the urgency label set with rules → Task clarity → explicit label space (classification embedded in extraction).
- Added one worked demonstration → Examples → few-shot format conditioning.

---

## Example 3: Reasoning on a small local model (target: 8B-class model on llama.cpp)

**Before**

```
Think step by step and tell me which of these three vendor quotes is the best deal over 3 years, considering the discount tiers and the renewal escalators. [three quotes pasted]
```

Reported symptom: "it rambles and the arithmetic is wrong every time."

**Diagnosis (excerpt)**

- [Blocker] Model fit — "Think step by step" as the load-bearing lever — chain-of-thought is an emergent ability of sufficiently large models; on an 8B-class model it produces reasoning-shaped text, not reliable arithmetic. Symptom matches.
- [Blocker] Model fit (serving layer) — before rewording anything, verify the runtime is applying the model's correct chat template with the recommended flags; wrong templates corrupt output in ways that look like prompt failure.
- [Degrader] Task clarity — "best deal" — undefined; define the objective function (lowest total cost? lowest cost per seat?).
- [Degrader] Reasoning scaffold — one monolithic prompt — decompose: extraction is within a small model's reach; multi-step arithmetic is where it fails.

**After** (multi-turn: prompt chaining, with arithmetic moved out of the model)

Turn 1:

```
### DATA
[three quotes pasted]
### END DATA

From each quote in DATA, extract exactly these fields, numbers only:

Vendor: <name>
Year1_price: <number>
Discount_pct: <number>
Escalator_pct: <number>

Output the three records and nothing else.
```

Turn 2 (after verifying the extraction by eye): compute the 3-year totals yourself in a spreadsheet or script from the extracted numbers, then, if a narrative comparison is wanted:

```
Given these computed 3-year totals: [numbers], write a five-sentence recommendation.
Best deal means lowest 3-year total cost. State the winner in the first sentence.
```

**Changelog**

- Dropped CoT as the primary lever → Model fit → CoT emergence caveat for small models.
- Split extraction from arithmetic from narrative → Reasoning scaffold → prompt chaining (decomposition).
- Moved arithmetic to a deterministic tool → Model fit → program-aided pattern (PAL analog: compute, don't estimate).
- Defined "best deal" → Task clarity → explicit success criterion.
- Added serving-layer check to the advice → Model fit → template verification precedes prompt rewrites.
