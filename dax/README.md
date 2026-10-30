# DAX

Measure definitions for the report. See [`measures.dax`](measures.dax).

## Status

| Measure | Bindings | Pages | Status |
|---|---|---|---|
| `[Welspun Products]` | 44 | 6 | ✅ Documented, no change needed |
| `[Non-Welspun Products]` | 24 | 7 | 🔧 Corrected version written — **not yet applied to the `.pbix`** |
| `[Avg_Welspun_Product]` | 41 | 7 | ❌ Expression not recovered |

The `.pbix` in this repository was downloaded from the Power BI Service, so its semantic model is
stored compiled. Expressions cannot be read out of or written into the file — they have to be
transcribed from, and edited in, Power BI Desktop.

---

## Applying the `[Non-Welspun Products]` fix

**Do not skip step 1.** The reconciliation result decides whether the fix changes the numbers
already published in this repository.

### Step 1 — reconcile

1. Open the `.pbix` in Power BI Desktop.
2. **Home → New measure.** Paste `NW Explicit` from [`measures.dax`](measures.dax). Repeat for
   `Unclassified Check` and `Blank Brand Rows`.
3. Drop a **Card** visual on any page and put `Unclassified Check` on it.
4. Clear all slicers so the card reads across the full dataset.

| Result | Meaning | What to do |
|---|---|---|
| `0` | The residual was behaving. No displayed figure will change. | Proceed to step 2. |
| Anything else | That many product counts are currently inflating the competitor figure. | Proceed to step 2, then **re-screenshot the report and update every figure in the README** — the 22.40% competitor share and all percentages derived from it will move. |

Also read `Blank Brand Rows`. A non-zero result means `programdetail` holds rows with no brand
classification, which is a data-quality item for the client regardless of the measure fix.

### Step 2 — apply

1. **Model view** → select `[Non-Welspun Products]` in the Data pane.
2. Select the whole expression in the formula bar.
3. Paste the corrected definition from [`measures.dax`](measures.dax) and press Enter.
4. Delete the three step-1 measures.
5. **Save**, then **Publish** to the workspace.

### Step 3 — verify

Open each of the ten pages and confirm the three category shares still sum to 100%. If
`Unclassified Check` returned non-zero, they will sum to 100% at different values than before —
that is the fix working, not a new fault.

---

## Outstanding

`[Avg_Welspun_Product]` still needs transcribing, and it is a higher priority than the fix above.
It is the second most-bound measure in the report and is displayed under non-Welspun labels in all
41 of its bindings, despite its name. Until it is read, it is not knowable whether the competitor
figures are correct at all.

To retrieve it: Power BI Desktop → **Model view** → select `Avg_Welspun_Product` in the Data pane
→ copy the formula bar → paste into [`measures.dax`](measures.dax) and
[`../docs/04-measures.md`](../docs/04-measures.md).

Full analysis: [`../docs/04-measures.md`](../docs/04-measures.md)
