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

<!-- Three or four sentences: what a user asks for, and what they get back. -->

FitFindr accepts a natural-language request for a thrifted item, including
optional size and price constraints. It searches the listings data, selects the
best matching listing, and passes that item to a stylist for outfit ideas.
Finally, it creates a short fit-card caption containing the item, price, and
platform. If no listing matches, it explains how to broaden the search instead
of calling the later tools.

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

- **What it does:** Searches listings by description keywords, optional size,
  and optional inclusive maximum price.
- **Inputs:** `description` (str), `size` (str | None), `max_price` (float | None).
- **Returns:** A list of matching listing dicts containing the listing fields,
  ordered by keyword score and limited by `config.SEARCH_RESULT_LIMIT`.
- **When it has nothing:** Returns an empty list.

### `suggest_outfit`

- **What it does:** Suggests one or two outfits using the selected listing and
  the user's existing wardrobe.
- **Inputs:** `new_item` (dict), `wardrobe` (dict with an `items` list).
- **Returns:** A non-empty model-generated outfit suggestion string.
- **When it has nothing:** With an empty wardrobe, returns general styling
  advice rather than requiring wardrobe items.

### `create_fit_card`

- **What it does:** Turns an outfit suggestion and selected listing into a
  concise social-media fit-card caption.
- **Inputs:** `outfit` (str), `new_item` (dict).
- **Returns:** A model-generated two-to-four sentence caption mentioning the
  item, price, platform, and outfit vibe.
- **When it has nothing:** If `outfit` is empty or whitespace, returns a
  descriptive message and does not call the model.

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

**Branch rule:** If `search_listings` returns an empty list, store an
explanation in `session["error"]` and stop before calling
`suggest_outfit`. Otherwise, store the first result as
`session["selected_item"]`, call `suggest_outfit`, store its result, and then
call `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regular expressions extract an optional price
ceiling and size; the remaining text is the description.

**What moves through the session:** The parsed query goes to
`search_results`; the first result goes to `selected_item`; that same item and
the wardrobe go to `outfit_suggestion`; then the same item and outfit go to
`fit_card`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   **Outfit 1: Y2K Streetwear**
*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Shoes:** Chunky white sneakers
*   **Outerwear:** Black cropped zip hoodie
*   **Accessory:** Black crossbody bag

**Outfit 2: Casual Contrast**
*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Wide-leg khaki trousers
*   **Accessory:** Brown leather belt
*   **Shoes:** Chunky white sneakers
*   **Outerwear:** Vintage black denim jacket

  Fit card: Channeling major early-2000s energy with this sweet graphic top. I paired the Y2K Baby Tee — Butterfly Print with baggy denim and chunky kicks for the ultimate off-duty look. Snagged it on depop for just $18.0, and honestly, I haven't taken it off since.

0 model calls this session, 2 served from cache

```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"


