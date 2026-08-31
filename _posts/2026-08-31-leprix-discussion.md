---
layout: post
title: "Workflow Crystallization Comes Before Sovereign AI"
---
A few weeks ago I sat in on a conversation with Brian, a CTO building infrastructure for global supply chains — inventory intake, supplier negotiation, cross-border compliance, the unglamorous plumbing that decides whether goods clear customs on time. The conversation ranged widely, from predictive analytics to the idea of an enterprise eventually running its own AI models instead of renting intelligence from frontier labs. But one thread stuck with me more than the others, because it cuts against how most companies are currently approaching AI: **you cannot distill what you have not first standardized.**

## The appeal of "just build our own model"

Every enterprise technology conversation now arrives, sooner or later, at the same fork: keep paying a frontier lab per token, or train something smaller and owned — a "sovereign" model, tuned to the company's own data and workflows, cheaper to run and immune to a vendor's roadmap changes. The economics are obviously attractive once volume is high enough. What's less obvious is what has to be true *before* that bet pays off.

Brian's team's actual day-to-day problem was not "we need a model." It was that the same inventory item, the same compliance exception, the same customer request got described five different ways by five different teams — buying, sales, operations, procurement — each with their own vocabulary and their own ad hoc spreadsheet. Before anyone could build an intelligence layer that flagged a mispriced item or a customs conflict, someone had to first agree on what a "correctly described item" even looked like across departments.

That ordering matters more than it sounds. A predictive system, or a future in-house model, trained on inconsistent, undifferentiated process data will just learn the inconsistency. It will get very good at predicting chaos. The fix isn't a smarter model — it's a legible workflow underneath it.

## Crystallization as the real prerequisite

The useful frame here is "crystallization": taking a workflow that currently lives in people's heads, in Slack threads, in a dozen slightly different spreadsheets, and turning it into a standardized, observable process with consistent inputs and outputs. Only once a workflow is crystallized does it become a candidate for automation at all — and only once it's automated at scale does it generate the kind of clean, task-specific data that makes distilling a smaller model worthwhile.

This is a familiar idea from software engineering, where teams have used similar clustering techniques to find the repeatable shape inside messy debugging workflows before automating them. Supply-chain operations are not so different: underneath the apparent chaos of exceptions and edge cases, there's usually a smaller number of recurring patterns than anyone believes, once someone bothers to name them consistently.

The order of operations, then, looks like this:

1. **Standardize terminology and structure** across the teams that touch a workflow, even before adding any intelligence.
2. **Observe and cluster** the now-legible workflow to find its real, recurring shape — where the genuine judgment calls are, and where they aren't.
3. **Automate the boring majority** of cases with rules or lightweight models, freeing frontier-model calls for the genuinely ambiguous ones.
4. **Only then distill.** With a large volume of consistently-labeled, narrow-task examples, a smaller in-house model can match a frontier model's performance on that specific task — at a fraction of the cost, and owned outright.

Skip straight to step four and you get an expensive way to automate your own confusion.

## Why this order gets skipped anyway

It gets skipped because standardization is organizationally hard and technically boring, while "let's fine-tune a model" is exciting and fundable. Crystallizing a workflow means sitting with procurement and sales until they agree on shared vocabulary — unglamorous, slow, political. Buying a model or a platform feels like progress in a way that a shared glossary doesn't, even when the glossary is the thing actually blocking value.

There's also a sequencing trap specific to compliance-heavy domains like cross-border trade: the intelligence an enterprise most wants — flagging a banned material before it ships, catching a pricing error before it's booked — only works if the underlying data about "what this item is" and "where it's going" is already consistent. Compliance and predictive-analytics layers are downstream of standardization, not a substitute for it, no matter how good the model underneath them is.

## The practical implication

If you're an operating team evaluating whether to build an in-house model this year, the honest first question isn't which model or which vendor. It's whether the workflow you want to automate is described the same way by everyone who touches it. If it isn't, the crystallization work — tedious, cross-functional, unglamorous — is the actual project. The model comes after, almost as a formality: a way of capturing, cheaply and repeatably, a pattern that by then is already understood.

Brian's team is still early in this — the standardization work is still in progress, the analytics layer still being scoped, the question of an owned model still a "later" conversation rather than a "now" one. That's the right order. I'd be skeptical of anyone telling a similar story in reverse.
