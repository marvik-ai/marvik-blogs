---
title: "Put a gate before the judge: cheap decision models in front of expensive AI, in a marketplace catalog and an anti-piracy pipeline"
category: Business Impact
author: Marvik
status: DRAFT v3 (2026-10-23). Builds on 03-blog-draft.md and adds the anti-piracy case. Not for publication.
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

Many AI systems in production share the same pipeline. Something upstream proposes candidates: two product pages that might be the same item, or a post that might be selling pirated TV. At the end, an expensive LLM agent reads each candidate and makes the call: are these the same product, is this a pirate offer? In between, something has to decide what reaches the agent.

The usual answer is rules, such as regular expressions over titles or shared identifiers. They are cheap, but they cannot read context. Whether two pages with different titles are the same product, or whether an ad for a TV box is selling hardware or a pirate subscription, depends on reading the whole listing, and rules that cannot do that miss many true positives. The alternative is to send everything to the agent, which then spends most of its time saying no: in the two projects in this post, **97 of every 100 product pairs were not duplicates**, and **about 9 of every 10 Instagram reels or Facebook Marketplace listings** opened while searching for piracy were not pirate offers.

The marketplace sent nearly every candidate to its agent and paid for it. The media group's first filters let real offers slip away and noise reach the agent. In both cases the stakes were commercial: a duplicate left in the catalog costs sales, and a pirate offer left online keeps selling the client's football for a fraction of the price. Both got the same answer: a cheap decision model placed in front of the agent.

## Why decision models were a good fit

We needed a middle ground between rigid regex rules and a full LLM verdict: flexible, context-aware rules with auditable decisions. Instead of asking a cheap model for a verdict, we asked it a few closed questions about each candidate and kept the probabilities.

**Decision models**, such as OpenAI's Decisions API or JEV from TypeSafe, answer a question by choosing among a fixed set of options and return a probability for each one. That makes them a good fit for a gate:

- **They are cheap and fast.** The output is not billed, so the cost depends only on what the model reads: a fraction of a cent per item, usually in under half a second.
- **A probability works like a dial.** The business sets the threshold to the error it can tolerate: a missed duplicate in one case, an analyst's wasted time in the other.
- **The policy lives in the question.** What counts as "the same product" or "a pirate offer" is written in the question's criteria, versioned and tested on labeled data before it changes. Every decision is logged with its probabilities and the question version, so anyone can audit why a candidate was dropped.

In both projects the decision model is a bounded component. It never makes the final call: the LLM agent still decides whether two pages are the same product or whether a post is a pirate offer, and the existing processes still handle merges and takedown requests. The gate only decides which candidates reach the agent.

## Case 1: duplicate detection in an e-commerce catalog

### Why it matters: a duplicated catalog sells less

In a large marketplace, the same physical product gets listed again and again, because every seller can create their own page with their own title, attributes and photos. Each copy splits what should be together. Buyers searching for a pair of headphones find the same model five times, with its reviews spread across pages, and cannot compare prices in one place. Sellers see their sales history and search ranking diluted. The marketplace pays twice: in conversion, because a fragmented catalog is harder to buy from, and in trust, because a catalog full of near-copies looks unreliable.

The problem also grows with the business. Every new listing can duplicate an existing one, so the more the catalog grows, the more candidate pairs need a decision. Keeping it clean cannot depend on a review bill that grows just as fast.

### The challenge: an accurate judge that spends most of its time saying no

A candidate generator proposes pairs of pages that might be the same product, using rules such as shared identifiers. Rules cannot settle the question: two pages with different titles can be the same product, and two nearly identical ones can differ in color or pack size. So an LLM agent judges each pair by reading both pages and looking at their photos, the way a human catalog analyst would.

The agent is accurate, but it runs on a generative model, which is time-consuming. Measured with its own prompt on real pairs, each decision takes **about 15 seconds and costs about one US cent**, 77% of it in reasoning-heavy output tokens. At roughly 10 million pairs per quarter, that is around **USD 98,000 and 41,500 compute hours a quarter**, nearly all of it spent confirming that two pages are *not* the same product.

The requirements were clear:

- **Catching duplicates comes first.** The agent cannot merge what it never sees, so the gate had to let through **at least 95% of true duplicates.**
- **Then filter as much as possible.** Every non-duplicate removed is time and money saved.
- **The agent stays in charge** of every merge decision.

Replacing the agent with a cheaper model had already been tested: a small model was clearly less accurate than the agent, and the agent's own prompt on a cheaper model caught only 40% to 57% of duplicates. A cheap model was not a good judge. But it could be a good gate.

### How it works: three ways to ask the question

We tested three families of gates, each with and without the product photo:

