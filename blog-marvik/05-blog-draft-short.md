---
title: "Put a gate before the judge: cheap decision models in front of expensive AI, in a marketplace catalog and an anti-piracy pipeline"
category: Business Impact
author: Marvik
status: DRAFT v3 short (2026-10-23). Shortened version of 05-blog-draft.md (~2,300 words). Not for publication.
pending:
  - Both clients stay unnamed. Naming the media group needs its approval.
  - Media group figures come from a confidential strategy document and need client approval - subscribers (1M to 600k), 1M+ pirate IPTV homes, one-third price, Argentina blocking (+31,000 subscriptions, +67% gross sign-ups), USD 1.6M a month, legal status of real-time blocking, two-person team, 80% precision target, takedown anecdote.
  - "Seven times as many pairs" is our own calculation (USD 9.76 / 1.38 per 1,000 pairs).
  - Catalog quarterly volume (10M pairs) needs confirmation. Derived figures use the 10k README per 1,000 pairs (USD 9.76 vs 1.38; 249 vs 35 minutes; 12.3% pass) - USD 97,600 vs 13,800, 41,500 vs 5,800 hours, 8.77M fewer calls.
  - Production status of both systems: earlier notes describe a PoC (catalog) and staging with enforce by default (anti-piracy). Confirm with both clients before saying "on its way to production".
---

# Put a gate before the judge: cheap decision models in front of expensive AI, in a marketplace catalog and an anti-piracy pipeline

*How Marvik helped a leading Latin American e-commerce marketplace and a global media group put cheap decision models in front of their most expensive AI steps: 86% less cost and compute time in duplicate detection, and a piracy filter that is more accurate and nearly four times faster than the one it replaced.*

## Most of what an AI pipeline reviews is a "no"

Many AI systems in production share the same pipeline. Something upstream proposes candidates: two product pages that might be the same item, or a post that might be selling pirated TV. At the end, an expensive LLM agent reads each candidate and makes the call. In between, something has to decide what reaches the agent.

The usual answer is rules, such as regular expressions over titles or shared identifiers. They are cheap, but they cannot read context, so they miss many true positives. The alternative is to send everything to the agent, which then spends most of its time saying no: **97 of every 100 product pairs were not duplicates**, and **about 9 of every 10 Instagram reels or Facebook Marketplace listings** opened while searching for piracy were not pirate offers. In both cases the stakes were commercial: a duplicate left in the catalog costs sales, and a pirate offer left online keeps selling the client's football for a fraction of the price.

## Why decision models were a good fit

We needed a middle ground between rigid regex rules and a full LLM verdict: flexible, context-aware rules with auditable decisions. Instead of asking a cheap model for a verdict, we asked it a few closed questions about each candidate and kept the probabilities.

**Decision models**, such as OpenAI's Decisions API or JEV from TypeSafe, answer a question by choosing among a fixed set of options and return a probability for each one. That makes them a good fit for a gate:

- **They are cheap and fast.** The output is not billed, so the cost depends only on what the model reads: a fraction of a cent per item, usually in under half a second.
- **A probability works like a dial.** The business sets the threshold to the error it can tolerate: a missed duplicate in one case, an analyst's wasted time in the other.
- **The policy lives in the question.** What counts as "the same product" or "a pirate offer" is written in the question's criteria and versioned. Every decision is logged with its probabilities, so anyone can audit why a candidate was dropped.

The gate never makes the final call. It only decides which candidates reach the agent.

## Case 1: duplicate detection in an e-commerce catalog

### Why it matters: a duplicated catalog sells less

In a large marketplace, every seller can create their own page, so the same product gets listed again and again. Buyers searching for a pair of headphones find the same model five times, with its reviews spread across pages, and cannot compare prices in one place. Sellers see their sales history and search ranking diluted. The marketplace pays twice: in conversion, because a fragmented catalog is harder to buy from, and in trust, because a catalog full of near-copies looks unreliable.

And the more the catalog grows, the more pairs need a decision. Keeping it clean cannot depend on a review bill that grows just as fast.

### The challenge: an accurate judge that spends most of its time saying no

