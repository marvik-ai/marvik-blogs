# Marvik blog post template

Reusable structure derived from `corpus-analysis.md`. The post is written in English, like the rest of the blog.

**How to use it:**

1. Fill in the **brief** (part A). If a field marked as required is missing, the post isn't ready to be written yet.
2. Pick the **archetype** and check which sections apply (part B).
3. Write following the **skeleton** (part C). The `{{placeholders}}` get replaced; the `<!-- -->` comments get deleted.
4. Review with the **checklist** (part D).

---

## A. Brief (fill in before writing)

| Field | Required | Content |
| --- | --- | --- |
| One-sentence thesis | Yes | {{What we learned that the reader doesn't know. It must be arguable: "SQL generation is the easy part" yes; "AI is transforming X" no.}} |
| Reader | Yes | {{Role and level: ML engineer, data engineer, CTO, business leader. Determines how much gets explained.}} |
| The unexpected | Yes | {{What turned out differently than planned. It's the tension that carries the post. E.g.: "the microphone was the bottleneck, not the model".}} |
| Measured result | Yes | {{At least one before/after metric: latency, cost, error rate, development time. It's the corpus's weakest point.}} |
| Key decisions | Yes | {{2 to 4 decisions with: options considered, the one chosen, why.}} |
| Limits | Yes | {{Where the solution or tool fails, even the recommended one.}} |
| Who it's for / not for | Yes | {{Decision criterion the reader takes away.}} |
| Marvik's role | Yes | {{What the team did and what Marvik offers based on this. Feeds the closing.}} |
| Artifacts | No | {{Architecture diagram, code snippets, comparison table, screenshots.}} |
| Related post | No | {{Previous or next post in a series, to link.}} |
| Authors | Yes | {{Names; format "First Last; First Last".}} |

---

## B. Archetypes and sections

| Section | Engineering story | Realities / lessons | Technology evaluation | Tutorial |
| --- | --- | --- | --- | --- |
| 1. Title and subtitle | ✓ | ✓ | ✓ | ✓ |
| 2. Hook | ✓ | ✓ | ✓ | ✓ (short) |
| 3. TL;DR | — | If over ~2,000 words | If over ~2,000 words | — |
| 4. Context: why it's hard | ✓ | ✓ | ✓ | Prerequisites |
| 5. Starting point | ✓ | Optional | How it works | Setup |
| 6. Core blocks | Decisions | Numbered realities | Capabilities evaluated | Steps |
| 7. Results | ✓ | Per block | Own tests if any | Verification |
| 8. Limits and trade-offs | ✓ | ✓ | ✓ | Common problems |
| 9. Lessons | ✓ | — (the blocks are already lessons) | Optional | — |
| 10. Decision guide | Optional | ✓ | ✓ | — |
| 11. Closing | ✓ | ✓ | ✓ | Next step + cleanup |
| 12. Sources | ✓ | ✓ | ✓ | ✓ |

Target length: 1,600–2,200 words (the range of the best posts in the corpus). Tutorial: whatever the code requires.

---

## C. Skeleton

```markdown
# {{Title}}

- Date: {{Month Dayth, YYYY}}
- Category: Technical
- Authors: {{Author 1; Author 2}}
- URL: https://www.marvik.ai/blog/{{slug}}

---

{{Subtitle: one line summarizing the content with concrete details.}}
<!-- E.g.: "A two-tier reasoning brain on OpenClaw, an on-prem Ollama stack, and an audio
pipeline that took longer to ship than the LLM." It also serves as the meta description. -->

{{Hook, paragraph 1: something concrete.}}
<!-- Pick one form: scene ("I stood next to the robot and said 'jarvis'"), a stakeholder's
recurring question, risk with a number, or a counterintuitive claim.
Never open with "In today's…", "Recently introduced by…", "The convergence of…". -->

{{Hook, paragraph 2: why it's harder than it looks.}}
<!-- The demo vs production contrast is the most effective device in the corpus. -->

{{Hook, paragraph 3: the promise.}}
<!-- What the reader takes away, ideally with the measured result:
"…and how combining them reduced our ASR from 73% to 3%." -->

## Key takeaways
<!-- Only if the post is over ~2,000 words. 3 to 5 bullets. The conclusion must NOT
repeat them: it must add the decision guide or the next question. -->
- {{takeaway}}

## {{Context as a claim: why the problem is hard}}
<!-- Define the concept only if the brief's reader doesn't know it, in one paragraph.
End with the question or decision that opens the next section. -->
{{content}}

## {{Starting point: architecture, shared foundation or constraints}}
<!-- What existed before starting and what limited the options (data, security,
hardware, budget). In comparisons, the common base that makes the comparison fair.
Natural place for the architecture diagram. -->
{{content}}

## {{Decision / Reality 1: heading as a claim}}
<!-- Repeat this block 2 to 4 times. Internal structure:
1. Situation: what concrete problem came up.
2. Options: what we considered.
3. Choice and why.
4. Evidence: number, code snippet, test.
5. Transition: what new problem this decision left → opens the next block.
Good heading: "Decision 2: audio, the real bottleneck". Bad: "Audio pipeline". -->
{{content}}

## {{Results}}
<!-- Before/after with numbers. If there are more than two metrics, use a table.
If they were already shown in a block, consolidate them here, don't repeat them. -->
{{content}}

## {{Limits: where it fails}}
<!-- Marvik brand trait. What we didn't solve, where the tool falls short,
what the chosen option costs (opacity, lock-in, price). -->
{{content}}

## What we learned
<!-- 3 to 6 lessons in the imperative, each with 1-2 lines of concrete justification.
E.g.: "Pick the LLM by stress testing, not vibes. Targeted tests on planning, strict JSON
and tool summarization shaped the two-tier design." -->
- **{{Lesson}}.** {{justification}}

## {{Decision guide: when to use each option}}
<!-- Format "Use X when… / Move to Y when… / Avoid when…". It's what the reader applies
to their own case. -->
{{content}}

## {{Closing}}
<!-- 3 short paragraphs at most:
1. The thesis restated in light of what was told (not a summary of sections).
2. Marvik paragraph: what we do in this space, tied to the role in the brief.
3. Concrete CTA: contact or related post.
Avoid rhetorical closing questions: they compete with the CTA. -->
{{content}}

## Sources
- {{source}}
```

---

## D. Review checklist

**Structure**
- [ ] Reading only the section headings, the story is clear.
- [ ] The first three paragraphs have something concrete, the difficulty and the promise.
- [ ] There is at least one measured result.
- [ ] There is a limits section.
- [ ] Each section ends by opening the next one.
- [ ] The conclusion doesn't repeat the TL;DR or summarize the sections.

**Writing** (with the skills in `.claude/skills/`)
- [ ] `/copy-editing`, only the clarity, voice, proof and specificity passes. The emotion and zero-risk passes are meant for conversion and don't apply.
- [ ] `/stop-slop` as a guide, not a fixed rule: remove formulas ("Not X, but Y", "The harsh truth about", "Here's why"), filler phrases and empty adverbs. Tolerate passive voice in technical descriptions ("the table is partitioned by…") and the occasional em dash (—) if the text calls for it.

**Format**
- [ ] Metadata header complete and row added to `index.csv`.
- [ ] Saved as `articles/<slug>.md` (file name = URL slug).
