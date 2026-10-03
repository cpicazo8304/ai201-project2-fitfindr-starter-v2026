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
| 1. matching query completes | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET |
| 2. impossible query stops early | 5 of 5 | PASS | PASS |  PASS |  PASS | PASS | MET |
| 3. state survives the handoff | 5 of 5 | PASS | PASS |  PASS |  PASS | PASS | MET |
| 4. fit card names price and platform | 5 of 5 | FAIL | FAIL | FAIL | FAIL | FAIL | MISS |
| 5. empty wardrobe still produces advice | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET |

**Real output from one try**:

**Criterion 2**, from `agent.py::run_agent` via the log:

```
- stopped early: yes — Nothing in the listings matched description 'designer ballgown',
  size XXS, under $5. Things to change: try broader words …
- selected_item: (none)
- search_results: 0
```

**Criterion 3**, the two trace lines it compares, from `trace.py::step`:

```
[3] select_item
      out: Shacket — Olive Canvas ($33.0, poshmark)
[4] suggest_outfit
      in:  Shacket — Olive Canvas ($33.0, poshmark)
```

**Criterion 4** — all five cards for the same item, from `tools.py::create_fit_card`:

```
Just scored this absolute dream of a butterfly baby tee on depop for only 18.0 and I am already obsessing over it. I am planning to lean all the way into the Y2K nostalgia with baggy denim and chunky sneakers, or grunge it down a bit with wide-leg khakis and my favorite heavy boots. Honestly, it is the ultimate little top for throwing on when you literally have nothing to wear.
```

**Criterion 5**, from `tools.py::suggest_outfit` with an empty wardrobe:

```
These are general ideas since you don't have an established wardrobe yet.

Look one leans casual streetwear. Pair the cropped light wash denim jacket with black high-waisted wide-leg cargo pants, a plain white ribbed tank top, and white leather Reebok Club C sneakers. 

Look two leans effortless everyday. Layer the jacket over a black ribbed cotton midi dress, and finish the outfit with well-worn Converse Chuck Taylor high-top sneakers and a black canvas tote bag.
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

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Matching query completes | MET (5/5) | Target 4 of 5; every try produced a card. Nothing close about it. |
| 2 | Impossible query stops early | MET (5/5) | All five stopped with `search_results: 0` and no fit card. Deterministic, as predicted. |
| 3 | Item found is item passed on | MET (5/5) | Both halves held on all five — session id matched, and the trace lines are identical strings. |
| 4 | Card names price and platform | **MISSED (0/5)** | Zero of five made it aware that the price was the price and the platform was the platform. Although others could infer from the surrounding words, it is better to be safe. Having a dollar sign for the price or a capitalized platform could help.|
| 5 | Empty wardrobe produces advice | MET (5/5) | 414–490 characters, all five opened by saying the ideas were general, all five reached the fit card. |


**The revision to criterion 4.** I wrote *"names the item's price"*, but I should make the target to have the price in digits and with the dollar symbol. Also, for the platform, it should be capitalized 
to emphasize it. The revision made the miss legible, not smaller; under the generous reading the criterion was 5/5 and I'd have learned nothing.


### Diagnosis — criterion 4

**Step: `create_fit_card`. Not the tool, the prompt.**

The information was never missing. `_describe_item` puts `$18.0 on depop` into
the prompt on every call, and the platform came back correctly all five times
from that same line. So this isn't retrieval, it isn't state, and it isn't the
branch — the model had the number and chose how to write it.

The problem was that my prompt didn't explain the specifics of how to include the
price and the platform. If I wanted to make the price and platform obvious, I would
have to include dollar signs and capitalization. The model did what it was told; 
what it was told was ambiguous in the same way my criterion was.

**The pattern worth naming:** the only criterion that missed is the only one
whose pass condition depends on the *wording* a model chose rather than on a
value my code controls. Criteria 1, 2, 3 and 5 all check things determined by
Python — a branch, an assignment, a length. That's not luck. It's a warning
about where to put criteria, and it cuts both ways: the four safe ones taught
me nothing.



---

## Loop Trace

**Happy path**

```
$python app.py ask 'Platform Mary Janes under $55' --trace

