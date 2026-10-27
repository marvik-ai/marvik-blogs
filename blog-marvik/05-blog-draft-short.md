---
title: "Put a gate before the judge: how to reduce LLM costs in product matching and piracy detection"
seo_title: "Cut LLM Agent Costs 86% with a Decision-Model Gate"
meta_description: "How cheap decision models in front of LLM agents cut duplicate detection costs 86% and made piracy filtering nearly 4x faster in two production AI pipelines."
slug: reduce-llm-costs-decision-model-gate
keywords:
  primary: reduce LLM costs
  secondary:
    - LLM cost optimization
    - LLM inference cost
    - LLM agent cost
    - LLM-as-a-judge
    - LLM cascade
    - decision model
  case_1:
    - duplicate product detection
    - product matching
    - e-commerce catalog deduplication
  case_2:
    - IPTV piracy detection
    - social media piracy monitoring
    - anti-piracy AI
category: Business Impact
author: Marvik
status: DRAFT v4 short (2026-10-24). Short version of 05-blog-draft.md with the SEO and clarity changes from the long-vs-short review. Not for publication.
pending:
  - Both clients stay unnamed. Final decision, not open for discussion.
  - CTA offers a one-week log review. Confirm that Marvik can deliver it (and offer the NDA) before publishing; add a turnaround time if Marvik can commit to one, or switch to the plain "we'd love to talk" closing.
  - Media group figures come from a confidential strategy document and need client approval - subscribers (1M to 600k), 1M+ pirate IPTV homes, one-third price, Argentina blocking (+31,000 subscriptions, +67% gross sign-ups), USD 1.6M a month, legal status of real-time blocking, two-person team, 80% precision target, takedown anecdote.
  - "Seven times as many pairs" is our own calculation (USD 9.76 / 1.38 per 1,000 pairs).
  - Catalog quarterly volume (10M pairs) needs confirmation. Derived figures use the 10k README per 1,000 pairs (USD 9.76 vs 1.38; 249 vs 35 minutes; 12.3% pass) - USD 97,600 vs 13,800, 41,500 vs 5,800 hours, 8.77M fewer calls.
  - Production status of both systems: earlier notes describe a PoC (catalog) and staging with enforce by default (anti-piracy). Confirm with both clients before saying "on its way to production". The "How we worked" step 5 (shadow run) depends on this.
  - Anti-piracy business result: get a team-level number from the client (analyst hours saved per week, or confirmed cases per analyst before and after) and fill the [TBD] in Case 2 results, or delete that bullet.
  - Check the product names and links for OpenAI's Decisions API and TypeSafe's JEV, and add them as external links.
  - Assets to produce - (1) pipeline diagram after the intro, alt text "Candidate generator sends items to a cheap decision-model gate; only likely positives reach the LLM agent, then human review"; (2) FAQ schema (FAQPage JSON-LD) built from the FAQ section.
  - Layout - render the five "How we worked" steps as step cards in Webflow (one card per step - number, title, one-line description, Catalog and Anti-piracy rows), horizontal on desktop and stacked on mobile.
  - "What is at stake" table - confirm catalog scale (1B+ items, ~100M product pages) with the client and check it does not identify it; the piracy losses (USD 10B / 2.4B) come from secondary coverage of the Alianza + Fabric Q4 2025 report (March 2026), so link the original report if available.
  - Before publishing - remove this frontmatter block and map seo_title, meta_description and slug to the Webflow fields.
---

# Put a gate before the judge: how to reduce LLM costs in product matching and piracy detection

*How Marvik helped a leading Latin American e-commerce marketplace and a global media group put cheap decision models in front of their most expensive AI steps: 86% less cost and compute time in duplicate product detection, and a piracy filter that is more accurate and nearly four times faster than the one it replaced.*

**What is at stake**

