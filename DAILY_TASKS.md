# Daily Tasks — One Session at a Time

> **How to use this file:** Find the first session that is not checked off. Do only that one.
> Ignore everything below it. Come back tomorrow for the next.
>
> Each session is ~30–90 min. Small on purpose. Finishing one session = a win.
> When you finish, tick its box `[x]`, then `git commit`.

**Legend:** ⬜ = todo  ✅ = done   👉 = do this one next

---

## ✅ Session 0 — The data (ALREADY DONE)
The 50 fake test failures already exist in `data/corpus.json`. Nothing to do here.
This is the "library" of past failures the tool will search through.

---

## 👉 Session 1 — Set up your workspace  (~45 min)
**Goal:** Get Python ready so code can run later. No AI yet — just plumbing.

**Steps:**
1. Open a terminal in this project folder.
2. Make a virtual environment (an isolated space for this project's packages):
   ```
   python3 -m venv .venv
   source .venv/bin/activate
   ```
   (You'll see `(.venv)` appear at the start of your terminal line. That means it worked.)
3. Create a file `requirements.txt` with these lines:
   ```
   chromadb
   sentence-transformers
   anthropic
   pydantic
   pytest
   python-dotenv
   ```
4. Install them:
   ```
   pip install -r requirements.txt
   ```
   (This takes a few minutes. Normal.)

**Done when:** `pip list` shows those packages installed. ✅ tick the box, commit.

> ❓ Don't worry about what each package does yet. You'll meet them one at a time.

---

## ⬜ Session 2 — Get your Claude API key working  (~30 min)
**Goal:** Prove you can talk to Claude from Python.

**Steps:**
1. Get an Anthropic API key (from console.anthropic.com).
2. Create a file called `.env` with one line (this file is secret, already git-ignored):
   ```
   ANTHROPIC_API_KEY=your-key-here
   ```
3. Also make `.env.example` with a fake placeholder (this one is safe to commit):
   ```
   ANTHROPIC_API_KEY=sk-put-your-key-here
   ```
4. Write a tiny throwaway file `scratch.py` that sends Claude one message ("say hello") and prints the reply. Run it.

**Done when:** Claude replies in your terminal. Delete `scratch.py` after. ✅ commit.

> 🎯 This is your first time calling an AI from code. That's the milestone today.

---

## ⬜ Session 3 — Build the answer key (ground truth)  (~60 min)
**Goal:** Write down the "correct answers" so we can grade the tool later.

**Why:** You can't measure if something is right without knowing the right answer first.

**Steps:**
1. Create `data/ground_truth.json`.
2. For ~15–20 of the failures in `corpus.json`, write down:
   - which category it *should* be (you already know — it's in the file)
   - which 1–2 other failures are *similar* to it
3. Ask me to draft this for you first (the corpus already has hints) — then you just check it.

**Done when:** `data/ground_truth.json` exists with ~15+ labeled entries. ✅ commit.

> 🛑 This is boring but it's the most important file. The whole "measuring" idea depends on it.

---

## ⬜ Session 4 — Turn text into numbers (embeddings)  (~60 min)
**Goal:** Write code that converts a failure's text into a list of numbers.

**Steps:**
1. Create `src/embed.py` with one function: `embed(text)` → returns a list of numbers.
2. Use the `sentence-transformers` package (one line to load the model).
3. Test it: embed two *similar* sentences and two *different* ones. Similar ones should have closer numbers.

**Done when:** `embed("hello")` prints a list of numbers. ✅ commit.

---

## ⬜ Session 5 — Store the failures (the vector database)  (~60 min)
**Goal:** Load all 50 failures into Chroma so we can search them.

**Steps:**
1. Create `src/index.py`.
2. Load `corpus.json`, embed each failure, store it in Chroma (saves to a local folder).

**Done when:** Chroma reports 50 items stored. ✅ commit.

---

## ⬜ Session 6 — Find similar failures (retrieval)  (~60 min)
**Goal:** Given a new failure, get the 5 most similar past ones.

**Steps:**
1. Create `src/retrieve.py` with `retrieve(failure, k=5)`.
2. Try it on 2–3 failures and eyeball: do the results *look* similar?

**Done when:** it returns 5 IDs that seem related. ✅ commit.

---

## ⬜ Session 7 — Your FIRST real number (retrieval eval)  (~90 min)
**Goal:** Measure how good retrieval is, using your answer key.

**Steps:**
1. Create `evals/eval_retrieval.py`.
2. For each failure in `ground_truth.json`, retrieve top-5, check how many *expected* similar ones showed up.
3. Print the score (`recall@5`).

**Done when:** you see a real number like `recall@5 = 0.68`. 🎉 **Big milestone.** ✅ commit.

---

## ⬜ Session 8+ — The AI answer + grading it
*(We'll expand these into small sessions when you get here — classification, then grading Claude's category, then grading the explanation quality, then the README.)*
Don't read ahead. Sessions 1–7 first.

---

## 📌 Where am I right now?
- **Next task:** Session 1 (set up workspace).
- **After each session:** tick the box, run `git add -A && git commit -m "session N done"`.
- **Stuck 10+ min?** Log it in `STUCK_LOG.md` and ask me. That's the rule.
