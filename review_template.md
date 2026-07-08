# Code Review Notes

Fill this in as you work through the milestones. Each section mirrors the structure of a real GitHub pull request review.

---

## PR #1 — Bulk Purchase (`pr1_bulk_purchase.py`)

### Summary
*What does this PR do? (1–2 sentences in your own words)*

> Adds `POST /lists/<list_id>/purchase-all`, intended to mark every *unpurchased* item in a list as purchased in one call and report how many items were newly purchased.

### Issues

For each issue you find, note: where it is (file + function), what's wrong, and why it matters in production.

**Issue 1**
- Location: `purchase_all_items()`, line 30 — `Item.query.filter_by(list_id=list_id).all()`
- What's wrong: The query has no `is_purchased` filter, so it fetches every item in the list, including ones already purchased. The loop below (lines 31–34) then unconditionally overwrites `purchased_by`/`purchased_at`/`is_purchased` on all of them.
- Why it matters: Confirmed via test — an item already purchased by one user (leo) had its `purchased_by` silently overwritten to the calling user (maya) after `purchase-all` ran. The original attribution is permanently lost after commit; there's no undo. In production this corrupts purchase history for any list with pre-existing purchased items.
- Suggested fix: Filter to `filter_by(list_id=list_id, is_purchased=False)` so only unpurchased items are touched.

**Issue 2**
- Location: `purchase_all_items()`, line 36 — `return len(items)`
- What's wrong: `items` (from line 30) is every item in the list, not just the ones newly purchased by this call. The response field is named `purchased`, implying "items purchased by this request."
- Why it matters: Confirmed via test — calling `purchase-all` on a list with 5 unpurchased + 3 already-purchased items returned `{"purchased": 8}`, not 5. Any caller tracking shopping progress (e.g., a UI showing "you just checked off N items") would display a wrong, inflated number.
- Suggested fix: Once the query is scoped to unpurchased items only (Issue 1's fix), `len(items)` naturally reflects the newly-purchased count.

**Issue 3**
- Location: `routes/lists.py` addition, line 52 — `user_id = data.get("user_id")`, used unchecked at line 54
- What's wrong: No existence check on `user_id` before it's passed into `purchase_all_items()`, unlike the existing `mark_purchased` route which explicitly validates and 400s on a missing `user_id`.
- Why it matters: Confirmed via test — POSTing `{}` (no `user_id`) still returned `200 OK` with `{"purchased": 8}` instead of an error. Based on the code, every item's `purchased_by` gets set to `None`, silently destroying attribution for the entire list with no error surfaced to the caller.
- Suggested fix: Mirror the existing `mark_purchased` route pattern — return 400 with an error message if `user_id` is missing, before calling the service function.

### Questions for the Author
*Things you're uncertain about — design choices that could be intentional or bugs depending on intent.*

>

### Verdict
- [ ] Approve — ship it
- [ ] Request Changes — needs fixes before merging
- [ ] Comment — needs discussion before a verdict

**Rationale** *(1–2 sentences)*:

>

---

## PR #2 — List Stats (`pr2_list_stats.py`)

### Summary
*What does this PR do? (1–2 sentences in your own words)*

>

### Issues

**Issue 1**
- Location:
- What's wrong:
- Why it matters:
- Suggested fix:

**Issue 2**
- Location:
- What's wrong:
- Why it matters:
- Suggested fix:

**Issue 3** *(if found)*
- Location:
- What's wrong:
- Why it matters:
- Suggested fix:

### Questions for the Author
*A good code review often surfaces design questions, not just bugs. What would you want to clarify before approving?*

>

### Verdict
- [ ] Approve — ship it
- [ ] Request Changes — needs fixes before merging
- [ ] Comment — needs discussion before a verdict

**Rationale** *(1–2 sentences)*:

>

---

## Reflection

*Answer after completing both reviews.*

**1.** Which issue was hardest to spot, and why?

>

**2.** Which issues do you think an LLM reviewer (like Claude reviewing its own code) would most likely miss? Why?

>

**3.** One thing you'd add to a code review checklist for AI-generated backend code:

>
