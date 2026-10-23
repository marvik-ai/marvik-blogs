---
title: "Put a gate before the judge: cutting LLM duplicate-detection costs by 86% in a marketplace catalog"
category: Business Impact
author: Marvik
status: DRAFT v1 (2026-10-22). Not for publication: client name, internal figures and model names pending approval.
---

# Put a gate before the judge: cutting LLM duplicate-detection costs by 86% in a marketplace catalog

*How Marvik helped [Mercado Libre]'s catalog team put a cheap decision filter in front of its duplicate-detection agent, keeping 95% of true duplicates while sending only 1 in 8 candidate pairs to the expensive model.*

## Catalog quality is a pairs problem

In a large marketplace, the same physical product is listed again and again. Different sellers create their own product pages, with their own titles, attributes and photos, and over time one pair of headphones or one bottle of shampoo ends up spread across several pages. Each copy splits what should be together: reviews, sales history, search ranking and the buyer's ability to compare offers in one place.

Finding those duplicates comes down to comparing pairs. A candidate generator proposes pairs of pages that might be the same product, using rules such as shared identifiers or transitivity. Then a judge decides, pair by pair, whether the two pages should become one. Today that judge is an LLM agent that reads both pages and looks at their photos, the way a human catalog analyst would.

The agent is accurate. The problem is everything it has to look at to find what matters.

## The challenge: an accurate judge that spends most of its time saying no

In real traffic, **only about 3 in every 100 candidate pairs are actual duplicates.** The other 97 still go through the full analysis.

That analysis is expensive because the agent runs on a reasoning model. Measured with its own prompt on real pairs, each decision takes **about 15 seconds and costs about one US cent.** Most of that cost is not the answer: 77% comes from output tokens, and almost all of them are reasoning. At the client's volume, roughly 8.6 million candidate pairs per quarter, that adds up to around **USD 84,000 and 36,000 compute hours a quarter**, nearly all of it spent confirming that two pages are *not* the same product.

The requirements were clear from the start:

- **Recall comes first.** A false positive costs one extra call to the agent; a missed duplicate never reaches it, and the agent cannot merge what it never sees. The target was to keep **at least 95% of true duplicates.**
- **Then filter as much as possible.** Every non-duplicate removed before the agent is time and money saved.
- **The agent stays in charge.** Merge decisions keep following the existing process.

The obvious idea, replacing the agent with a cheaper model, had already been tested and ruled out. In earlier rounds, a small model fell 5 F1 points short of the agent on a benchmark, and the agent's own prompt on a cheaper model became overly cautious, catching only 40% to 57% of duplicates. A cheap model was not a good judge. But it could be a good gate.

## The approach: typed questions, probabilities and a threshold

Instead of asking a cheap model for a verdict, we asked it a few closed questions about each pair and kept the probabilities.

We used **decision models**: services such as OpenAI's Decisions API, or Jev, that answer a question by choosing among a fixed set of options and return a probability for each one. Two properties make them a good fit for a gate. First, the output is not billed, so the cost depends only on what the model reads: about **USD 0.0002 and a quarter of a second per pair.** Second, a probability works like a dial. Instead of a yes/no decision, the team can set the threshold to the recall the business needs.

A typical question bank had four questions, all asked in a single call with the titles, attributes and the first photo of each page:

1. Are A and B the same purchasable product, with the same variant and the same sale unit?
2. Do they state an explicit difference in a variant attribute, such as color, size, capacity or voltage?
3. Do they sell a different sale unit, such as a single item against a pack?
4. Are they different objects, such as a product and an accessory or refill for it?

The client's merge policy went into the wording. A different barcode is seller noise and never separates two pages, while a shared one supports a match but can be copied. Placeholder brands like "Generic" are neutral, and "Preto", "Negro" and "Black" are the same color. A learned combiner (logistic regression or gradient-boosted trees) turned the answers into a single score.

We compared three families of filters, each with and without photos:

- **A single question**, a simplified version of the agent's own prompt.
- **A hand-written question bank plus a combiner**, from four broad questions to fifteen narrow yes/no checks.
- **Autoresearch:** an LLM loop that proposed new questions, tested them on development data and kept only the ones that improved the filter.

In total, 33 filters reached the final test.

