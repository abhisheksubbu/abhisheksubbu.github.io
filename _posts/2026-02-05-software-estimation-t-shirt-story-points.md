---
title: Software Estimation - T-Shirt Sizing and Story Points
layout: blog
category: [Software, Architecture]
excerpt: A practical guide to software estimation - what it is, T-shirt sizing, and Story Point Estimation with Risk, Complexity, Repetition, and triangulation.
comments: true
---

*This post expands on [Project Estimation from the Architect-101 series](/architect-101/){:target="_blank"}.*

Estimation is one of the most talked-about and often misunderstood activities in software delivery. Done well, it helps teams commit realistically, plan capacity, and spot work that needs to be broken down. Done poorly, it becomes a source of stress, missed deadlines, and distrust. This article covers what software estimation is, two common approaches: T-shirt sizing and Story Points - and how to use Story Points in a disciplined way.

---

## What is Software Estimation?

**Software estimation** is the process of predicting the effort, time, or cost required to deliver a piece of work - a feature, user story, epic, or project. It is not a promise or a guarantee; it is a *forecast* based on what we know today.

Why we estimate:

- **Planning** - Decide what fits in a sprint, release, or quarter.
- **Prioritization** - Compare relative size and value of items.
- **Capacity** - See how much the team can take on.
- **Conversation** - Uncover ambiguity, dependencies, and missing information.
- **Improvement** - Reflect on accuracy over time and calibrate.

Estimation is useful when it drives these outcomes. It becomes harmful when it is treated as a commitment to a fixed date without re-estimating as we learn.

---

## Types of Estimation

Two widely used *relative* estimation approaches are **T-shirt sizing** and **Story Point Estimation**. Both compare work items to each other instead of guessing hours.

---

## T-Shirt Sizing

**T-shirt sizing** uses labels like XS, S, M, L, XL (and sometimes XXL) to indicate relative size. No numbers are attached-only order of magnitude.

| Size | Meaning (typical) |
|------|--------------------|
| **XS** | Trivial; minutes to an hour. |
| **S** | Small; part of a day. |
| **M** | Medium; a day or a few days. |
| **L** | Large; a week or more. |
| **XL** | Very large; multiple weeks; consider splitting. |

**When to use:** Roadmap discussions, epics, initiatives, or when the audience is non-technical. It is quick and avoids false precision. It is less suitable for sprint planning where you need finer granularity.

**Limitation:** Only a few buckets, so many items land in “M” or “L” and still need to be broken down or refined before development.

---

## Story Point Estimation

**Story Points** are a unit of *relative* size. They combine effort, uncertainty, and complexity into a single number so that the team can compare stories and plan sprints. Points are *not* hours - they are consistent within a team over time.

### The Fibonacci Scale (and a cap at 13)

Story points are usually chosen from a **Fibonacci-like sequence** so that the gap between numbers grows as size grows. This reflects the fact that we are less precise when things are bigger.

A common scale is:

**0, 1, 2, 3, 5, 8, 13** (and sometimes **21** for “too big to fit in a sprint”).

- **0** - Already done or negligible (e.g. duplicate, clarification only).
- **1** - Tiny; minimal effort and no real risk.
- **2** - Small.
- **3** - Small–medium.
- **5** - Medium.
- **8** - Large; should make you pause.
- **13** - Very large; strong signal to split before committing.
- **21** - Treated as “a project on its own”; not a single story. Break it down.

**Why cap at 13 (or treat 21 as out-of-scope for a single story)?**  
High point values mean high uncertainty. A “13” or “21” is a reminder: *don’t commit to this as one story*. Split it into smaller, shippable stories with lower points (e.g. 3, 5, 8). So: **higher story points are an indicator to break that user story into more achievable stories or tasks with smaller points.**

---

### What a Story Point Should Account For

A story point should reflect three factors:

1. **Risk**  
   Unclear demand, dependence on a third party, or uncertainty about the future.  
   - Unclear or changing requirements.  
   - External APIs, vendors, or systems we don’t control.  
   - Unknowns in technology, performance, or integration.  
   More risk → consider a higher point or splitting the story.

