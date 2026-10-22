# Corpus analysis: what makes a Marvik post work

Traits extracted from the 10 posts in `articles/` (Feb–Jun 2026). This is the basis for `blog-template.md`; every decision in the template points back to a finding here.

## 1. Archetypes

The posts fall into four types. The type determines which sections of the template apply.

| Archetype | Posts | Narrative engine |
| --- | --- | --- |
| **Engineering story** | Unitree G1, Genie vs Mosaic, Spec-Driven Development | A real problem and the decisions made to solve it |
| **Realities / lessons** | Securing LLM Applications, Simplicity vs Control | N numbered claims, each with its own case or metric |
| **Technology evaluation** | Claude Managed Agents, S3 Tables Future, IoT Architecture, AI in Healthcare | What it is, how it works, where it fails, who it's for |
| **Tutorial** | S3 Tables Quick Setup | Steps with code; ends when the reader has something working |

The most engaging posts are the engineering stories and the realities: they have a tension (something turned out differently than expected) that carries the reading. The weakest evaluations (Healthcare, IoT) read like a catalog because they lack that tension.

## 2. Hooks

**The ones that work** open with something concrete and promise a result within the first three paragraphs:

- **Scene:** "I stood next to a Unitree G1 EDU+ in our lab, said 'jarvis' out loud, and waited." (Unitree)
- **Recurring business question:** "Every few weeks, someone on the business side would ask the same thing: 'Can I just type my question and get the data?'" (Genie)
- **Risk + number:** "one prompt injection is all it takes…", followed by "reduced our ASR from 73 and 66% down to 3 and 0%." (Security)
- **Counterintuitive:** "SQL generation is the easy part." (Genie); "an audio pipeline that took longer to ship than the LLM." (Unitree)

**The ones that don't work** open with generalities that could head any post:

- "The convergence of artificial intelligence and healthcare is no longer a promise…" (Healthcare)
- "Recently introduced by AWS, S3 Tables provides…" (S3 Tables Future)
- "In this post, we provide a simple code example…" (S3 Quick Setup)

**Pattern of a good opening (3 moves):** something concrete → why it's harder than it looks ("In a demo, it looks almost magical. In a real governed environment… it's a different story.") → what the reader will take away ("This post is the engineering story of…").

**Subtitle (dek):** Claude Managed Agents, Unitree and SDD put a line under the title that summarizes the content ("A two-tier reasoning brain on OpenClaw, an on-prem Ollama stack, and an audio pipeline that took longer to ship than the LLM."). It works as a second hook and as the meta description.

## 3. Body structure

- **Section headings as claims, not labels.** "The first realization: SQL generation was not enough", "Decision 2: audio, the real bottleneck", "LLM Defense is not a model tweak". Compare with "Technology overview" or "S3 Tables Main Features": the first kind tells the story just by reading the headings.
- **Numbered decision blocks** (Unitree: Decision 1/2/3; Security: 5 realities). They give rhythm and an implicit promise of progress.
- **Shared foundation before comparing.** Genie spends two sections on "the shared foundation" before comparing tools; that makes the comparison fair and the reader understands why the data matters more than the tool.
- **Transitions by consequence:** each section ends with the question that opens the next one ("That brought us to a real decision: use a managed Databricks experience, or build the agent ourselves?").

## 4. Brand traits (present in almost all posts)

1. **Explicit limits.** "Where Genie showed its limits", "MCP Integration: What Works and Where the Gaps Are", "S3 Tables Limitations", "The Opacity Trade-off", "Trade-offs and current limitations", "Why 3% Still Matters". Marvik earns credibility by showing where the tool fails, even the one it recommends.
2. **Decision guide.** "When is S3 Tables the right choice? / When may it not be?", "Who is Claude Managed Agents really for?", "Start with Genie when… Move to Mosaic when…", "Use Cases: Choosing the Right Approach". The reader leaves with a criterion, not just information.
3. **Lessons in the imperative.** "Pick the LLM by stress testing, not vibes." "The robot's mic was the bottleneck, not the model." Each lesson is a claim + one or two lines of concrete justification.
4. **First-person team voice** ("we built", "we learned"; "I" in SDD). Never an impersonal corporate voice in the good posts.
5. **Closing with a Marvik paragraph + CTA.** "At Marvik, we build end-to-end AI products…", "we'd love to talk", "Explore more articles on Marvik's blogs".

## 5. Evidence: the corpus's weakest point

- Only Security shows a before/after metric (ASR 73% → 3%). It is also the strongest hook in the corpus.
- Genie, the most complete engineering story, does not have a single measurable number in the text.
- Simplicity vs Control compares 2025 vs 2026 solutions without quantifying cost or quality in most cases.

**Opportunity:** require at least one measured result (latency, cost, error rate, development time) per post. It is what would most set a new post apart from the rest.

> Note: the scraping did not keep tables or images (0 tables across the 10 files), so the use of diagrams or comparison tables can't be evaluated. The originals likely had them.

## 6. Problems to avoid

- **Repetition at the top and bottom:** Security opens with "Key Takeaways" and closes with a conclusion that repeats the same thing. If there is a TL;DR, the conclusion must add something else (the decision guide or the next question).
- **Generic conclusions:** "Generative AI in healthcare is no longer experimental." It could close any post in the sector.
- **Rhetorical closing questions** ("Final Thought: If you ran a jailbreak scan today, would you be prepared…?", "Strategic Reflection: Is your organization ready…?"). They compete with the CTA.
- **Formulas flagged by `/stop-slop`** found in the corpus: "Not theory, but a practical framework", "This isn't to say…", "The harsh truth about…", "It's not about X, but about Y". They are marks of generated text that weaken otherwise solid posts.
- **Catalog without tension:** listing products or features (Healthcare: three families of solutions described one by one) without a problem that orders them.
- **Tutorial without an ending:** S3 Quick Setup ends with "Now you can explore your data using Athena!" with no summary, resource cleanup or next step.

## 7. Length

1,070 to 3,275 words. The engineering stories and realities, which are the best, sit between 1,600 and 2,200. Above ~2,500 (S3 Tables Future, Simplicity) the text starts to deliver less per word.
