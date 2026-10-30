# Data Model

[← Back to README](../README.md)

Fifteen tables sourced from the Azure SQL Server database, connected over DirectQuery.

## Logical grouping

![Table groupings](../assets/model/table-groupings.png)

### Geography (4 tables)

| Table | Role |
|---|---|
| `zone` | Top-level geographic grouping |
| `region` | Regional grouping within a zone |
| `area` | Area grouping within a region |
| `pincode` | Postal code — the finest geographic grain |

These four form the filter hierarchy that drives every geographic visual and the synced slicer
set at the top of each page.

### Product and classification (6 tables)

| Table | Role |
|---|---|
| `product` | Product-level information |
| `productdetail` | Additional product attributes |
| `brand` | Brand information — the basis of the client / competitor / unbranded split |
| `branddetail` | Additional brand attributes |
| `category` | Product category classification |
| `categorydetail` | Additional category attributes |

`brand` is the pivot of the whole analysis: it is what allows a shelf observation to be
classified into one of the three mutually exclusive buckets.

### Retail operations and supporting (5 tables)

| Table | Role |
|---|---|
| `store` | Retail outlet master — name, city, outlet code |
| `storetype` | Outlet classification by type |
| `storevisit` | **Central fact table** — visit records, visit date, brand measures |
| `storevisitoutput` | Per-visit shelf observations, including unbranded counts |
| `useranalytics` | Field-staff capture activity — users, user types, image counts |

![Table inventory](../assets/model/table-inventory-1.png)
![Table inventory continued](../assets/model/table-inventory-2.png)

## Field usage across the report

Extracted from the report definition — how often each column is bound to a visual:

| Column | Visuals |
|---|---|
| `storevisitoutput[Unbranded]` | 43 |
| `storevisit[created_at]` | 26 |
| `useranalytics[imagecount]` | 17 |
| `region[Region]` | 16 |
| `store[city]` | 15 |
| `store[name]` | 15 |
| `area[name]` | 14 |
| `storetype[name]` | 13 |
| `zone[Zone]` | 12 |
| `pincode[pincode]` | 8 |
| `brand[name]` | 8 |
| `store[code]` | 8 |
| `useranalytics[username]` | 5 |
| `category[name]` | 4 |
| `useranalytics[usertype]` | 4 |
| `useranalytics[store_name]` | 4 |
| `product[name]` | 3 |
| `useranalytics[created_at]` | 3 |
| `useranalytics[user_id]` | 1 |

Twelve of the fifteen tables are referenced directly by visuals. The three that are not
(`productdetail`, `categorydetail`, `programdetail`) support the model without appearing on a
canvas.

**`programdetail` is the important one of those three.** No visual binds to it, but it is the
table every brand measure aggregates — `SUM(programdetail[count])`, classified by
`brand[is_welspun]`. It reaches the canvas only through the measures, which is why a field-usage
count scored it zero. A reviewer reading the model should start there, not at `storevisit`.

### Reading the distribution

`storevisit` is referenced far more than any other table (384 references across the report),
which is the expected shape for a star-ish model with a single dominant fact. `store`,
`storevisitoutput` and `useranalytics` follow, then the geography and product dimensions.

`storevisitoutput[Unbranded]` is the most-bound single column in the entire report — unsurprising,
since the unbranded category turned out to be the dominant finding.

## Measures

Three explicit measures carry the brand comparison:

| Measure | Home table | Aggregates |
|---|---|---|
| `[Welspun Products]` | `storevisit` | `programdetail[count]`, filtered `brand[is_welspun] = TRUE()` |
| `[Non-Welspun Products]` | `storevisit` | `programdetail[count]` minus the above (a residual) |
| `[Avg_Welspun_Product]` | `storevisit` | **unknown — see [04-measures.md](04-measures.md)** |

The unbranded series is produced by implicit aggregation —
`Sum(storevisitoutput[Unbranded])` — rather than by an explicit measure. Making it explicit is a
[roadmap item](05-roadmap.md).

Note that the three brand series are sourced from `programdetail` while the unbranded series is
sourced from `storevisitoutput`. The three categories are presented as mutually exclusive, so the
two tables must not overlap — a reconciliation check for this is given in
[04-measures.md](04-measures.md).

Expressions are documented (pending transcription) in [04-measures.md](04-measures.md).

### A note on display names

Several visuals surface the same underlying measure under different display names —
`% Welspun Products`, `# Welspun Products`, `Percentage of Welspun Products`,
`Occupancy Percentage`. These are per-visual display names and formatting over the three base
measures, not separate calculations. Consolidating the naming is a
[roadmap item](05-roadmap.md).

---

[← Data Architecture](01-data-architecture.md) · [Next: Report Pages →](03-report-pages.md)