[1] parse_query
      in:  Platform Mary Janes under $55
      out: dict with keys: description, size, max_price
[2] search_listings
      in:  dict with keys: description, size, max_price
      out: 2 items: Platform Mary Janes — Black Patent, Platform Sneakers — White Chunky Sole
      →    Found 2 matches
[3] select_item
      out: Platform Mary Janes — Black Patent ($55.0, depop)
[4] suggest_outfit
      in:  Platform Mary Janes — Black Patent ($55.0, depop)
      out: Outfit 1: - Black cropped athletic top - Dark blue baggy denim bottoms - Black vintage classic denim outerwear…
      →    10 wardrobe item(s)
[5] create_fit_card
      in:  Platform Mary Janes — Black Patent ($55.0, depop)
      out: Finally scored these Demonia patent platforms on depop for $55.0 and I am already obsessed with them. They add…

  Found:    Platform Mary Janes — Black Patent — $55.0 on depop
```

**Empty search**

```
$python app.py ask 'nfl jersey' --trace
[1] parse_query
      in:  nfl jersey
      out: dict with keys: description, size, max_price
[2] search_listings
      in:  dict with keys: description, size, max_price
      out: [] (empty)
      →    Found 0 matches
[3] branch
      →    search returned []: stopping before suggest_outfit

  Nothing in the listings matched description 'nfl jersey'.
Things to change: try broader words — 'jacket' finds more than 'cropped corduroy jacket'.
```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

**What I changed:** Added details to price and platform in prompt for `create_fit_card`. 

``
...
f"-Price (written in digits with $, not spelled out in words): {new_item.get('price', 'N/A')}",
f"-Platform (first letter capitalized like a title): {new_item.get('platform', 'N/A')}",
...
```

**Which failure it was meant to fix:** This was meant to fix the specifics for criterion 4 of having the price in the format "${digits}" and the platform capitalized like a title. 

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. matching query completes | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET |
| 2. impossible query stops early | 5 of 5 | PASS | PASS |  PASS |  PASS | PASS | MET |
| 3. state survives the handoff | 5 of 5 | PASS | PASS |  PASS |  PASS | PASS | MET |
| 4. fit card names price and platform | 5 of 5 | FAIL | FAIL | FAIL | PASS | FAIL | MISS |
| 5. empty wardrobe still produces advice | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET |

**Before:**
```
Just scored this absolute dream of a butterfly baby tee on depop for only 18.0 and I am already obsessing over it. I am planning to lean all the way into the Y2K nostalgia with baggy denim and chunky sneakers, or grunge it down a bit with wide-leg khakis and my favorite heavy boots. Honestly, it is the ultimate little top for throwing on when you literally have nothing to wear.
```

**After:**

```
Scored this little butterfly tee on depop for just $18 and I am obsessed with the nostalgic pink and purple print. I am definitely styling it with baggy indigo denim and chunky sneakers for daytime, then swapping into khaki trousers and grunge boots when I want an edgier look.
```

**Did it help, and how do I know:**

It did help with the price by adding the dollar sign, but didn't help with the platform being capitalized with the first letter ("Depop" vs. "depop").

---

## What's Still Broken

The capitalization in the first letter of the platform's name doesn't happen. It is a small issue. But, for a model that might not be as sophisticated as others, it could need an example in the prompt to help it get right. 



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [x] criteria.md has five numbered criteria, each with a target
       [x] Each criterion has a reason underneath it
       [x] All five unit 3 sections above have real content
       [x] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [x] Planning Loop names the branch rule and agent.py::run_agent
       [x] Sample Run: one full query plus the three per-tool tests, as text
       [x] At least four new commits
       [x] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [x] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [x] Run Log — Before, five criteria, five tries each
       [x] Real output pasted underneath, naming file and function
       [x] A verdict on every criterion
       [x] A diagnosis for every miss, naming a place AND a mechanism
       [x] Loop Trace, with the MCP call visible in it
       [x] All three failure modes triggered and handled
       [x] One improvement, with Run Log — After in the same format
       [x] What's Still Broken
       [x] At least four new commits
       [x] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