Rules such as shared identifiers propose candidate pairs, but they cannot settle the question: two pages with different titles can be the same product, and two nearly identical ones can differ in color or pack size. So an LLM agent judges each pair by reading both pages and their photos, as a human analyst would.

The agent is accurate, but slow and expensive. Measured with its own prompt on real pairs, each decision takes **about 15 seconds and costs about one US cent**, 77% of it in reasoning-heavy output tokens. At roughly 10 million pairs per quarter, that is around **USD 98,000 and 41,500 compute hours a quarter**, nearly all of it spent confirming that two pages are *not* the same product.

The gate had to let through **at least 95% of true duplicates**, because the agent cannot merge what it never sees, and then filter out as many non-duplicates as possible, leaving every merge decision to the agent. Replacing the agent with a cheaper model had already been tested, and it caught only 40% to 57% of duplicates. A cheap model was not a good judge. But it could be a good gate.

### How it works: three ways to ask the question

We tested three families of gates, each with and without the product photo:

1. **A single question**, a simplified version of the agent's own prompt.
2. **A hand-written question bank with a trained decider.** Closed questions about each way two pages can differ (variant, sale unit, or a product against its accessory), combined into one score by a small trained model.
3. **An LLM in a research loop**, which generates, rewrites and prunes questions while a trained decider scores each set and feeds the results back.

The client's merge policy went into the wording. A different barcode is seller noise, placeholder brands like "Generic" are neutral, and "Preto", "Negro" and "Black" are the same color.

There was no single winner: each option led on one axis. The single question with a photo was the cheapest, but kept only 93.7% of duplicates, just under the target. A bank of four broad questions filtered the most while keeping 95%. A bank tuned for 97% missed the fewest duplicates, but sent about a fifth more pairs to the agent. Choosing among them is a business decision: fewer missed duplicates or more savings. The results below use the four-question bank.

### How we validated it

The rule was **decide everything first, then test once.** Questions and thresholds were set on labeled development data. The blind test used a separate week of real traffic, at the real prevalence of 3% duplicates, and every filter ran once, unchanged. It paid for itself: **one in four filters that met the target in development fell below it on new data.** We also measured the agent's cost instead of estimating it, and it turned out to be twice the previous estimate.

### Results

On the blind test, the four-question bank with photos delivered:

- **95.0% of duplicates still reach the agent**, and **90.3% of non-duplicates are filtered out.** Only **12.3% of pairs** move on.
- **Cost per 1,000 pairs falls from USD 9.76 to USD 1.38, and compute time from 249 to 35 minutes: 86% less on both.**
- **At quarterly volume: about USD 14,000 instead of USD 98,000, about 5,800 compute hours instead of 41,500, and 8.8 million fewer calls to the agent.**
- **The same budget now covers about seven times as many candidate pairs**, so catalog cleanup can keep pace with catalog growth.

## Case 2: finding piracy offers for a media group

### Why it matters: every pirate subscription is a customer the client already lost

For a media group with live football rights in Latin America, piracy is a subscriber problem. In Chile, its sports channel went from about 1 million subscribers to about 600,000 in five years, and the company believes most of that gap went to piracy. More than a million Chilean homes are estimated to use illegal IPTV services, which sell hundreds of channels, football included, for about a third of the price of a legal subscription.

Those homes are not lost for good. Pirate IPTV users already pay for TV every month, and when they lose access, many come back. When IPTV services were blocked in Argentina in February 2026, the client's streaming service gained **31,000 incremental subscriptions** and a **67% jump in gross sign-ups**. By the client's own estimate, winning back 10% of Chile's pirate IPTV homes would be worth about **USD 1.6 million a month**.

Blocking domains in real time is not legally available in Chile or Argentina. The weak spot is the storefront: pirate services find customers on Instagram and Facebook Marketplace, and close each sale through WhatsApp. Every offer found and reported early cuts one sales channel for a pirate operator.

### The challenge: a two-person team and filters that could not tell an offer from noise

Before this project, offers were found by hand and sent to an anti-piracy team of **two people**. Marvik built a monitor with the same pipeline as the catalog. Browser agents search Instagram and Facebook Marketplace in Argentina and Chile, and a first filter decides which candidates become cases. Each case then goes to an **analysis agent**, a generative LLM that reads the post and its screenshot, decides whether it is a pirate offer and extracts its details, before analysts review it. The client's target: **at least 80% of flagged cases confirmed as piracy**, because every false alarm costs the team time.