1. **A single question.** A simplified version of the agent's own prompt, asked to a decision model: are these two pages the same product?
2. **A hand-written question bank with a trained decider.** Several closed questions, each targeting one way two pages can differ: the variant (color, size, voltage), the sale unit (a single item against a pack) or the object itself (a product against its accessory). A small trained model combines the answers into one score.
3. **An LLM in a research loop.** An LLM generates, rewrites and prunes questions that work as features. A trained decider evaluates each set on development data and feeds the results back to the LLM, which adjusts the questions for the next round.

The client's merge policy went into the wording of every question. A different barcode is seller noise, while a shared one supports a match but can be copied. Placeholder brands like "Generic" are neutral, and "Preto", "Negro" and "Black" are the same color.

There was no single winner: each option led on one axis. The single question with a photo was the cheapest, but kept only 93.7% of duplicates, just under the target. A bank of four broad questions, asked in one call with the titles, attributes and first photo of each page, filtered the most while keeping 95%. A bank tuned for 97% missed the fewest duplicates, but sent about a fifth more pairs to the agent. Longer banks of narrow checks lost on all three. Choosing among the rest is a business decision: fewer missed duplicates or more savings. The results below use the four-question bank.

### How we validated it

The rule was **decide everything first, then test once.** Questions and thresholds were set on a development set of labeled pairs. The blind test used a separate week of real traffic, at the real prevalence of 3% duplicates, with no product shared with development. Every filter ran once, unchanged. We also measured the agent's cost instead of estimating it: it was twice the previous estimate.

The blind test paid for itself: **one in four filters that met the target in development fell below it on new data.**

### Results

On the blind test, the four-question bank with photos downscaled to 512 pixels delivered:

- **95.0% of duplicates still reach the agent**, and **90.3% of non-duplicates are filtered out.** Only **12.3% of pairs** move on.
- **Cost per 1,000 pairs falls from USD 9.76 to USD 1.38, and compute time from 249 to 35 minutes: 86% less on both.**
- **At quarterly volume: about USD 14,000 instead of USD 98,000, about 5,800 compute hours instead of 41,500, and 8.8 million fewer calls to the agent.**
- **The same budget now covers about seven times as many candidate pairs**, so catalog cleanup can keep pace with catalog growth.

## Case 2: finding piracy offers for a media group

### Why it matters: every pirate subscription is a customer the client already lost

For a media group with live football rights in Latin America, piracy is a subscriber problem. In Chile, its sports channel went from about 1 million subscribers to about 600,000 in five years, and the company believes most of that gap went to piracy. More than a million Chilean homes are estimated to use illegal IPTV services, which sell hundreds of channels, football included, for about a third of the price of a legal subscription.

Those homes are not lost for good. Pirate IPTV users already pay for TV every month, and when they lose access, many come back. When IPTV services were blocked in Argentina in February 2026, the client's streaming service gained **31,000 incremental subscriptions** and a **67% jump in gross sign-ups**. By the client's own estimate, winning back 10% of Chile's pirate IPTV homes would be worth about **USD 1.6 million a month**.

Blocking streams is slow, and blocking domains in real time is not legally available in Chile or Argentina. The weak spot is the storefront. Pirate services find customers on Instagram posts and Facebook Marketplace listings, and they close each sale through WhatsApp or a direct message. Every offer found and reported early cuts one sales channel for a pirate operator.

### The challenge: a two-person team and filters that could not tell an offer from noise

Before this project, the social media team watched official accounts by hand on match days and sent screenshots over WhatsApp to an anti-piracy team of **two people**. What they found worked: in one case, accounts reselling match access were taken down within hours. The problem was scale.

Marvik built a monitor to close that gap, with the same pipeline as the catalog. Browser agents search Instagram and Facebook Marketplace in Argentina and Chile, and a first filter decides which candidates become cases, with a screenshot saved as evidence. Each case then goes to an **analysis agent**, a generative LLM that reads the post and its screenshot, decides whether it is a pirate offer and extracts its details. Analysts review what the agent confirms and report the listing. The client's target: **at least 80% of flagged cases confirmed as piracy.** With two people on the team, that share is enforcement capacity: every false alarm is time not spent on a real offer.

The weak point was the first filter on each surface:

- **On Facebook Marketplace, keyword rules on the title decided which listings to open.** Rules cannot read context, and a title alone is rarely enough. In a test, the rules recovered **only about 3% of the pirate offers** among the listings they had set aside.
- **On Instagram, a generative LLM gate answered yes or no for each post.** It was reasonably accurate but slow (3.2 seconds per post), and it **failed open**: when a call errored, the post went through. In one September week, **almost 9 of every 10 Instagram cases** that reached the analysis agent were not piracy, most likely because of a provider error the gate treated as a yes.

