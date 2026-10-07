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

FitFindr takes a natural language query for a clothing item (e.g., "vintage graphic tee under $30"), searches a thrift store dataset, and selects the best matching piece. If an item is found, the agent considers the user's existing wardrobe to suggest how to style the new item. Finally, it generates an engaging, platform-specific social media caption showcasing the thrifted item and the outfit idea.
---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Searches the listings data for items matching keywords and optionally a size and max price limit. A size match requires the exact size string to be present as an isolated word (case-insensitive) in the listing's size field, not just as a substring.
- **Inputs:** `description` (str), `size` (str or None), `max_price` (float or None)
- **Returns:** A list of listing dicts (each containing id, title, description, category, style_tags, size, condition, price, colors, brand, platform), sorted by best keyword match first.
- **When it has nothing:** Returns an empty list `[]`.

### `suggest_outfit`

- **What it does:** Suggests one or two outfits combining a new thrifted item with pieces from the user's wardrobe.
- **Inputs:** `new_item` (dict), `wardrobe` (dict containing an 'items' list)
- **Returns:** A non-empty string with outfit suggestions formatted by the model.
- **When it has nothing:** If the wardrobe is empty, it returns general styling advice for the item as a string.

### `create_fit_card`

- **What it does:** Writes a short, 2-4 sentence caption for a social media post about the new item and outfit.
- **Inputs:** `outfit` (str), `new_item` (dict)
- **Returns:** A string containing the social media caption, mentioning the item, price, and platform.
- **When it has nothing:** If `outfit` is empty or whitespace, it returns a descriptive fallback message string.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If search_listings returns an empty list, put a message in the session and stop. Otherwise, take the first result and go to suggest_outfit.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Asking the model to extract the `description`, `size`, and `max_price` using `generate()`.

**What moves through the session:** `query` -> `parsed` (dict) -> `search_results` (list) -> `selected_item` (dict) -> `outfit_suggestion` (str) -> `fit_card` (str).

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'
=== A query the data can match ===
  found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop
  outfit:   Here are 2 outfit ideas using your new Y2K butterfly baby tee and pieces already in your wardrobe:

### Outfit 1: 2000s Streetwear Contrast
*Balance the fitted, girly butterfly tee with relaxed denim and chunky footwear for a classic Y2K off-duty look.*

* **Top:** Y2K Butterfly Baby Tee (lst_002)
* **Bottoms:** Baggy straight-leg jeans, dark wash (w_001)
* **Outerwear:** Vintage black denim jacket (w_006)
* **Shoes:** Chunky white sneakers (w_007)
* **Accessories:** Black crossbody bag (w_010)

  fit card: Channeling major 2000s off-duty model energy with this Y2K butterfly baby tee! 🦋 Pair it with baggy denim and chunky sneakers. Grab this absolute steal for just $18.0 over on my Depop before it's gone! ✨
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"
[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', ...}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', ...}, ...]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Great find! Vintage Levi’s 501s are an absolute staple. Since you already own some great basics and streetwear pieces, here are two ways to style your new medium-wash jeans using items straight from your wardrobe:
### Outfit 1: The Classic Off-Duty Look (Casual & Streetwear)
*Lean into the vintage streetwear vibe with crisp white basics and chunky sneakers.*
* **Top:** White ribbed tank top (`w_003`)
* **Layer (Optional):** Oversized grey crewneck sweatshirt (`w_004`) worn over the shoulders or thrown on if it gets chilly
* **Shoes:** Chunky white sneakers (`w_007`)
...
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Found the holy grail of denim: these perfectly worn-in Vintage Levi's 501s in a dream medium wash. Just throw them on with your favorite crisp white sneakers for that effortlessly cool 90s off-duty look. Grab them on my Depop right now for just $38!
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked my AI assistant to help me refine my size filtering logic in `search_listings`.
- *What came back:* It suggested a `substring` check like `size.lower() in item.get('size').lower()`, but simultaneously noted that it would incorrectly match a requested size "S" inside "US 9" or "XS".
- *What I changed:* I bypassed substring checking entirely and wrote a custom tokenizer using Python's `re.split` to extract distinct alphanumeric size tokens, guaranteeing that sizes are matched precisely as whole words.

**Moment 2**

- *What I asked for:* I requested help writing a generative JSON parser prompt to extract the `description`, `size`, and `max_price` within the `agent.py` planning loop.
- *What came back:* The AI wrote a clear generative prompt and a `json.loads` block, but failed to include fallback logic in case the AI generated text surrounding the JSON block or the schema crashed.
- *What I changed:* I added custom Python code to aggressively strip markdown backticks (` ```json `) from the model's output and included an `except` block to supply a safe default fallback containing the raw search query string if parsing failed.

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