The weak point was the first filter on each surface:

- **On Facebook Marketplace, keyword rules on the title decided which listings to open.** In a test, they recovered **only about 3% of the pirate offers** among the listings they had set aside.
- **On Instagram, a generative LLM gate answered yes or no for each post.** It was reasonably accurate but slow (3.2 seconds per post), and it **failed open**: when a call errored, the post went through. In one September week, **almost 9 of every 10 Instagram cases** that reached the analysis agent were not piracy.

Saying no well is harder than it looks. In Argentina, "TV digital" is the street name for pirate IPTV and also the name of legal free-to-air TV. And the offer is often only in the image: about three in four pirate posts missed by a text-only model had the offer in the flyer, not the caption.

### How it works

The same component took two different places:

1. **Instagram: one question and the screenshot.** The decision model replaces the LLM gate. It sees the masked caption and the screenshot, and a second question checks that the post is in Spanish, a requirement the client set for its market.
2. **Facebook Marketplace: a two-stage cascade.** The keyword rules are gone. **Stage 1 reads only the text and can only reject** clear negatives, because some offers only show in the image. **Stage 2 takes the screenshot**, the expensive part (loading the page, three times the tokens, more exposure to anti-bot controls), and decides. That screenshot becomes the case's evidence.

The client's enforcement policy went into the wording. *Yes* means selling a service, a channel list, an app, premium accounts, or a TV box shipped with pirate apps. *No* means hardware alone, an identified internet provider selling its own plan, or news about a blocked app. The TV box rule came from the data: in an early test, about two thirds of the false alarms were boxes the human labeler had marked as not pirate, while the analysis agent had confirmed every such box as piracy. The client settled it, and the question says so.

### Results

On Instagram, we replayed real posts with their screenshots and compared the new filter with the old gate, using the analysis agent's verdicts as the reference:

- **It catches more pirate offers and raises fewer false alarms.** It finds 94% of the pirate offers, up from 88%, and 93% of what it flags is real piracy, up from 91%, well above the client's 80% target.
- **Median time per post fell from 3.2 to 0.85 seconds**, at about **USD 0.29 per 1,000 posts.**
- **Requiring Spanish means catching 82% of pirate offers instead of 94%**, with the same share of real piracy among flagged posts. The client accepted that trade-off: the lost offers are outside its market.

On Facebook Marketplace, the first live run in Buenos Aires and Santiago showed where the cascade pays:

- **The text stage discarded 87% of listings without taking a screenshot**, at a median of 425 milliseconds per listing.
- **The savings are in screenshots, not tokens**: about one page capture for every eight listings, for a model cost of about USD 0.14 per 1,000 listings.
- These listings have no human labels yet, so this run does not yet measure how many offers were caught or missed.

## What these projects show: learn to say no

1. **Don't replace the judge; put a gate in front of it.** In both pipelines, an LLM agent still makes the final call. The gate decides what reaches it: 1 in 8 pairs in the catalog, and only the likely offers in anti-piracy.
2. **Write the policy into a question that returns a probability.** Barcodes, TV boxes and "TV digital" were rules and words someone had to write down, and the threshold is where each client chose which error to live with.
3. **Decide what the gate does when it fails.** A gate that says yes when in doubt can turn one provider error into a week of junk cases for the people downstream.
4. **Measure the expensive step, and the labels you measure against.** The catalog agent cost twice the estimate; in anti-piracy the expensive part was screenshots. And about 8% of the catalog's duplicate labels looked wrong on review.

## Where they stand and what's next

Both systems are on their way to production. The catalog gate is moving to live traffic next to the current agent, and the anti-piracy filters already decide which posts and listings become cases, so the client's team spends its time on offers that are likely to be real.

The projects also opened new challenges where we are now studying the same framework, above all real-time decisions where an LLM cannot answer in time: fraud checks at checkout in an e-commerce marketplace, or filtering problematic products out of search results before they are shown.

**Expensive models should spend their time where judgment is needed. A good gate makes sure they do.**
