# Project Guide — Read Once, Then Forget

> This is the big picture. You do **not** need to hold all of this in your head.
> For "what do I do today?" open **`DAILY_TASKS.md`** instead.

## What am I building? (one sentence)
A small AI tool that reads a **test failure** (error + logs) and answers:
*what kind of failure is this, what probably caused it, and have we seen ones like it before* — and then **measures itself to prove it actually works.**

## Why does this matter?
Most beginner projects show a demo where the answer happens to be right and call it done.
This project **measures** how good it is with real numbers. That's the skill that stands out.

## How it works (the flow)
```
A new test failure comes in
        │
   (1) turn it into numbers  ...............  "embedding"
        │
   (2) find the 5 most similar past failures   "retrieval" (Chroma)
        │
   (3) put the new failure + those 5 examples into a message
        │
   (4) ask Claude: "what category? what caused it?"   the AI part
        │
   (5) check Claude's answer has the right shape        "validation"
        │
   show the user: category, cause, similar IDs, confidence
```
Steps 1, 2, 5 are **plain code** (predictable). Step 4 is **the AI** (not predictable).
Knowing which parts are which is the whole point of the project.

## The 4 building blocks
1. **Data** — 50 fake past failures to search through. ✅ *already done* (`data/corpus.json`)
2. **Retrieval** — find similar past failures (Chroma vector database)
3. **Classification** — ask Claude to categorize + explain
4. **Evaluation** — grade each block with real numbers (this is the star of the show)

## The 3 things we measure (the "eval")
- **Retrieval:** did we find the *right* similar failures? → number called `recall@5`
- **Classification:** did Claude pick the *right* category? → number called `F1`
- **Answer quality:** was the explanation actually good? → a checklist score
- Plus: how much does each query **cost** and how **fast** is it?

## The finish line (what "done" looks like)
A public GitHub repo with a README that shows real numbers like:
> *"recall@5 = 72%, F1 = 0.75, quality = 0.81, $0.003 per query."*

## The only rules that matter
1. **Fake data only.** Never use real company data. Ever.
2. **One task per session.** Don't jump ahead. `DAILY_TASKS.md` tells you the next step.
3. **Stuck for 10+ minutes?** Write it in `STUCK_LOG.md` and ask for help. Don't grind alone.

---
*The detailed week-by-week plan lives in `ACTION_PLAN.md` if you ever want it. You don't need it day-to-day.*