*[Figure 1: the cascade. Candidate pairs → decision gate (0.25 s, USD 0.0002 per pair) → duplicate-detection agent (15 s, USD 0.01 per pair) → merge. Out of 1,000 candidate pairs, 123 reach the agent, including 95% of the duplicates.]*

## How we validated it

A filter that only looks good on the data it was tuned on is not useful, so the evaluation followed a simple rule: **decide everything first, then test once.**

- **Development:** 2,000 labeled pairs. Questions, combiners and thresholds were set here, with the threshold fixed to let 95% of duplicates through under cross-validation grouped by product.
- **Blind test:** 10,000 new candidate pairs from one week of real traffic, with **300 duplicates (3%, the real prevalence)** across 901 product domains and no product shared with development. Every filter ran once, exactly as it was. Nothing was adjusted after seeing the results.
- **The expensive step was measured, not estimated.** We ran the agent's prompt on real pairs to get its true cost and latency. The measured cost was twice the previous estimate, because the earlier formula did not account for reasoning tokens.

The blind test justified the effort: **8 of the 33 filters that reached 95% recall in development fell below it on new data.** Without the test, one of them could have gone to production.

## Results

The best option was a **four-question bank, with photos downscaled to 512 pixels and a logistic-regression combiner.** On the 10,000 unseen pairs:

- **95.0% of duplicates still reach the agent** (285 of 300), and **90.3% of non-duplicates are filtered out.** Only **12.3% of candidate pairs** move on.
- **Cost per 1,000 pairs falls from USD 9.76 to USD 1.38: 86% less.** The filter itself costs USD 0.18 per 1,000 pairs.
- **Compute time per 1,000 pairs falls from 249 minutes to 35 minutes: 86% less.**
- **At quarterly volume, that means about USD 12,000 instead of USD 84,000, 5,000 compute hours instead of 36,000, and 7.5 million fewer calls to the agent.**
- **For more margin**, a similar four-question bank with its threshold tuned for 97% kept **97.3% of duplicates** while still filtering 88% of non-duplicates.

*[Figure 2: duplicates that reach the agent vs. non-duplicates filtered out, one point per filter, with the 95% recall line marked.]*

Three findings shaped the recommendation:

- **Photos pay for themselves.** In every filter family, adding a photo downscaled to 512 pixels filtered out between 3 and 10 more points of non-duplicates at the same recall target. For the single-prompt filter, full-size photos cost twice the tokens and filtered less than the downscaled ones.
- **A few broad questions beat many narrow ones.** Four questions filtered more than banks of twelve to fifteen specific checks, at lower cost.
- **The value of the gate depends on the price of the judge.** If the agent ran on a cheaper model without reasoning, the same gate would save about half the cost instead of 86%. The filter costs cents; what moves the bill is how many pairs it lets through.

The full evaluation, covering 33 filters on 10,000 pairs, cost about USD 37 in API calls and took under an hour to run.

## What this project shows

1. **Don't replace the judge; put a gate in front of it.** The agent was too slow and expensive for every pair, and the cheap models were not good enough to decide alone. Together, the system keeps the agent's quality and pays for it only where it matters.
2. **Ask for probabilities, not verdicts.** A probability turns a model into a dial that the business can set to its tolerance for missed duplicates.
3. **Measure the expensive step.** The agent's real cost was double the estimate. Every saving depends on that number.
4. **Small samples flatter; blind tests at real prevalence don't.** A quarter of the filters that passed in development failed on fresh data.
5. **Labels are part of the system.** In an earlier round, an LLM review suggested that about 8% of the pairs labeled as duplicates were labeling errors. Some of the duplicates "missed" by the gate may not be duplicates at all.

## Where it stands and what's next

This is a validated proof of concept, not yet a production system. The next steps are a **shadow run on live traffic** next to the current agent, a manual review of the 15 duplicates the gate filtered out, a run of the same question banks on Jev with photos (about 2.4 times cheaper per token), and a check of the projected savings against the agent's real bill.

The pattern applies well beyond catalogs. Wherever an expensive model spends most of its time on easy negatives, as in fraud review, content moderation, document matching or lead qualification, a cheap, tunable gate can carry most of the load.

**Expensive models should spend their time where judgment is needed. A good gate makes sure they do.**
