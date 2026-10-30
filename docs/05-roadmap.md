# Roadmap — next iteration

[← Back to README](../README.md)

Items identified by auditing the report definition against its intent. Listed in priority order.

---

## P0 — Identify `[Avg_Welspun_Product]` and resolve the display-name collision

**What:** the second most-bound measure in the report (41 bindings, 7 pages) is named
`Avg_Welspun_Product` but is displayed under non-Welspun labels in all 41 cases. It also shares
three display names with `[Non-Welspun Products]`, so two different calculations appear under
identical labels.

**Why it is P0:** every other item on this list is cosmetic or localised. This one determines
whether the report's competitor figures are correct at all. Until the expression is read, it is
not knowable whether the 22.40% competitor share is right.

**Fix:** read the expression in Power BI Desktop, then either correct the labels (if the measure
computes a client average) or rename the measure (if it computes a competitor figure). Full
detail in [04-measures.md](04-measures.md).

---

## P1 — Correct the field bindings on the Unbranded pages

**What:** Several visuals on *Unbranded Region Details* and *Unbranded Store Details* are bound
to competitor measures rather than unbranded ones.

**Where:**

| Page | Visual | Currently bound to | Should be |
|---|---|---|---|
| Unbranded Region Details | Detail table | `% / # Non-Welspun Products` | Unbranded |
| Unbranded Region Details | Area bar chart | `% Non- Welspun Products` | `% Unbranded` |
| Unbranded Region Details | Store Type donut | `Non-Welspun Products` | Unbranded |
| Unbranded Region Details | Store Name column | `% Non-Welspun Products` | `% Unbranded` |
| Unbranded Store Details | Region table | `% / # Non-Welspun Products` | Unbranded |
| Unbranded Store Details | Area bar chart | `% Non- Welspun Products` | `% Unbranded` |
| Unbranded Store Details | Store Type donut | `Non-Welspun Products` | Unbranded |
| Unbranded Store Details | Store City table | `% / # Non-Welspun Products` | Unbranded |

**Cause:** the Unbranded pages were built by duplicating the Non-Welspun pages — the correct
approach, given the deliberate layout symmetry — but the field bindings were not fully
re-pointed afterwards.

**Why it matters:** the correctly-bound visuals on those pages (the pincode and store-city
matrices, the zone donut and pie, the region and area charts using `% Unbranded`) will disagree
with the mis-bound ones on the same canvas. A user comparing two visuals side by side sees two
different numbers for what the titles claim is the same thing.

**Fix:** re-point the field wells on the eight visuals above, then verify each page by checking
that the three category shares on it sum to 100%.

---

## P2 — Consolidate measure naming

**What:** display names are inconsistent across the report, and in three cases the *same* name
covers *two different measures*.

**The collisions** (resolve these with P0):

| Display name | `[Non-Welspun Products]` | `[Avg_Welspun_Product]` |
|---|---|---|
| `Non-Welspun Products` | 13 | 9 |
| `% Non-Welspun Products` | 5 | 8 |
| `# Non-Welspun Products` | 5 | 4 |

**The cosmetic variants:**

| Variant | Note |
|---|---|
| `% Non- Welspun Products` | extra space — used only by `[Avg_Welspun_Product]` |
| `# Non- Welspun Products` | extra space — used only by `[Avg_Welspun_Product]` |
| `Percentage of Welspun Products` | same as `% Welspun Products` |
| `Occupancy Percentage` | `[Welspun Products]` in table visuals |
| `Products Count` | `[Welspun Products]` again |
| `Welspun Products1` | auto-suffixed duplicate |
| ` Welspun Products` | leading space |

**Why it matters:** the collisions mean a user comparing two visuals with the same title may be
reading two different numbers. The variants mean changing a calculation requires hunting every
spelling of it across ten pages.

**Fix:** one calculation, one name. Apply consistently, then verify no two measures share a
display name anywhere in the report.

---

## P1b — Replace the `[Non-Welspun Products]` residual

**What:** `[Non-Welspun Products]` is defined as `SUM(programdetail[count]) - [Welspun Products]`
— everything not flagged as the client's — rather than as an explicit competitor definition.

**Why it matters:** blanks, join failures and any unbranded rows in `programdetail` are swept into
the competitor figure silently. The last case double-counts against the separate unbranded series.

**Status:** corrected definition written and ready to paste; **not applied**, because the `.pbix`
stores its model compiled and the change must be made in Power BI Desktop.

**Sequencing:** run the reconciliation check first. If it returns non-zero, applying the fix
changes published figures and this repository's screenshots and numbers need regenerating in the
same change.

Procedure: [`../dax/README.md`](../dax/README.md) · Analysis: [04-measures.md](04-measures.md)

---

## P3 — Make the Unbranded measure explicit

**What:** the unbranded series is an implicit `Sum(storevisitoutput[Unbranded])` rather than a
named measure.

**Why it matters:** the other two categories are explicit measures. Mixing an implicit
aggregation into a three-way comparison means the unbranded series cannot carry the same
formatting, description or future logic as its two counterparts — and it is the largest category
in the report.

**Fix:** create `[Unbranded Products]` as an explicit measure and re-point the bindings.

---

## P4 — Suppress low-sample dates on the trend chart

**What:** the *Share of Shelf Occupancy Trend* chart is unreadable in its early period because
days with one or two store visits swing the percentage between 0% and 100%.

**Why it matters:** the chart currently relies on the user moving the date slider to exclude the
noisy window. A reader who does not know to do that sees volatility that is a sampling artefact,
not a business signal.

**Fix:** add a minimum-visit-count threshold, or a visual-level filter on visit count, so days
below the threshold are suppressed rather than plotted.

---

## P5 — Resolve the `root` user bucket

**What:** the `root` account contributes 6.26% of all captured images on the *User Analysis*
page.

**Why it matters:** `root` is an administrative or system account with no owner in the field
hierarchy. Images entering through it cannot be attributed to a region, area or store team, which
weakens any store-level completeness claim.

**Fix:** a client-side question first — what writes through `root`? — then either map those
records to real users or exclude them with a documented note.

---

## P6 — Review duplicated visuals

**What:** several pages carry visually identical pairs (two matching donuts on *Product Wise*,
two matching column charts on *Non-Welspun Store & Brand Details*, two area charts on *Store
Details*).

These may be intentional — seven bookmarks are defined, and stacked visuals toggled by bookmark
is a standard pattern. If so, no change is needed beyond documenting it. If not, they are
leftovers from layout iteration and should be removed.

---

## P7 — Document the DAX

See [04-measures.md](04-measures.md). The measure expressions need transcribing from Power BI
Desktop, since the Service-downloaded `.pbix` does not expose them.

---

[← Measures](04-measures.md) · [Back to README](../README.md)
