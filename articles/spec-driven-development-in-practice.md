# Spec-Driven Development in Practice

- Date: February 24th, 2026
- Category: Technical
- Authors: Ezequiel González
- URL: https://www.marvik.ai/blog/spec-driven-development-in-practice

---

#### Designing work for AI systems instead of fighting them

‍

## Introduction — Why I started exploring Spec-Driven Development

Over the last weeks I’ve been working on a project focused on automatic code generation . Not demos, not toy examples, real workflows, real constraints, real iteration.

In the early stages, while running isolated experiments, everything seemed to work just fine. Modern models are genuinely very good at writing code, so initial results looked promising and progress felt fast.

But very quickly, a pattern showed up: the problem wasn’t the quality of the models, it was how I was structuring the work , with too much reliance on manual fixes and not enough upfront structure.

That’s what pushed me to dig deeper into Spec-Driven Development (SDD) . Not as a theory, but as a practical response to the friction of building software with LLMs involved in every step.

#### The pre-SDD workflow — Prompting works… until it doesn’t

My initial workflow looked familiar:

- I had an idea or requirement

- I wrote a prompt

- I got code

- I tweaked it

- I repeated the loop

This works surprisingly well… at the beginning.

The problems appear when:

- requirements change

- the system grows in size and complexity

- or you need to reproduce results reliably

What kept breaking wasn’t the code. It was intent .

The “why” behind decisions lived in my head, not in the system. Each new prompt was trying to reconstruct context that was never explicitly captured.

#### Context first — before starting SDD

One key realization while adopting Spec-Driven Development is that code generation quality depends on context quality .

When context only lives in prompts or in someone’s head, results tend to drift, iteration becomes expensive, and manual fixes pile up. SDD works best when intent is made explicit before any plan or code is generated.

To see the difference in real-time, compare these two levels of "contextual depth":

- Low Context: > "Import a PDF." The Problem: The AI has to guess everything: the language, the size limits, the architecture, and the error handling. You’ll likely get a snippet that doesn't fit your project.

- The Problem: The AI has to guess everything: the language, the size limits, the architecture, and the error handling. You’ll likely get a snippet that doesn't fit your project.

- Higher Context (The SDD starting point): "Import a PDF asynchronously, validating that the size is less than 5 MB, using a separate 'FileManager' class to handle the logic." The Win: You’ve defined the behavior (async), the constraint (5MB), and the architecture (extra class). You aren't just asking for code; you are providing a blueprint.

- The Win: You’ve defined the behavior (async), the constraint (5MB), and the architecture (extra class). You aren't just asking for code; you are providing a blueprint.

Several tools approach this problem from different angles, but they all aim at the same goal: establishing durable context outside the prompt.

#### Three complementary ways to define context

In the traditional workflow, "context" is just whatever you can fit into a single prompt before the AI starts hallucinating. In Spec-Driven Development, context is a structured environment. It’s the "operating system" in which your AI agents live.

To build this environment, we need to decompose context into three fundamental elements:

- The Behavioral Layer (Steering): These are the non-negotiables. It’s the "personality" of your codebase. Does your team prefer functional programming? Do you use specific naming conventions for your files? Steering ensures that every agent, no matter when it’s called, follows the same philosophical direction.

- The Situational Layer (Memory): This is the "tribal knowledge" of your project. It’s the record of why you chose Library A over Library B, or the specific edge case you found in the last sprint. Memory prevents the "Groundhog Day" effect, where you have to explain the same constraint to the AI every single morning.

- The Operational Layer (Rules): These are the "guardrails" for the immediate task. It’s the syntax, the folder structure, and the specific API constraints. Rules act as a real-time linter for the AI’s thought process, stopping errors before they even become code.

By separating these three, you stop being a "prompt engineer" and start being a Systems Architect. You aren't just giving orders; you are designing the boundaries within which intelligence operates.

This is exactly what modern tooling is starting to solve through different lenses:

- Kiro — Steering files: Defines long-term direction and architectural intent to ensure alignment and avoid improvisation.

- Claude — Memory: Enables continuity by persisting important decisions and preferences across sessions. ‍

- Cursor — Rules: Provides control through explicit, local constraints that prevent low-level mistakes during execution.

#### Team Adoption: Sharing Context for Consistency

To effectively implement Spec-Driven Development (SDD) across a team, the core context layers, specifically the Situational Layer (Memory) and the Operational Layer (Rules) , must be shared, visible, and version-controlled.

The most straightforward approach is to commit these contextual files directly into the project repository, allowing all team members and all AI agents to operate from a single source of truth.

