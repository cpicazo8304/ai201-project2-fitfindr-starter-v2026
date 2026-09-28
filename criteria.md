# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
If search_listings doesn't find nothing, then that will be covered in criterion 2. This criteria covers when search_listings finds items. So, the way this will fail is through the next two functions, which use generation calls. So, this criteria checks that calls finish and a fit card is returned, which can sometimes fail. Hence, it is 4 of 5.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
Should be 5 of 5 because the branch is on an if on empty list returned in search_listings. Also, there is not randomness in search_listings. It gets a fixed list of items in a wardrobe and finds matches. If the query is impossible, it should return an empty list. If it is not 5 of 5, then something is wrong with search_listings or equivalent.

---

## 3.  The item search found is the item the next tool received

For 5 of 5 tries, the id in session["selected_item"] equals the id of the first entry in session["search_results"], and the item title printed on the trace's suggest_outfit input line is that same item's title.

**Why this target:**

The first part shows that the session was built right (has to be 5 of 5). The second part helps show that the generation keeps track of the new item (which also has to be 5 of 5). There should be no variance since it is deterministic. 

---

## 4. The fit card names the price and the platform

For 5 of 5, the fit card should mention the price and platform of the item. 


**Why this target:**
The fit card function is required to name the price and platform of the item. This is because we want someone else to read the fit card and find the item and maybe do the outfit themselves. Having the price and platform helps them find the item and actually decide if they want it (the price). So, we need 5 of 5 for this criteria.


---

## 5. General Styling given for an empty wardrobe.

For 5 of 5, an empty wardrobe in suggest_outfit still produces a result that gives general styling advice for the thrifted item of at least 150 characters. It should also still reach create_fit_card. Also, the suggestion should contain a phrase talking about how it is general advice and not based on an existing wardrobe.


**Why this target:**
suggest_outfit shouldn't end on "" or something vague. It should still give good enough general advice that can lead to a good fit card. 150 characters helps prevents this. Also, this helps in the new user case where they don't have a wardrobe, so we don't want to treat this as an edge case but something normal.


---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
