# Genie Spaces vs. Mosaic AI Agent Framework: Building a Text-to-SQL Agent on Databricks

- Date: June 4th, 2026
- Category: Technical
- Authors: Mateo Madrigal; Diego Zimmerman; Iara Osatnik; Carolina Piro
- URL: https://www.marvik.ai/blog/genie-spaces-vs-mosaic-ai-agent-framework-building-a-text-to-sql-agent-on-databricks

---

Every few weeks, someone on the business side would ask the same thing: “Can I just type my question and get the data?” After hearing it enough times, we built a Text-to-SQL agent on Databricks, and ended up comparing two very different paths: Genie Spaces and the Mosaic AI Agent Framework.

The concept sounds simple. A user types a question in plain English. The agent figures out what they mean, writes the SQL, runs it against Databricks, and returns an answer. In a demo, it looks almost magical. In a real governed environment with actual business data, it’s a different story.

The first thing we learned is that SQL generation is the easy part. The harder question is whether the agent can understand your data model, work within your security rules, and return answers that actually hold up when someone acts on them.

That brought us to a real decision: use a managed Databricks experience, or build the agent ourselves?

On one side, Genie Spaces, Databricks’ managed conversational analytics experience, looked like the fastest path. On the other side, Mosaic AI Agent Framework, Databricks’ proprietary framework for building custom agentic applications in Python, offered control over every layer. More powerful, but also significantly more work.

We tested both with the same dataset and the same business questions not just, to see which produced better SQL, but to understand what each approach actually required.

## What is a Text-to-SQL Agent? Genie Spaces and Mosaic AI Explained

Before going into what we built, it helps to define the three main pieces.

A Text-to-SQL agent is an AI system that takes a natural language question and turns it into executable SQL. In a demo it looks straightforward: a user types “What is the current balance for account 1001?” and the agent produces SELECT balance FROM accounts WHERE account_id = 1001. In a real governed environment, the same question also requires the agent to know which table “accounts” maps to, whether the current user is allowed to see that row, and what “balance” means in your schema.

Genie Spaces are Databricks’ managed option for this experience. You configure a Space over selected tables, views, and Unity Catalog metadata, layer in instructions and examples, connect a SQL Warehouse, and get a working conversational interface without building the application layer yourself. Out of the box: SQL generation, query execution, response presentation, Unity Catalog permissions, and a way to add business context. The trade off is control you can guide Genie, but you don’t own the reasoning flow, tool orchestration, memory, or validation logic.

Mosaic AI Agent Framework is Databricks’ proprietary framework for building custom agentic applications. It is not a generic open source library. It is a native part of the Databricks platform, designed so teams can define the model, tools, prompts, orchestration logic, evaluation, and deployment within the same governed environment where their data lives.

## The first realization: SQL generation was not enough

This one was obvious in retrospect, but it took some broken queries to really land.

A Text-to-SQL agent is not useful just because it can generate SQL. A query can be syntactically perfect and still be wrong:

- It can join through a deprecated relationship.

- It can misread what “active account” means in your context.

- It can pull a field that should be masked.

- It can return data a particular user isn’t supposed to see.

So the real work wasn’t building the agent. It was building the context the agent needed to reason correctly: what each table represents, what each column means in business terms, how entities relate to each other, which filters are approved, who can access what, and which Unity Catalog functions should be called as controlled tools.

## Before comparing the tools, we needed a shared foundation

Before comparing Genie and Mosaic, we had to make sure both were working from the same base. Otherwise we wouldn’t be comparing the tools, we’d be comparing two different versions of the problem.

That meant documenting tables and columns, encoding business rules that previously lived only in people’s heads, validating permissions, configuring masking and row level security, and wrapping reusable logic into Unity Catalog functions. It took longer than expected. But it also made everything that came after much more honest.

In a production environment, an agent can’t rely on a clever prompt to compensate for missing context. It needs a data foundation that was actually prepared for it. That preparation is where most of the real work lives, regardless of which tool you choose.

## The shared challenge: making Unity Catalog functions work for both agents

Once the foundation was in place, we needed a way to expose reusable business logic to both agents consistently. Unity Catalog functions were the natural mechanism, instead of letting each agent rediscover the same logic on every call, we could define controlled functions for specific tasks.

This is where things got more nuanced than expected.

A developer can read documentation, explore a schema, ask a colleague. An agent decides whether to use a function based almost entirely on its name and description. That single difference changes what “well designed” means.

For agent use, functions need to be:

- Named clearly and self-explanatorily

- Narrow in scope, so the agent can match them to the right task

- Documented at the parameter level, so the agent knows what to pass

- Predictable in output, so the agent can reason over the result

- Correctly permissioned

- Not structured like a broad stored procedure that does several things at once

This was a design requirement, not a side effect of wanting functions to work in two tools. Functions that read like product interfaces worked. Functions that read like internal SQL shortcuts didn’t.

## How Genie Spaces Works for Text-to-SQL on Databricks

We started with Genie because it was the most direct route: a conversational interface over governed data, without writing the full agent stack from scratch.

The setup is simple. Create a Space, select tables or views, add instructions and examples, define relationships, connect a SQL Warehouse, start asking questions. For a self-service analytics use case aimed at business analysts, this is a genuinely strong starting point.

