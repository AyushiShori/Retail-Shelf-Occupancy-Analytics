# Report Pages

[← Back to README](../README.md)

Ten pages, 203 visual containers. Page order below matches the report's own navigation order.

## Design principle

The report follows an **overview → detail** structure, repeated three times — once for each
product category. A user who learns the layout of *Region Details* can read *Non-Welspun Region
Details* and *Unbranded Region Details* without relearning anything.

Every page carries the same three pieces of furniture:

- **Synced slicer bar** across the top
- **Page navigator** down the left
- **Clear all Filters** action button

![Report canvas](../assets/report/report-canvas-region-wise.png)

---

## 1. Region Wise — landing page

**Slicers:** Date · Zone · Region · Area · Pincode

| Visual | Category | Values |
|---|---|---|
| 100% stacked area | Date | Client · Competitor · Unbranded |
| Donut | — | Client · Competitor · Unbranded |
| 100% stacked column | Region | Client · Competitor · Unbranded |
| 100% stacked bar | Area | Client · Competitor · Unbranded |

The entry point. Trend on the left, overall split on the right, then the same split broken by
region and by area. A national head answers "how are we doing, and where is it worst?" without
scrolling.

---

## 2. Product Wise

**Slicers:** Brand · Date · Category · Programme

| Visual | Category | Values | Tooltip |
|---|---|---|---|
| 100% stacked area | Date | Client · Competitor · Unbranded | — |
| Donut | Brand | Client measure | — |
| Column | Category | % Client | # Client |
| Bar | Programme | % Client | # Client |

Shifts the question from *where* to *what*: which product categories and which trade programmes
carry the client's shelf presence. Every percentage visual exposes the absolute count on hover.

---

## 3. Store Wise

**Slicers:** Store Name · Store Type · City · Date

| Visual | Category | Values | Tooltip |
|---|---|---|---|
| 100% stacked area | Date | Client · Competitor · Unbranded | — |
| Donut | Store Type | Client measure | — |
| 100% stacked column | Store Name | Client · Competitor · Unbranded | Outlet Code |
| 100% stacked bar | City | Client · Competitor · Unbranded | — |

Introduces **store type** as a dimension — a modern trade outlet and a neighbourhood store are
not comparable shelf opportunities, and the donut separates them. The city bar chart from this
page is the one reproduced in the README.

---

## 4. Region Details — client brand

**Slicers:** Date · Zone · Region · Area · Pincode · Store

| Visual | Detail |
|---|---|
| 100% stacked area | Trend by visit date |
| Donut | Zone split |
| Table | Pincode · Client · Occupancy % · Brands · Categories |
| Clustered column | Region — % Client, tooltip # Client |
| Bar | Area — % Client, tooltip # Client |
| Matrix | Rows: Pincode · Values: # Client, % Client |
| Clustered column | Region — Percentage of Client Products |

The first true detail page. The matrix carries pincode-level numbers that the charts summarise —
a chart to find the problem, a table to name it.

---

## 5. Store Details — client brand

**Slicers:** Store Name · Store Type · City · Date

| Visual | Detail |
|---|---|
| 100% stacked area ×2 | Trend by visit date |
| Table | Store Type · Client · Occupancy % · Store Name · Store City |
| Table | Store Type · Store Name · Store City · Occupancy % · Products Count |
| Donut ×2 | City split |
| Clustered column | Store Name — Percentage of Client Products |
| Bar | Store Name — % Client, tooltips # Client + Outlet Code |

The operational endpoint. The **Outlet Code** tooltip is what makes this page actionable — a
field manager needs the code, not just the store name, to raise a visit.

---

## 6. Non-Welspun Region Details — competitors

**Slicers:** Date · Zone · Region · Area · Pincode

| Visual | Detail |
|---|---|
| Matrix | Rows: Zone, Region · Values: % and # Competitor |
| Bar | Area — % Competitor |
| Donut | Store Type |
| Donut | Zone |
| Column | Region — % Competitor |
| Column | Store Name — % Competitor, tooltip Outlet Code |
| Table | Store City · % and # Competitor |
| **Pie** | **Brand — Competitor** |
| Bar | Area — % Competitor |
| Matrix | Rows: Pincode · Values: # and % Competitor |

The competitor mirror of page 4, with one addition: the **brand pie**. This is the only place in
the report that names *which* rival brands hold the shelf — the difference between "we are losing
22%" and "we are losing 22% to these three names."

---

## 7. Non-Welspun Store & Brand Details

**Slicers:** Store Name · Store Type · City · Date · Brand

Mirrors page 5 for competitors, and adds a **Brand slicer** plus a bar chart of
**% Competitor by Brand Name**. Combined with the brand pie on page 6, this lets a user isolate a
single rival and trace it down to the stores where it is winning.

---

## 8. Unbranded Region Details

**Slicers:** Zone · Pincode · Date · Region · Area

| Visual | Detail |
|---|---|
| Donut | Zone — Unbranded |
| Pie | Zone — Unbranded |
| Column | Region — % Unbranded, tooltip # Unbranded |
| Bar | Area — % Unbranded, tooltip # Unbranded |
| Matrix | Rows: Pincode · Values: # and % Unbranded |
| *(plus detail visuals — see roadmap)* | |

Covers the largest category — the 68.38% that dominates every city.

> **Known issue:** several visuals on this page are bound to competitor measures rather than
> unbranded ones. See [05-roadmap.md](05-roadmap.md).

---

## 9. Unbranded Store Details

**Slicers:** Store Name · Date · Store Type · City

Store-level view of unbranded stock, anchored by a matrix of **# and % Unbranded Products by
Store City** and a column chart of **% Unbranded by Store Name** with Outlet Code tooltips.

> **Known issue:** as with page 8, part of this page is bound to competitor measures. See
> [05-roadmap.md](05-roadmap.md).

---

## 10. User Analysis — data capture coverage

**Slicers:** User Type · Date · User ID · Username · Store Name

| Visual | Detail |
|---|---|
| Card ×4 | Total Image Count · Total Users · Total Visits · Total Stores |
| Pie | User Type — Images |
| Column | User Name — % Images, tooltip # Images |
| Bar | Store Name — % Images, tooltip # Images |
| Table | Usertype · Username · # Images · % Images |

The governance page. Four KPI cards establish the sample size, then the breakdown shows who
captured it. This is the page that lets a reader judge how much weight the other nine deserve —
see the README for what the user-type split revealed.

---

## Interaction model

| Technique | Where |
|---|---|
| Synced slicers | All 10 pages |
| Drill-through | Overview pages → matching detail pages |
| Page navigator | All 10 pages (left rail) |
| Back to report | All detail pages |
| Clear all Filters | All 10 pages |
| Bookmarks | 7 defined |
| Cross-filtering | Within every page |
| Custom tooltips | Every percentage visual carries its count |

---

[← Data Model](02-data-model.md) · [Next: Measures →](04-measures.md)