Saying no well is harder here than it looks. In Argentina, "TV digital" is the street name for pirate IPTV and also the name of legal free-to-air TV. Pirate apps change names every few months. And the offer is often only in the image: about three in four pirate posts missed by a text-only model had the offer in the flyer, not the caption.

### How it works

The same component took two different places:

1. **Instagram: one question and the screenshot.** The decision model replaces the LLM gate. It sees the caption, with phone numbers, links and usernames masked, and the screenshot. A second question checks that the post is in Spanish, a requirement the client set for its market.
2. **Facebook Marketplace: a two-stage cascade.** The keyword rules are gone. **Stage 1 reads only the text and can only reject**: a listing with a pirate probability of 0.2 or less is discarded, and everything else moves on, because some offers only show in the image. **Stage 2 takes the screenshot**, the expensive part (loading the page, three times the tokens, more exposure to anti-bot controls), and accepts at 0.5. That screenshot becomes the case's evidence.

The client's enforcement policy went into the wording. *Yes* means selling a service, a channel list, an app, premium accounts, or a TV box shipped with pirate apps. *No* means hardware alone, an identified internet provider selling its own plan, news about a blocked app, or merchandise. The TV box rule came from the data: in an early test, about two thirds of the false alarms were boxes the human labeler had marked as not pirate, while the analysis agent had confirmed every such box as piracy. The client settled it, and the question says so.

### Results

On Instagram, we replayed real posts with their screenshots and compared the new filter with the old gate, using the analysis agent's verdicts as the reference:

- **It catches more pirate offers and raises fewer false alarms.** It finds 94% of the pirate offers, up from 88%, and 93% of what it flags is real piracy, up from 91%.
- **Median time per post fell from 3.2 to 0.85 seconds.**
- **Between 91% and 94% of what it flags is real piracy in every variant we tested**, above the client's 80% target.
- **Quality barely moved across a wide range of thresholds**, so the client can make the filter stricter or looser without losing accuracy.
- **Requiring Spanish means catching 82% of pirate offers instead of 94%**, with the same share of real piracy among flagged posts. The client accepted that trade-off: the lost offers are outside its market.
- The filter costs **about USD 0.29 per 1,000 posts.**

On Facebook Marketplace, the first live run in Buenos Aires and Santiago showed where the cascade pays:

- **The text stage discarded 87% of listings without taking a screenshot**, and about 12% were accepted as cases. Median time per listing was 425 milliseconds.
- **The savings are in screenshots, not tokens**: about one page capture for every eight listings, for a model cost of about USD 0.14 per 1,000 listings.
- These listings have no human labels yet, so this run does not yet measure how many offers were caught or missed.

## What these projects show: learn to say no

1. **Don't replace the judge; put a gate in front of it.** In both pipelines, an LLM agent still decides whether two pages are the same product or a post is a pirate offer. In the catalog, the cheap model could not make that call alone, but as a gate it kept at least 95% of duplicates and sent the agent only 1 in 8 pairs. In anti-piracy, it took the place of a keyword rule and an LLM gate, and left the final call to the analysis agent and the analysts.
2. **Ask for probabilities, not verdicts.** One client chose how many duplicates it could afford to miss; the other chose to require Spanish and catch fewer offers outside its market. Both were business decisions made on a dial.
3. **Write the policy into the question, including the local vocabulary.** Barcodes, TV boxes and "TV digital" were not model problems. They were rules and words someone had to write down.
4. **Decide what the gate does when it fails.** A gate that says yes when in doubt can turn one provider error into a week of junk cases for the people downstream.
5. **Measure the expensive step before you optimize it.** The catalog agent cost twice the estimate; in anti-piracy, the expensive part was screenshots. Knowing which tells you where the gate pays.
6. **Labels are part of the system.** About 8% of the catalog's duplicate labels looked wrong on review, and every anti-piracy metric so far is measured against the analysis agent, not against people.

## Where they stand and what's next

Both systems are on their way to production and already changing how each client works. In the catalog, the gate is moving to live traffic next to the current agent, where its savings are checked against the agent's real bill. In anti-piracy, both filters already decide which posts and listings become cases, so the client's team spends its time on offers that are likely to be real, and human review keeps sharpening the questions.

The projects also opened new challenges for the clients, where we are now studying the same framework. The most promising are real-time decisions, where an LLM cannot answer in time: checking for fraud at checkout in an e-commerce marketplace, or filtering problematic products out of search results before they are shown. A decision model answers in a fraction of a second, which puts those jobs within reach.

**Expensive models should spend their time where judgment is needed. A good gate makes sure they do.**