### Where Genie showed its limits

The limits showed up when we pushed beyond simple questions. Not because Genie failed, but because guiding behavior through configuration is different from controlling it through code. You can add instructions and examples, but you don’t control the reasoning loop directly.

### Testing Genie through the API

We also tested the API and Python path to see whether the setup could be versioned and automated. It can be, but it requires a lot of upfront structure. Tables, instructions, examples, joins, and semantic context all need to be explicitly defined. That makes it well-suited for a Space that’s already stable and needs to be operationalized. It’s a rough experience if you’re still figuring out what the Space should look like.

The way we think about it: use the UI while you’re discovering the right configuration, switch to the API once the Space is stable and you want to automate or version it.

One more thing worth noting, Genie can be called from outside Databricks through APIs, but the Space itself still runs inside the Databricks platform. An external app can send questions to an existing Space, but it doesn’t host or replace the runtime.

## Building a Custom Text-to-SQL Agent with Mosaic AI Agent Framework

Switching to Mosaic felt like going from a configured product to a proper engineering project.

Instead of adding instructions to a Space, we were writing Python. We controlled:

- LLM selection and system prompt design

- Which tools to expose and how the agent selects them

- Error handling and response evaluation

- Trace logging and agent deployment

That level of control is the main point of Mosaic. It’s not a more flexible version of Genie, it’s a different category entirely, a framework for building a custom agentic application. The use cases that make sense here are ones where the agent needs to do things a managed Space can’t:

- Call external APIs

- Combine Databricks data with documents

- Trigger workflows

- Follow orchestration logic too complex to express through configuration.

The cost of that control is engineering ownership. Someone has to design the agent, maintain the code, test tool behavior, evaluate outputs, and monitor production. That’s not a reason to avoid Mosaic, but it’s a real factor in whether a team is ready for it.

By the end of this path, the comparison had stopped being about features. It had become about two different ways of working.

## Where the comparison became real

The hardest part wasn’t building each agent. It was making them genuinely comparable.

To compare them fairly, both needed to work from the same base: the same Unity Catalog tables, the same business definitions, the same security rules, the same masking and row-level access logic, and the same Unity Catalog functions.

Getting there took more iteration than expected. Some functions had to be simplified. Some column comments had to be rewritten to be unambiguous. Some parameters needed better documentation. Some return formats had to be made more consistent.

Every time the shared foundation improved, both agents improved not just one of them. The quality of the agent is largely determined by the quality of the governed context behind it, not by the model or interface sitting on top.

### Cost and ownership: what changed between both paths

Auto-stop is a SQL Warehouse setting that shuts the warehouse down automatically after a defined period of inactivity, avoiding costs during quiet hours. For a small or medium team using the Space during business hours, costs can be reasonably predictable with the right warehouse size and auto-stop configuration.

The decision isn’t really about which option costs less. It’s about what kind of team you have and what you’re prepared to own over time. In our case, we only want to compare how each agent works for a simple case.

## 6 Lessons from Building a Text-to-SQL Agent on Databricks

Six things we’d tell ourselves if we were starting this over.

- Start with the data context, not the model. The LLM can generate SQL. Unity Catalog context is what makes the answer correct. If the metadata is missing or wrong, even a strong model produces plausible-looking queries that don’t actually work.

- Design Unity Catalog functions like product interfaces, not SQL shortcuts. The agent discovers tools by reading their name and description. Clear naming, narrow scope, documented parameters, and predictable outputs matter more than elegant internal logic.

- The Genie API is better for repeatability than for exploration. It’s the right tool once a Space is stable and you want to version or automate it. It’s a rough experience if you’re still figuring out what the Space should look like.

- Mosaic gives more control, but the ownership is real. Custom workflows, external integrations, and full orchestration control are genuine advantages but they come with a team commitment that shouldn’t be underestimated.

- Test security as part of the agent, not after it. Permissions, masking, and row-level security aren’t a final QA step. They affect what queries get generated and what data comes back. They’re part of the implementation from day one.

- The foundation matters more than the interface. We improved the Unity Catalog context midway through the experiment, and both agents got meaningfully better. Before picking a tool, ask whether your data is actually ready for an agent to reason over it.

## Conclusion

Both tools worked. Neither one did the hard part for us.

Genie made it fast to get a working interface in front of users, but it was less configurable in its specifics. Mosaic made it possible to build something that behaved exactly the way we needed, although it complicated each step along the way. But in both cases, the quality of the output came down to the quality of the metadata, the clarity of the business rules, and the design of the Unity Catalog functions. The tool was almost secondary.

Start with Genie when the goal is governed self-service analytics and the data foundation is reasonably clean. Move to Mosaic when the use case needs custom orchestration, external integrations, or behavior that configuration simply can’t express.

But before making that choice, spend time on the data. The question isn’t “which agent tool should we use?” It’s “is our data actually ready for an agent to reason over it?” Get that right, and the rest becomes a lot more tractable.

If we already have good data, we can think about what we are looking for: accessibility without freedom or freedom without accessibility.

## Keep reading

If this was useful, we write regularly about applied AI, data engineering, and building real systems on modern data platforms.

Explore more articles on Marvik's blogs