The rules/ directory contains low-level, immediate constraints that govern code quality, structure, and execution for the immediate task.

The memory/ directory captures high-level decisions, historical context, and project-specific knowledge that prevent repetitive instruction or "Groundhog Day" scenarios.

#### A simple example structure

In practice, this usually maps to a small, intentional structure:

```
project-root/
├─ specs/
│  ├─ system.md
│
├─ steering/
│  ├─ principles.md
│
├─ rules/
│  ├─ coding.md
│
├─ memory/
│  ├─ decisions.md
```

The exact tooling may vary, but the pattern remains the same:

separate direction, constraints, and persistent context so AI systems can operate with less manual intervention and fewer post-generation fixes.

For deeper details, the official documentation is the best place to start:

- https://code.claude.com/docs/en/memory

- https://cursor.com/en-US/docs/context/rules

- https://kiro.dev/docs/steering/

This explicit context layer is not separate from Spec-Driven Development, it is the foundation that makes SDD possible in practice.

The integration into the team workflow is straightforward and discipline-driven:

- Read the Context: Before starting a task, a developer (or the specialized AI agent acting on their behalf) first ingests the relevant steering/, rules/, and memory/ files.

- Define the Spec: The developer writes a new specification (specs/new-feature.md) that references the constraints and history.

- Update the Context: If a new architectural decision is made during the execution of the spec, the developer must update the memory/decisions.md file and push the change alongside the resulting code.

- Review the Context Changes: In pull requests, the review process should include scrutiny of any proposed changes to the memory/ and rules/ files to ensure they accurately reflect the team's consensus.

#### What Spec-Driven Development means (in practice)

With context in place, Spec-Driven Development flips the center of gravity.

Instead of:

- prompts → code → fixes

You move to:

- specs → plans → execution → review

In practice, SDD means:

- Specs are the primary input

- Code is a derived artifact

- Intent is explicit and inspectable

This is not:

- writing more documentation

- waterfall in disguise

- over-engineering

It is:

- designing constraints

- defining scope

- making decisions visible to both humans and models

#### Specs as a control layer for AI-assisted development

LLMs are excellent executors.

They are not great at guessing boundaries.

Specs act as:

- contracts

- alignment mechanisms

- scope governors

This idea is reflected across several tools and approaches:

- GitHub Spec Kit

- Kiro steering files

- Tessl’s skills, docs, and rules

- Martin Fowler’s SDD work

#### The new workflow — SDD in action

#### Tooling and references that shaped my approach

- GitHub Spec Kit

- Kiro steering files

- Tessl — skills, docs, rules as primitives

- Martin Fowler — Exploring GenAI & SDD

Claude is used mainly as an executor inside this structure.

#### Building my own SDD workflow

I forked an existing repository and evolved it into my own workflow:

https://github.com/perzequiel/perze-claude-code

It includes:

- custom agents

- spec → plan → execution loops

- an intentionally extensible structure

#### From writing code to coordinating agents

I spend less time in the IDE and more time:

- defining specs

- reviewing plans

- evaluating trade-offs

The value shifts from implementation to problem structuring .

#### Trade-offs and current limitations

Costs:

- more upfront design

- more review cycles

- discipline required

Benefits:

- less chaos

- better reasoning

- clearer systems

#### What’s next (short spoiler)

- MCPs

- more specialized skills

- tighter agent collaboration

Details for another post.

#### A note on Warp

I’m using Warp as a work operating system with multi-agent workflows, abstracting me further from the IDE.

Talk:

https://www.youtube.com/watch?v=w0bJFC0u0pE

## Closing

In the modern development landscape, the sheer accessibility of powerful tools and frameworks has dramatically shifted the focus of engineering effort. When code becomes cheap, design becomes the work. This isn't to say that writing code is trivial, but rather that the cost, in terms of time, complexity, and expertise, of producing working code has decreased relative to the cost of producing well-designed, sustainable, and correct systems. The value lies in thoughtful architecture, clear interfaces, and a shared understanding of intent.

Spec-Driven Development is one practical way to make that design explicit, reviewable, and executable. SDD mandates that the definition of a system's behavior, its specification, or "spec", is created before or concurrently with the implementation. This spec serves as the single source of truth, moving the design process out of the developer's head and into a formalized, accessible artifact. By formalizing the design through executable specifications, SDD transforms abstract architectural concepts into concrete, testable contracts. This approach facilitates early feedback, automated compliance testing, and a higher degree of confidence in the system's ability to meet its requirements.
