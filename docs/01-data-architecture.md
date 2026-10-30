# Data Architecture

[← Back to README](../README.md)

## Pipeline

```
Client JSON  →  Azure SQL Server  →  DirectQuery  →  Power BI  →  Business insight
   (raw)         (15 tables)         (live)         (10 pages)
```

![Pipeline stages](../assets/architecture/pipeline-stages-1.png)
![Pipeline stages continued](../assets/architecture/pipeline-stages-2.png)

## Stage detail

![Process description](../assets/architecture/process-description.png)

| Stage | What happened |
|---|---|
| **Data source** | The client provided the original retail observation data in JSON format. |
| **Data ingestion** | The JSON was processed and loaded into a SQL Server database hosted on Microsoft Azure. |
| **Cloud database** | The Azure-hosted SQL Server acted as the centralised, single source of truth for reporting. |
| **Data connectivity** | Power BI connected to the cloud database using **DirectQuery**. |
| **Data preparation** | Power Query handled data-type correction, missing values, category standardisation, and date / location validation. |
| **Data analysis** | Shelf occupancy analysed across product categories, brands, regions, areas and stores. |
| **Visualisation** | Interactive charts, synced slicers and ten report pages presented results to stakeholders. |
| **Business outcome** | Management could compare client, competitor and unbranded products and identify shelf-visibility opportunities. |

## Storage mode: why DirectQuery

The report runs in **DirectQuery** mode (confirmed in the Power BI Desktop status bar on the
report canvas screenshot).

**The reasoning:**

The client's operational systems continued writing store-visit records after the report shipped.
Field staff were capturing shelf images daily, and regional managers wanted to see the effect of
a store intervention without waiting for a refresh window.

| | Import | DirectQuery |
|---|---|---|
| Currency | Stale between refreshes | Always current |
| Query speed | Fast (in-memory) | Depends on source |
| DAX available | Full | Restricted subset |
| Model size | Bound by capacity | Unbounded |

With a daily-updated operational source and a report whose whole purpose was to drive
*this week's* store visits, currency outranked responsiveness. The cost of that decision —
dependence on Azure SQL availability and query performance — is recorded in
[LIMITATIONS.md](../LIMITATIONS.md).

## Transformation layer

Cleaning and shaping happened in the **Power Query Editor** before the data reached the model:

- **Data types** — corrected across all loaded tables, with particular attention to date columns
  feeding the trend visuals and numeric columns feeding the share measures.
- **Missing values** — handled so that absent shelf counts did not silently distort percentage
  calculations.
- **Category standardisation** — brand and category labels normalised so that the three-way
  client / competitor / unbranded split was clean and mutually exclusive.
- **Date and location validation** — visit dates and geographic fields validated, since every
  slicer in the report filters on one or the other.

## Deployment

The completed report was published to the **Power BI Service** and deployed to a designated
workspace. Permissions were configured at two levels:

1. **Report level** — who can open the report.
2. **Semantic model level** — who can query the underlying dataset.

Both were required. A user with report access but no dataset permission could not render the
visuals, which was the intended behaviour for a report carrying commercial shelf data.

---

[← Back to README](../README.md) · [Next: Data Model →](02-data-model.md)