| | Catalog | Anti-piracy |
|---|---|---|
| Scale | 1B+ items across ~100M product pages; ~10M candidate pairs to review a quarter | 1M+ pirate IPTV homes in Chile; the client's sports channel there fell from ~1M to ~600,000 subscribers in five years |
| Cost of the problem | ~USD 100k and ~42k of compute hours a quarter to review every pair with an LLM agent | Online piracy losses in Latin America: ~USD 10 billion a year, plus ~USD 2.4 billion lost by pay-TV operators ([Alliance Against Audiovisual Piracy](https://piracymonitor.org/latin-america-in-q4-of-2025-pirates-invaded-46-million-online-households-nearly-10-billion-lost/)) |

**Key numbers**

| | Before | After |
|---|---|---|
| Catalog: cost per 1,000 product pairs | USD 9.76 | USD 1.38 (−86%) |
| Catalog: pairs sent to the agent | 100% | 12.3% (95% of duplicates kept) |
| Anti-piracy (Instagram): time per post | 3.2 s | 0.85 s |
| Anti-piracy (Instagram): pirate offers caught | 88% (with cheap LLM) | 94% (with decision model)|

## Most of what an LLM agent reviews is a "no"

Many AI systems in production share the same pipeline. Something upstream proposes candidates: two product pages that might be the same item, or a post that might be selling pirated TV. At the end, an expensive LLM agent (the *judge*) reads each candidate and makes the call. In between, something has to decide what reaches the judge. We call that step the *gate*, and it is where you can reduce LLM costs the most.

The usual gate is a set of rules, such as regular expressions over titles or shared identifiers. Rules are cheap, but they cannot read context, so they miss many true positives. The alternative is to send everything to the agent, which then spends most of the LLM budget saying no: **97 of every 100 product pairs were not duplicates**, and **about 9 of every 10 Instagram posts or Facebook Marketplace listings** opened while searching for piracy were not pirate offers.

## Why decision models make a good gate for LLM cost optimization

Instead of asking a cheap model for a verdict, we asked it a few closed questions about each candidate and kept the probabilities. **Decision models**, such as OpenAI's Decisions API or JEV from TypeSafe, answer a question by choosing among a fixed set of options and return a probability for each one. That makes them a good gate:

- **They are cheap and fast.** The output is not billed: a fraction of a cent per item, in under half a second on text and under a second with a screenshot.
- **A probability works like a dial.** The business sets the threshold to the error it can tolerate.
- **The policy lives in the question.** The criteria are versioned and tested on labeled data before they change, and every decision is logged with its probabilities and question version, so anyone can audit why a candidate was dropped.

The gate never makes the final call. It only decides which candidates reach the agent.

## How we worked

Both projects followed the same five steps.

**Step 1 · Measure the expensive step**
Run it on real candidates to know what each decision costs and how long it takes.
- *Catalog:* the agent took about 15 seconds and one US cent per pair.
- *Anti-piracy:* the screenshots cost more than the tokens.

**Step 2 · Write the client's policy into closed questions**
Business rules and local vocabulary go into the wording.
- *Catalog:* a different barcode is seller noise; "Preto", "Negro" and "Black" are the same color.
- *Anti-piracy:* a TV box shipped with pirate apps counts as piracy; "TV digital" can mean either.

**Step 3 · Set the target and tune to it**
The client picks the error it can live with; we tune questions and thresholds on labeled examples.
- *Catalog:* keep at least 95% of true duplicates.
- *Anti-piracy:* at least 80% of flagged cases must be real piracy.

**Step 4 · Test once, blind**
Every filter runs unchanged on data it has never seen.
- *Catalog:* one week of real traffic; one in four filters that passed in development failed here.
- *Anti-piracy:* real Instagram posts replayed against the old gate.

**Step 5 · Run next to the current system**
The gate works on live traffic before it takes over.
- *Catalog:* moving to live traffic next to the current agent.
- *Anti-piracy:* first live run in Buenos Aires and Santiago.

## Case 1: duplicate product detection in an e-commerce catalog

### Why it matters: e-commerce catalog deduplication at scale

In a large marketplace, every seller can create their own page, so the same product gets listed again and again. Buyers find the same headphones five times, with reviews spread across pages and no single place to compare prices. Sellers see their sales history and search ranking diluted. And the more the catalog grows, the more pairs need a decision, so keeping it clean cannot depend on a review bill that grows at the same rate. This project cut that bill, so the same budget now reviews about seven times as many pairs. Its effect on sales is not measured yet.

### The challenge: an accurate judge that spends most of its time saying no

Rules such as shared identifiers propose candidate pairs, but they cannot settle the question: two pages with different titles can be the same product, and two nearly identical ones can differ in color or pack size. So an LLM agent judges each pair by reading both pages and their photos, as a human catalog analyst would.

The agent is accurate, but slow and expensive. Measured on real pairs, each decision takes **about 15 seconds and costs about one US cent**. At roughly 10 million pairs per quarter, that is around **USD 98,000 and 41,500 compute hours a quarter**, nearly all of it spent confirming that two pages are *not* the same product.

The gate had to let through **at least 95% of true duplicates**, because the agent cannot merge what it never sees, and filter out as many non-duplicates as possible. Our first approach was to replace the agent with a cheaper model running the agent's own prompt. It caught only 40% to 57% of duplicates, too few to make the final call. But a cheap model can do an easier job: discard the pairs that are clearly not duplicates and send the rest to the agent.

### How the gate works: three ways to ask the question

We tested three families of gates, each with and without the product photo:

1. **A single question**, a simplified version of the agent's own prompt.
2. **A hand-written question bank.** Closed questions about each way two pages can differ (variant, sale unit, or a product against its accessory), combined into one score by a small trained model, the *decider*.
3. **An LLM in a research loop**, which generates, rewrites and prunes questions while a decider scores each set.

The client's merge policy went into the wording: a different barcode is seller noise, "Generic" is a neutral brand, and "Preto", "Negro" and "Black" are the same color.

There was no single winner. The single question was the cheapest but kept only 93.7% of duplicates. A bank of four broad questions, asked in one call with the titles, attributes and first photo of each page, filtered the most while keeping 95%. A bank tuned for 97% missed fewer duplicates but sent about a fifth more pairs to the agent. Choosing among them is a business decision; the results below use the four-question bank.

The blind test used a separate week of real traffic at the real share of duplicates (3%), with no product shared with development. It paid for itself: **one in four filters that met the target in development fell below it on new data.**

### Results: 86% lower cost per product pair

On the blind test, the four-question bank with photos downscaled to 512 pixels delivered:

| | Agent alone | Gate + agent |
|---|---|---|
| Duplicates that reach the agent | 100% | 95.0% |
| Pairs sent to the agent | 100% | 12.3% |
| Cost per 1,000 pairs | USD 9.76 | USD 1.38 |
| Compute time per 1,000 pairs | 249 min | 35 min |
| Cost per quarter (~10M pairs) | ~USD 98,000 | ~USD 14,000 |
| Compute hours per quarter | ~41,500 | ~5,800 |

The gate filters out 90.3% of non-duplicates and saves 8.8 million agent calls a quarter. **The same budget now covers about seven times as many candidate pairs**, so catalog cleanup can keep pace with catalog growth.

## Case 2: IPTV piracy detection on Instagram and Facebook Marketplace

### Why it matters: every pirate subscription is a customer the client already lost

For a media group with live football rights in Latin America, piracy is a subscriber problem. In Chile, its sports channel went from about 1 million subscribers to about 600,000 in five years, and the company believes most of that gap went to piracy: more than a million Chilean homes are estimated to use illegal IPTV services, at about a third of the price of a legal subscription.

Those homes are not lost for good. When the government blocked IPTV services in Argentina in February 2026, the client's streaming service gained **31,000 incremental subscriptions** and a **67% jump in gross sign-ups**. By the client's own estimate, winning back 10% of Chile's pirate IPTV homes would be worth about **USD 1.6 million a month**. These figures size the opportunity. They are not results of this project.

Blocking domains in real time is not legally available in Chile or Argentina. The weak spot is the storefront: pirate services find customers on Instagram posts and Facebook Marketplace listings, and close each sale through WhatsApp or a direct message. Every offer found and reported early cuts one sales channel for a pirate operator.

### The challenge: a two-person team and filters that could not tell an offer from noise

Before this project, the social media team watched official accounts by hand on match days and sent screenshots over WhatsApp to an anti-piracy team of **two people**. What they found worked: in one case, accounts reselling match access were taken down within hours. The problem was scale.

Marvik built a social media piracy monitor with the same pipeline as the catalog. Browser agents search Instagram and Facebook Marketplace in Argentina and Chile, a first filter decides which candidates become cases, and an **analysis agent** (a generative LLM) reads each case, decides whether it is a pirate offer and extracts its details before analysts report it. The client's target: **at least 80% of flagged cases confirmed as piracy**, because with two people on the team, every false alarm is time not spent on a real offer.

The weak point was the first filter on each surface:

- **On Facebook Marketplace, keyword rules on the title decided which listings to open.** In a test, they recovered **only about 3% of the pirate offers** among the listings they had set aside.
- **On Instagram, a generative LLM gate answered yes or no for each post.** It was accurate (91% precision) but slow (3.2 seconds per post), and it **failed open**: when a call errored, the post went through as if the answer were yes. In one September week, **almost 9 of every 10 Instagram cases** that reached the analysis agent were not piracy.

Saying no well is harder than it looks. In Argentina, "TV digital" is the street name for pirate IPTV and also the name of legal free-to-air TV. And the offer is often only in the image: about three in four pirate posts missed by a text-only model had the offer in the flyer, not the caption.

### How the gate works

We used the same component in two places:

1. **Instagram: one question and the screenshot.** The decision model replaces the LLM gate. It sees the masked caption and the screenshot, and a second question checks that the post is in Spanish, as the client requires.
2. **Facebook Marketplace: a two-stage cascade** replaces the keyword rules. **Stage 1 reads only the text and can only reject**: listings at a pirate probability of 0.2 or less are discarded, because some offers only show in the image. **Stage 2 takes the screenshot**, the expensive part (page load, three times the tokens, anti-bot exposure), and accepts at 0.5. That screenshot becomes the case's evidence.

The client's enforcement policy went into the wording. *Yes* means selling a service, a channel list, an app, premium accounts, or a TV box shipped with pirate apps. *No* means hardware alone, an internet provider selling its own plan, or news about a blocked app. The TV box rule came from the data: about two thirds of the false alarms in an early test were boxes the human labeler called legal and the analysis agent called piracy. The client settled it, and the question says so.

### Results: a faster, more accurate piracy filter

On Instagram, we replayed real posts with their screenshots and compared the new filter with the old gate, using the analysis agent's verdicts as the reference:

| | Old LLM gate | Decision-model gate |
|---|---|---|
| Pirate offers caught (recall) | 88% | 94% |
| Flagged posts that are real piracy (precision) | 91% | 93% |
| Median time per post | 3.2 s | 0.85 s |
| Cost per 1,000 posts | not measured | ~USD 0.29 |

- **Precision stayed between 91% and 94% in every variant**, 11 to 14 points above the client's 80% target, and stayed in that range across thresholds, so the filter can be made stricter or looser without losing accuracy.
- **Requiring Spanish lowers recall to 82%** at the same precision. The client accepted that: the lost offers are outside its market.
- **For the two-person anti-piracy team, that means [X] confirmed offers reported per week, up from [Y] before the monitor, and about [Z] hours a week no longer spent on false alarms.** [TBD with the client]

On Facebook Marketplace, the first live run in Buenos Aires and Santiago showed where the cascade pays. **The text stage discarded 87% of listings without taking a screenshot**, at a median of 425 milliseconds each, so the savings are in page captures (about one for every eight listings), at a model cost of about USD 0.14 per 1,000 listings. Fewer captures mean fewer page loads and less exposure to anti-bot blocks.

## What these projects show: learn to say no

1. **Don't replace the judge; put a gate in front of it.** In both pipelines, an LLM agent still makes the final call. The gate decides what reaches it: 1 in 8 pairs in the catalog, and only the likely offers in anti-piracy.
2. **Write the policy into a question that returns a probability.** Barcodes, TV boxes and "TV digital" were rules and words someone had to write down, and the threshold is where each client chose which error to live with.
3. **Decide what the gate does when it fails.** A gate that says yes when in doubt can turn one provider error into a week of junk cases for the people downstream.
4. **Measure the expensive step, and the labels you measure against.** In the catalog, the agent's cost per pair set the size of the savings; in anti-piracy, the expensive part was screenshots, not tokens. And about 8% of the catalog's duplicate labels looked wrong on review.

## Frequently asked questions

**What is a decision model?**
A model that picks among fixed options and returns a probability for each, instead of generating text. With no output to pay for, it costs less and answers faster than a generative LLM call.

**How is a decision-model gate different from LLM-as-a-judge?**
The judge, a generative LLM, makes the final decision. The gate sits before it and only removes clear negatives.

**When should you put a gate in front of an LLM agent?**
When most candidates are negatives (about 90% or more in both projects), each agent call is slow or expensive, and what you look for fits in a few closed questions.

**How much does a decision-model gate cost?**
In anti-piracy, USD 0.14 to 0.29 per 1,000 items. In the catalog, gate plus agent came to USD 1.38 per 1,000 pairs, against USD 9.76 for the agent alone.

## Where they stand and what's next

Both systems are on their way to production. The catalog gate is moving to live traffic next to the current agent, and the anti-piracy filters already decide which posts and listings become cases. Next, we are studying the same framework for real-time decisions where an LLM cannot answer in time, such as fraud checks at checkout or filtering problematic products out of search results.

More on LLMs in production: [securing LLM applications](https://www.marvik.ai/blog/beyond-the-prompt-5-realities-of-securing-llm-applications), [building a text-to-SQL agent on Databricks](https://www.marvik.ai/blog/genie-spaces-vs-mosaic-ai-agent-framework-building-a-text-to-sql-agent-on-databricks) and [the cost of Claude Managed Agents](https://www.marvik.ai/blog/claude-managed-agents-technical-overview-architecture-and-cost-analysis).

At Marvik, we build AI systems that run in production and pay for themselves. If your pipeline sends an expensive model thousands of candidates a day and most of the answers are no, send us one week of its logs. Anonymized logs are fine, and we sign an NDA first. We'll tell you what the LLM step costs today and how much of it a gate could save. [Contact link]

**Expensive models should spend their time where judgment is needed. A good gate makes sure they do.**
