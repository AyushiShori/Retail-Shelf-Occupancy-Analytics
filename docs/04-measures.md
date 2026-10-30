# Measures (DAX)

[← Back to README](../README.md)

Three explicit measures carry the brand comparison. All three live on the `storevisit` table but
aggregate `programdetail[count]`, with `brand[is_welspun]` as the classifier.

| Measure | Visual bindings | Pages | Status |
|---|---|---|---|
| `[Welspun Products]` | 44 | 6 | Documented |
| `[Avg_Welspun_Product]` | 41 | 7 | **Expression not yet recovered** |
| `[Non-Welspun Products]` | 24 | 7 | Documented |

The unbranded series is not a measure — it is an implicit `Sum(storevisitoutput[Unbranded])`,
sourced from a different table entirely.

---

## `[Welspun Products]`

```dax
Welspun Products =
CALCULATE(
    SUM( programdetail[count] ),
    brand[is_welspun] = TRUE()
)
```

**What it does.** Sums the product count from `programdetail`, restricted to rows whose brand is
flagged as belonging to the client. This is the client's shelf footprint — the 9.22% headline,
and the measure behind the SPACES / WELSPUN split.

**Why it is written this way.** The classification lives on the `brand` dimension as a boolean
flag rather than being hard-coded into the measure as a list of brand names. That matters: when
the client adds a brand, it is classified once in the dimension table and every visual in the
report picks it up. A measure written as `brand[name] IN {"SPACES", "WELSPUN"}` would have needed
editing on every portfolio change.

**DirectQuery note.** The filter is a simple column predicate, so it folds cleanly to a SQL
`WHERE` clause rather than forcing a scan. This matters in DirectQuery, where every measure
evaluation is a round trip to Azure SQL.

**One subtlety worth knowing.** A bare column predicate inside `CALCULATE` is shorthand for
`FILTER( ALL( brand[is_welspun] ), brand[is_welspun] = TRUE() )` — it *replaces* any existing
filter on that column rather than intersecting with it. No visual in this report slices by
`is_welspun`, so there is no practical difference today. It would matter the moment someone added
such a slicer.

---

## `[Non-Welspun Products]`

```dax
Non-Welspun Products =
SUM( programdetail[count] ) - [Welspun Products]
```

**What it does.** Everything in `programdetail` that the client measure did not claim.

**Why this is a residual, not a definition — and why that is a problem.**

This measure never asks what a competitor *is*. It subtracts, so it returns *everything that is
not flagged as the client's*. Four categories of row land in it silently:

| Row type | Lands in Non-Welspun? | Should it? |
|---|---|---|
| `is_welspun = FALSE` — a genuine competitor | Yes | ✅ Yes |
| `is_welspun = BLANK` — unclassified brand | Yes | ❌ No |
| Row fails to join to `brand` | Yes | ❌ No |
| Unbranded rows, if `programdetail` holds any | Yes | ❌ No — double-counts against `storevisitoutput[Unbranded]` |

A competitor count defined as "everything else" absorbs every data-quality problem in the model.
The fourth row is the serious one: the report presents client / competitor / unbranded as three
mutually exclusive categories. If any unbranded rows exist in `programdetail`, they are counted
once inside `[Non-Welspun Products]` and again in the separate unbranded series. A donut chart
normalises to 100% regardless, so the chart would still *look* correct.

### How to test it

Add these three measures temporarily and put them on a card:

```dax
-- 1. Competitors, defined explicitly rather than by subtraction
NW Explicit =
CALCULATE( SUM( programdetail[count] ), brand[is_welspun] = FALSE() )

-- 2. The gap between the residual and the explicit definition.
--    Returns 0 if the residual is safe. Anything else is being swept up silently.
Unclassified Check =
[Non-Welspun Products] - [NW Explicit]

-- 3. Does programdetail contain unbranded rows at all?
Blank Brand Rows =
CALCULATE( SUM( programdetail[count] ), ISBLANK( brand[is_welspun] ) )
```

If `Unclassified Check` returns 0 across every page, the residual is behaving and the only change
needed is the explicit rewrite below for maintainability. If it returns anything else, the
competitor figures in the report are overstated by that amount.

### The corrected definition

```dax
Non-Welspun Products =
CALCULATE(
    SUM( programdetail[count] ),
    brand[is_welspun] = FALSE()
)
```

Symmetric with `[Welspun Products]`, immune to blanks and join failures, and it states what a
competitor *is* rather than what it is not.

> **Not yet applied to the `.pbix` in this repository.** The model is stored compiled, so this has
> to be typed in Power BI Desktop. Step-by-step procedure: [`../dax/README.md`](../dax/README.md).

**Order matters.** Run the reconciliation above first. If `Unclassified Check` returns anything
other than zero, applying this fix will move the 22.40% competitor share and every percentage
derived from it — meaning the screenshots and figures throughout this repository would need
regenerating in the same change.

---

## `[Avg_Welspun_Product]`

```dax
-- expression not yet recovered from Power BI Desktop
```

**Status: this measure needs investigating before anything else in this document.**

It is the second most-bound measure in the report — 41 bindings across 7 of the 10 pages — and it
has never once been displayed under a name resembling its own:

| Display name on canvas | Times |
|---|---|
| `# Non- Welspun Products` | 12 |
| `Non-Welspun Products` | 9 |
| `% Non- Welspun Products` | 8 |
| `% Non-Welspun Products` | 8 |
| `# Non-Welspun Products` | 4 |

A measure named `Avg_Welspun_Product` is labelled as a **non-Welspun count or percentage**
everywhere it appears. One of those two things is wrong, and which one determines whether the
report's competitor figures are correct.

### The collision

`[Non-Welspun Products]` and `[Avg_Welspun_Product]` share display names:

| Display name | `[Non-Welspun Products]` | `[Avg_Welspun_Product]` |
|---|---|---|
| `Non-Welspun Products` | 13 | 9 |
| `% Non-Welspun Products` | 5 | 8 |
| `# Non-Welspun Products` | 5 | 4 |

**Two different calculations appear under identical labels, on the same report, sometimes on the
same page.** A user comparing two visuals that both say "% Non-Welspun Products" may be reading
two different numbers with no way to tell.

This also explains the spacing variants (`% Non- Welspun Products` with the extra space): those
are used *exclusively* by `[Avg_Welspun_Product]`. The space looks like a typo, but it is the only
thing distinguishing the two measures on those visuals — and it is not applied consistently, since
both measures also share the unspaced name.

### What to do

1. Open the `.pbix` in Power BI Desktop → Model view → select `Avg_Welspun_Product` → read the
   formula bar.
2. If it computes an **average of client products**, every visual labelling it as non-Welspun is
   mislabelled and those figures are wrong.
3. If it computes a **non-Welspun count or percentage**, the measure is simply misnamed — rename
   it to match what it does.
4. Either way, resolve the display-name collision so one label means one calculation.

---

## Summary of the calculation layer

```
programdetail[count]  ──┬── brand[is_welspun] = TRUE()  ──▶  [Welspun Products]
                        │
                        └── (total − Welspun)           ──▶  [Non-Welspun Products]   ⚠ residual

storevisitoutput[Unbranded]  ──── implicit Sum()        ──▶  Unbranded series

??? ─────────────────────────────────────────────────── ──▶  [Avg_Welspun_Product]   ⚠ unknown
```

Two of the four series in the report's core comparison are fully specified. One is a residual that
needs a reconciliation check. One is unidentified.

---

[← Report Pages](03-report-pages.md) · [Next: Roadmap →](05-roadmap.md)
