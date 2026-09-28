# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

You tell FitFindr what you're after — "vintage graphic tee under $30", "corduroy jacket size L under $45" — and it searches 40 second-hand listings, picks the best match, works out how it would go with clothes you already own, and writes the caption you'd post about the find. It's a command line tool: python app.py ask 'your query'.

When nothing matches, it stops and tells you which of the three things you control — the words, the size, the price ceiling — to change. It doesn't hand an empty result to the next tool and hope.


---

## Tool Inventory

### `search_listings`

- **What it does:** Searches the 40-listing catalogue and returns matches, best first.
- **Inputs:** 
| Input | Type | Notes |
|---|---|---|
| `description` | `str` | Free text. Matched as keywords against title, description, category, brand, style tags and colours |
| `size` | `str \| None` | Optional. `None` skips size filtering entirely |
| `max_price` | `float \| None` | Optional, in dollars, **inclusive** |
- **Returns:**  `list[dict]`. Each dict has `id`, `title`, `description`,
`category`, `style_tags` (list), `size`, `condition`, `price` (float), `colors`
(list), `brand` (`str` or `None`), `platform`. Ordered by keyword-overlap score
descending, then by price ascending
- **When it has nothing:** returns `[]`. Not `None`, not an exception. A branch would be needed in this situation. If search_listings returns an empty list, put a message in the session and stop.

### `suggest_outfit`

- **What it does:** Suggest an outfit or two given the wardrobe and the new item that was thrifted.
- **Inputs:**
| Input | Type | Notes |
|---|---|---|
| `new_item` | `dict` | a listing dict — the item the user is considering.|
| `wardrobe` | `dict` | a wardrobe dict with an 'items' key holding a list of items.
**It may be empty.** |
- **Returns:** A non-empty string with outfit suggestions.
- **When it has nothing:** an empty wardrobe is not an error. It returns general
advice describing pieces generically, opening with a line saying the ideas are
general because no wardrobe is saved. It never returns `""`.

### `create_fit_card`

- **What it does:** Write a short caption someone would actually post about the find.
- **Inputs:**
| Input | Type | Notes |
|---|---|---|
| `outfit` | `str` | The suggestion string from `suggest_outfit` |
| `new_item` | `dict` | The listing dict |
- **Returns:** A two-to-four sentence caption.
If `outfit` is empty or whitespace, return a descriptive message rather
than raising.
- **When it has nothing:** If `outfit` is empty or whitespace, return a descriptive message rather
than raising.


---

## Planning Loop

**Branch rule:**If `search_listings` returns an empty list, put a message in `session["error"]` naming what the user could change, and return the session without calling `suggest_outfit`. Otherwise take the first result, put it in `session["selected_item"]`, and continue.

**Where it lives:** `agent.py::run_agent` (the empty-case message is built by `agent.py::_nothing_found_message`.)

**How the query is parsed:** parsing is regex, in `agent.py::parse_query`, not a model call. Three reasons: it's free, it returns the same answer twice (criterion 3 depends on that), and when it's wrong the reason is readable in the pattern instead of being a model's opinion. 

**What moves through the session:** Everything goes through the session. `search_listings` writes to `session["search_results"]`; the next step reads `session["search_results"][0]` back out and writes `session["selected_item"]`; suggest_outfit is called with that.

---

## Sample Run

**One full query**

```
$ python app.py ask 'corduroy jacket size L under $45'
[1] parse_query
      in:  corduroy jacket size L under $45
      out: dict with keys: description, size, max_price
[2] search_listings
      in:  dict with keys: description, size, max_price
      out: 1 items: Shacket — Olive Canvas
      →    Found 1 matches
[3] select_item
      out: Shacket — Olive Canvas ($33.0, poshmark)
[4] suggest_outfit
      in:  Shacket — Olive Canvas ($33.0, poshmark)
      out: Outfit 1: - Olive canvas shacket - White fitted basics top - Dark blue baggy denim bottoms - White chunky stre…
      →    10 wardrobe item(s)
[5] create_fit_card
      in:  Shacket — Olive Canvas ($33.0, poshmark)
      out: Scored this olive canvas shacket on poshmark for just 33.0 and it is seriously the ultimate transitional layer…

  Found:    Shacket — Olive Canvas — $33.0 on poshmark

  Outfit:   Outfit 1:
- Olive canvas shacket
- White fitted basics top
- Dark blue baggy denim bottoms
- White chunky streetwear sneakers
- Black minimal everyday accessories

Outfit 2:
- Olive canvas shacket
- Grey charcoal oversized cozy top
- Khaki tan minimal wide-leg bottoms
- Black grunge classic boots
- Brown classic earth tones accessories

  Fit card: Scored this olive canvas shacket on poshmark for just 33.0 and it is seriously the ultimate transitional layer. I've already planned two completely different ways to wear it, from crisp white basics and baggy denim to cozy charcoal and chunky boots. It has that perfect heavy-duty feel without being too warm for everyday running around.

2 model calls this session, 754 prompt + 146 output tokens
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(len(search_listings('graphic tee', max_price=30)))"
6
```


```
$ python -c "from tools import search_listings; print(search_listings('designer ballgown', 'XXS', 5))"
[]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import load_listings, get_example_wardrobe; print(suggest_outfit(load_listings()[0], get_example_wardrobe())[:])"
Outfit 1:
- Vintage Levi's 501 Jeans
- White fitted t-shirt
- Black denim jacket
- White chunky sneakers
- Black minimal belt

Outfit 2:
- Vintage Levi's 501 Jeans
- Grey oversized hoodie
- Black grunge boots
- Brown classic belt

```
```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('', load_listings()[0]))"
No fit card — create_fit_card was called with no outfit suggestion, so there was nothing to write about. Check that suggest_outfit returned something before this step.
```
---

## How I Used AI

**Moment 1**

- *What I asked for:* I asked for explanations of regular expressions I can use for the tool calls in short and simple terms.
- *What came back:* It gave me short explanations but was too general sometimes.
- *What I changed:*I made it specific to certain parts of functions and files.

**Moment 2**

- *What I asked for:* I asked for the explanation of the agent.py file in short terms.
- *What came back:* It explained in bullet points what the file contains and did and needed to implement. 
- *What I changed:* I didn't make any changes.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
