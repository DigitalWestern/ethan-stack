---
name: writing-for-agents
description: Use when writing anything an agent will read, such as a prompt or brief for another agent, a skill, or an AGENTS.md or CLAUDE.md file. Also use when creating or editing skills.
disable-model-invocation: false
---

Reference for writing any document an agent consumes, whether a skill, an `AGENTS.md` / `CLAUDE.md`, or a doc reached by a pointer. The packaging differs, but the writing does not. The same levers make each one predictable, since the agent takes the same _process_ every run rather than producing the same output.

When the document you're writing is a skill, read [`SKILL-MECHANICS.md`](SKILL-MECHANICS.md) for frontmatter, invocation choice, and router skills.

## Context pointers

A **context pointer** is a reference held in the agent's context that names some out-of-context material and encodes the condition for reaching it. A skill's description is one, and a line in `AGENTS.md` naming a doc is the same object. The pointer's _wording_, not its target, decides when the agent reaches the material, and how reliably. A must-have target behind a weakly worded pointer is a variance bug. Sharpen the wording first, and inline the material only if sharpening fails.

A pointer does two jobs. It states what the material is, and it lists the **branches** that should trigger reaching it (a branch is a distinct case the document handles, so different runs take different paths through it). Every word of an always-loaded pointer costs on every turn, so it earns even harder pruning than the body:

- **Front-load the leading word.** The pointer is where it does its triggering work.
- **One trigger per branch.** Synonyms that rename a single branch are one branch written twice. Collapse them and keep only genuinely distinct branches.
- **Cut identity the body already carries.**

## The two loads

Every document and pointer you add spends one of two budgets:

- **Context load** is the cost of always-loaded material on the agent's window. An `AGENTS.md` line, a skill description, or anything else sitting in context every turn spends tokens and attention whether or not it fires.
- **Cognitive load** is the cost on the human, who must know which documents exist and when to reach for each. The human is the index. This is not a cost to minimise. It is the price of human agency, so spend it where human judgement matters and remove it where it does not.

Material reached only through a pointer escapes context load at the price of the pointer's own line. Material with no pointer at all rides entirely on cognitive load.

## Information hierarchy