2. **Complexity**  
   Effort and brainpower needed to design and implement the feature.  
   - A simple formula field or config change → low (e.g. 1–2).  
   - A flow or workflow with a few branches → medium (e.g. 2–3).  
   - Integration code, multiple systems, non-trivial logic → higher (e.g. 3–5 or more).  
   Complexity drives *effort*; risk drives *uncertainty*.

3. **Repetition**  
   How much the team has done something similar before.  
   - “We’ve built this kind of integration before” → easier, fewer points.  
   - “I know this module well; it’s mostly repetition” → 1 or 2.  
   Repetition reduces both effort and uncertainty, so points go down.

So: **Story Point ≈ f(Risk, Complexity, Repetition)**. Same “raw” effort can be 3 or 5 depending on risk and repetition.

---

### Checklist Before Assigning Points

Before the team assigns a number, make sure these are discussed:

- **Do we have design?** (even rough wireframes or flows)
- **How much code do we expect?** (order of magnitude: one file vs many, new service vs config)
- **What is the acceptance criteria?** (clear “done” definition)
- **Any integration dependencies?** (other teams, APIs, legacy systems)
- **Do we have the expertise?** (in-house vs learning on the job)
- **How complex is it?** (straightforward vs multi-step, many edge cases)
- **How much uncertainty or risk exists?** (unknowns, third parties, changing requirements)

If most answers are “we don’t know,” the story is not ready to estimate. Refine it first, or give it a high point and plan to split.

---

### Triangulation: Why Not 3 or 8?

**Triangulation** means checking your estimate against nearby numbers. If you think a story is **5**, explicitly ask:

- *“Why is it not a 3?”* (smaller, less risk, or more repetition?)
- *“Why is it not an 8?”* (more complexity, more unknowns?)

This does two things:

1. **Surfaces assumptions** - Someone might be thinking of extra scope or a dependency you didn’t consider.
2. **Calibrates the team** - Over time, everyone converges on a shared understanding of what 3, 5, and 8 mean.

So: **triangulate on the points.** “I put 5-can we debate why it’s not 3 or 8?” is a healthy ritual in refinement or planning.

---

## Summary

| Concept | Takeaway |
|--------|----------|
| **Software estimation** | A forecast of effort/time/cost to support planning and conversation, not a fixed promise. |
| **T-shirt sizing** | XS–XL for high-level, roadmap or epic-level comparison; quick and coarse. |
| **Story points** | Relative size in a Fibonacci-like scale (0, 1, 2, 3, 5, 8, 13; 21 = split). |
| **High points** | Signal to break the work into smaller, achievable stories with lower points. |
| **Three factors** | Risk (uncertainty, dependencies), Complexity (effort to build), Repetition (familiarity). |
| **Before pointing** | Design, scope, acceptance criteria, dependencies, expertise, complexity, risks. |
| **Triangulation** | Debate why the story is not the next number down or up (e.g. why 5 and not 3 or 8). |

---

## Common Questions on Estimation

### Is it wrong to map 1 story point to 0.5 days (or 4 hours)?

**Short answer:** It’s not wrong, but it’s a trade-off.

Many teams use a rough conversion like “1 point = 0.5 days” or “1 point = 4 hours” for capacity planning or to explain velocity to stakeholders. That’s **acceptable** and can be useful when:

- You need to translate “we can do ~20 points per sprint” into calendar time for release planning.
- Stakeholders think in days, not points, and a simple rule helps communication.
- The same team keeps the same conversion so it stays consistent.

The **risks** of fixing “1 point = X hours” are:

- **Points get treated like hours** - People start estimating in hours and then converting to points, which defeats the purpose of relative sizing (Risk, Complexity, Repetition).
- **Pressure and gaming** - If management insists “1 point = 4 hours,” teams may inflate or deflate points to match expectations instead of sizing honestly.
- **Cross-team misuse** - One team’s “1 point = 4 hours” is not comparable to another team’s; velocity and conversion are team-specific.

**Recommendation:** Use a conversion only as an *internal, loose* guideline for planning (e.g. “we treat 1 point as about half a day for capacity”). Don’t bake it into contracts or use it to compare teams. Keep estimating in relative terms first; convert to time only when you need to communicate or plan.

---

Estimation is a team skill. Keep the scale consistent, cap big numbers so they get split, and use Risk, Complexity, and Repetition plus a short checklist and triangulation - so that Story Points and T-shirt sizes actually improve planning and delivery rather than becoming a ritual.