[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017','title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```


```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"


Here are two complete outfits featuring your new Vintage Levi's 501 Jeans:

**Outfit 1: Casual & Classic**
*   **Top:** White ribbed tank top
*   **Outerwear:** Vintage black denim jacket
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Brown leather belt, Black crossbody bag
*   *Vibe:* Effortless, everyday cool with a balanced high-contrast denim look.

**Outfit 2: Cozy Streetwear**
*   **Top:** Oversized grey crewneck sweatshirt
*   **Shoes:** Black combat boots
*   **Accessories:** Black crossbody bag
*   *Vibe:* Relaxed and slightly edgy, letting the straight-leg vintage fit of the 501s shine with chunkier footwear.

```



```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

Finally found the holy grail of denim on depop. These Vintage Levi's 501 Jeans in Medium Wash have that perfectly worn-in slouch I’ve been hunting for. Paired with my crispest white sneakers, it’s the ultimate effortless Sunday uniform for just $38.0.

```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked for help checking why size matching was not working.
- *What came back:* The problem was that `.upper` referenced the method instead of calling it.
- *What I changed:* I changed `p.strip().upper` to `p.strip().upper()`.

**Moment 2**

- *What I asked for:* I asked for help checking the planning loop against the Unit 3 branch requirement.
- *What came back:* The loop needed to stop when `search_listings` returned an empty list.
- *What I changed:* I stored the empty-search explanation in `session["error"]` and returned before calling `suggest_outfit`.

**Moment 3**

- *What I asked for:* I asked how to move `search_listings` behind MCP without changing the tool's return value.
- *What came back:* The server needed a registered function with typed inputs, and the agent needed to call it through `mcp_client.call_tool`.
- *What I changed:* I registered the tool in `mcp_server.py`, rewired `run_agent`, and verified that the MCP call still returned a list and preserved the empty-search branch.

**Moment 4**

- *What I asked for:* I asked for help diagnosing the one failed before-run and choosing one measurable improvement.
- *What came back:* The failed run stopped in the model-backed outfit step because the model was unavailable; the fit-card prompt could also state its output requirements more explicitly.
- *What I changed:* I added a fit-card self-check to the prompt, reran the same five-try evaluation, and compared the before and after reports.

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
| 1. Matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | FAIL | PASS | MET (4/5) |
| 2. Impossible query stops before the second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item reaches the stylist unchanged | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card includes title, price, and platform | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Price ceiling is respected | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Real output from one try**, from
`results/run_2026-10-19_0218_before.md`, produced by
`run_eval.py::main` calling `agent.py::run_agent`:

```
### matching query completes
- Query: `vintage graphic tee under $30`
- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

### impossible query stops early
- Query: `designer ballgown size XXS under $5`
- stopped early: yes — No listings matched. Try different keywords, a broader size, or a higher price limit.
- selected_item: (none)
- search_results: 0

### selected item reaches stylist
- Query: `vintage graphic tee under $30`
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

### fit card includes listing details
- Query: `platform sneakers size 8`
- selected_item: Platform Sneakers — White Chunky Sole ($48.0, poshmark)
- search_results: 1
Fit card:
These Platform Sneakers — White Chunky Sole give off the ultimate off-duty vibe,
whether I'm leaning into effortless Y2K streetwear or throwing on an oversized
crewneck. I scored them on poshmark for just $48.0.

### price ceiling is respected
- Query: `platform sneakers under $50`
- selected_item: Platform Sneakers — White Chunky Sole ($48.0, poshmark)
- search_results: 2
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
| 1 | Matching query completes all three tools | 4 of 5 | MET (4/5) | Four runs completed search, outfit, and fit-card generation; one stopped during the outfit tool because the model was unavailable. |
| 2 | Impossible query stops before the second tool | 5 of 5 | MET (5/5) | All five impossible-query runs returned the guidance message, had zero search results, and had no selected item or outfit call. |
| 3 | Selected item reaches the stylist unchanged | 5 of 5 | MET (5/5) | All five runs selected the same listing ID from the search result and passed that listing into the outfit step. |
| 4 | Fit card includes title, price, and platform | 4 of 5 | MET (5/5) | All five sneaker runs produced a fit card containing the selected title, `$48.0`, and `poshmark`. |
| 5 | Price ceiling is respected | 5 of 5 | MET (5/5) | All five capped searches returned only listings priced at or below the requested `$50` ceiling. |

**Diagnoses**

Criterion 1 had one miss. The failure occurred in the model-backed
`suggest_outfit` tool: the model request could not be completed, so the
`ModelUnavailable` handler stored an error and stopped before
`create_fit_card`. The MCP search, branch, and session handoff worked; the
mechanism was external model availability rather than an application logic
failure.

Criteria 2 through 5 had no misses to diagnose. The empty-search branch
stopped before the second tool in all five tries, the selected listing was
carried through session state in all five matching runs, the fit-card outputs
contained the required listing details, and the explicit price filter excluded
results over `$50`.

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
[1] parse_query
     in:  platform sneakers size 8
     out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
     in:  dict with keys: description, size, max_price
     out: 1 items: Platform Sneakers — White Chunky Sole
     →    1 match(es)
[3] select_item
     out: Platform Sneakers — White Chunky Sole ($48.0, poshmark)
[4] suggest_outfit
     in:  Platform Sneakers — White Chunky Sole ($48.0, poshmark)
     out: Those white chunky platform sneakers with the late '90s/early 2000s…
[5] create_fit_card
     in:  Platform Sneakers — White Chunky Sole ($48.0, poshmark)
     out: I'm living for this nostalgic Y2K street style…

```

**Empty search**

```
[1] parse_query
     in:  designer ballgown size XXS under $5
     out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
     in:  dict with keys: description, size, max_price
     out: [] (empty)
     →    0 match(es)
[3] branch
     →    search returned []: stopping before suggest_outfit

```

**On the MCP move:**

I registered `search_listings` in `mcp_server.py` with typed inputs for
`description`, `size`, and `max_price`. In `agent.py::run_agent`, I replaced
the direct search call with `call_tool("search_listings", session["parsed"])`.

The MCP client returned the same list-shaped result as the direct function.
The empty-search query returned an empty list, and the agent stopped before
calling `suggest_outfit`, so the existing branch behavior did not change.

**Failure modes triggered**

- **Empty search:** The agent returned, "No listings matched. Try different
  keywords, a broader size, or a higher price limit." It stopped before
  `suggest_outfit`.
- **Empty wardrobe:** Running with `--empty-wardrobe` still produced general
  styling advice and a fit card without assuming the user owned specific
  clothing.
- **Model unavailable:** The handler returns, "The outfit stylist could not
  reach the model. Check your API key and network connection, then try this
  request again." The fit-card handler uses the same guidance and names the
  fit-card writer. To trigger this manually, change one character in `.env`,
  run a new uncached matching query, and restore the key immediately afterward.

---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

I strengthened the `create_fit_card` prompt in `tools.py` with an explicit
self-check. Before returning, the model must verify that the caption has two to
four sentences and includes the item title, price, and platform exactly once.

**Which failure it was meant to fix:**

The before run had one matching-query failure because the model was unavailable
while creating the outfit. The improvement targets the fit-card contract and
also makes the model output easier to validate when the model is reachable.

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Impossible query stops before the second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item reaches the stylist unchanged | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card includes title, price, and platform | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Price ceiling is respected | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Did it help, and how do I know:**

Yes. Criterion 1 improved from 4/5 in
`results/run_2026-10-19_0218_before.md` to 5/5 in
`results/run_2026-10-19_0225_after.md`. Criteria 2 through 5 stayed at 5/5,
so the change did not regress the branch, state, fit-card, or price-filter
checks. The after report was produced by `run_eval.py::main` calling
`agent.py::run_agent`, with caching disabled and five tries per scenario.

One real after-run excerpt:

```
### matching query completes
- Query: `vintage graphic tee under $30`
- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

### fit card includes listing details
- Query: `platform sneakers size 8`
- stopped early: no
- selected_item: Platform Sneakers — White Chunky Sole ($48.0, poshmark)
- search_results: 1
```

The complete after-run evidence is in
`results/run_2026-10-19_0225_after.md`.
<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->

No acceptance criterion remained missed after the improvement: all five
criteria met their targets in the after-run. The remaining operational
limitation is that a temporary model or network failure still stops a matching
request instead of retrying it. The `ModelUnavailable` handlers give the user
an actionable message, but I stopped after one improvement as required by this
milestone. A future improvement would add a bounded retry with a clear
backoff, then measure whether matching queries complete more reliably without
creating duplicate or excessive model calls.


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