A document is built from two content types, **steps** (the ordered actions the agent performs) and **reference** (definitions, rules, facts consulted on demand). The two mix freely. A document can be all steps (a recipe), all reference (a review's rules, this skill), or both. The core decision is where each piece sits on the **information hierarchy**, a ladder ranked by how immediately the agent needs the material:

1. **In-file step** is the primary tier, covering what the agent does, in order.
2. **In-file reference** is consulted on demand. It is often a legitimately flat peer-set (every rule of a review on one rung), which is a fine arrangement, not a smell.
3. **Disclosed reference** is pushed out into a separate file, reached by a context pointer, and loaded only when the pointer fires. It spans a sibling file in the same folder through fully external reference that lives anywhere and any document can point at.

Push too little down and the top bloats. Push too much and you hide material the agent actually needs. That tension is the whole decision.

**Progressive disclosure** is the move down the ladder (out of the main file and behind a pointer) so the top stays legible. It is not primarily a token optimisation. It is how the hierarchy is protected. Branching is the cleanest disclosure test. Inline what every branch needs, and push behind a pointer what only some branches reach. When a document has steps, in-file reference that should be disclosed buries them and turns attending to them into a coin-flip. That makes it a variance lever, not just a legibility one.

**Co-location** is the within-file companion. Where the ladder decides _how far down_ a piece sits, co-location decides _what sits beside it_ once there. Keep a concept's definition, rules, and caveats under one heading rather than scattered, so reading one part brings its neighbours with it. The test is that the document should read like documentation written for the agent. Grouped material reads that way, and scattered material does not. (This is distinct from duplication. Duplication repeats one meaning in two places, while scattering fragments one meaning across many.)

**Sprawl** is the failure mode here, a document simply too long even when every line is live and unique. Attention thins across the excess, and every extra line is one more to keep relevant. The cure is the ladder. Disclose reference behind pointers, and split by branch or sequence so each path carries only what it needs.

## Steps and completion criteria

Every step ends on a **completion criterion**, the condition that tells the agent the work is done. Two properties make it a lever:

- **Clarity** asks whether the agent can tell done from not-done. A vague bound ("understanding reached") invites **premature completion**, which is ending the step before it is genuinely done because attention slipped to _being done_. The visible steps still ahead (the **post-completion steps**) supply the pull, and the criterion's clarity is the resistance. Defend in order. **Sharpen the bound first** (local and cheap). Only if it is irreducibly fuzzy _and_ you observe the rush, hide the later steps by splitting the sequence. Hiding only works across a real context boundary, such as a hand-off or a subagent dispatch. An inline call leaves the later steps in context and clears nothing.
- **Demand** is how much it requires. "Every modified model accounted for" forces thorough work where "produce a change list" does not. Demand drives **legwork** (the digging the agent does within the work, latent in the wording rather than written as its own step), and it is not step-bound. "Every rule applied" binds a body of flat reference just as "every step done" binds a sequence, which is how an all-reference document still carries an exhaustiveness bar.

The strongest criteria are both checkable and exhaustive.

## When to split

Splitting one document into two spends one of the two loads, so split only when the cut earns it:

- **By sequence**, split a run of steps where the post-completion steps tempt the agent to rush the one in front of it. Keeping them out of view drives more legwork on the current task. Beware the reverse. Merging sequences exposes each step's later steps to what follows, inviting premature completion.
- **By invocation**, which is skill-specific. See [`SKILL-MECHANICS.md`](SKILL-MECHANICS.md).

## Leading words

A **leading word** is a compact concept already living in the model's pretraining that the agent thinks with while running the document (_lesson_, _fog of war_, _tracer bullets_). Repeated as a token, never as a sentence, it accumulates a distributed definition and anchors a whole region of behaviour in the fewest tokens, by recruiting priors the model already holds. Coining your own works if you define it clearly, but a made-up word recruits no priors. You pay in definition tokens what a pretrained word gives free, so reach for an existing word first.

It anchors twice. In the body it anchors _execution_, because the agent reaches for the same behaviour every time the word appears, and inside flat reference it focuses attention on a class of thing to look for. In a pointer it anchors _invocation_. When the same word lives in your prompts, your docs, and your codebase, the agent links that shared language to the material and reaches it more reliably.

Hunt for opportunities to refactor with leading words. Look for a triad spelled out at three sites, or a pointer spending a sentence to gesture at one idea. Each is a passage begging to collapse into a single token:

- "fast, deterministic, low-overhead" becomes _tight_ (a _tight_ loop).
- "a loop you believe in" becomes _red_, turning a fuzzy gate into a binary observable state (the loop goes _red_ on the bug, or it doesn't).

You win twice, with fewer tokens and a sharper hook for the agent to hang its thinking on. Assume every document is carrying restatements that leading words retire. Go find them.

**Negation** is the failure mode beside this lever. Steering by prohibition drags the forbidden behaviour into context and makes it _more_ available, not less. _Don't think of an elephant_, and the elephant is all there is. The negation is a weak modifier the strongly-activated concept overruns, so the ban half-reads as an instruction to do the thing. Prompt the **positive** instead. State the target behaviour ("write one-line comments") so the banned one is never spoken. A prohibition earns its place only as a hard guardrail you cannot phrase positively. Even then, pair it with the positive target so attention lands on what to do.

## Pruning

- Keep each meaning in a **single source of truth**, one authoritative place, so changing the behaviour is a one-place edit. **Duplication** (the same meaning in more than one place) costs maintenance and tokens, and inflates a meaning's prominence on the ladder past its real rank. (It is the accidental inverse of a leading word, which repeats a token on purpose, never the meaning.)
- The **environment** is a source of truth too (`package.json` scripts, config files, the directory layout, `--help` output), and a document that restates it is a **cache**. A cache is a copy of a lookup, earning its load only when the lookup is expensive. Cache what the agent cannot find by looking, such as the unwritten convention, the reason behind a choice, or the gotcha no config confesses. Leave the one-file, one-command lookups to the environment, where they cannot go stale.
- Check every line for **relevance**. Does it still bear on what the document does? A line loses relevance by never bearing on the task (mere exposition, or a branch that should be disclosed) or by going stale as the behaviour or world it describes changes. Shorter documents are easier to keep relevant. Without a pruning discipline the default fate is **sediment**, meaning stale layers that settle because adding feels safe and removing feels risky, until you must core down through them to find what is still live.
- Hunt **no-ops** sentence by sentence. A no-op is an instruction the model already obeys by default, paying load to say nothing. The test (does it change behaviour versus the default?) is model-relative, not reader-relative. Two people disagreeing about a no-op disagree about the default, and they settle it by running the document, not by debate. When a sentence fails, delete the whole sentence rather than trim words from it. The test also grades leading words. A word too weak to beat the default (_be thorough_ when the agent is already thorough-ish) is a no-op, and the fix is a stronger word (_relentless_), not a different technique.
