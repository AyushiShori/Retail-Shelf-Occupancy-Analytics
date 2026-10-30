# Limitations

[← Back to README](README.md)

Constraints of this build, stated plainly. Items that are fixable are tracked in
[docs/05-roadmap.md](docs/05-roadmap.md).

---

## 1. Restricted report access

The dashboard was published to the Power BI Service but was accessible only to authorised users
with the necessary workspace permissions. External stakeholders and anyone outside the permitted
group could not open it.

**Consequence for this repository:** the report cannot be shared as a live link. The screenshots
and documentation here are the record of what was built.

---

## 2. Semantic model permissions

Viewing the report required permission on the underlying semantic model as well as on the report
itself, depending on how the report was shared and configured. A user granted report access alone
would see the page furniture but no data.

This was intended behaviour for a report carrying commercial shelf data, but it made onboarding
new viewers a two-step administrative task rather than a one-step one.

---

## 3. DirectQuery dependency

The report depends on the availability and performance of the Azure-hosted SQL Server. Every
slicer change and every page navigation issues live queries against the source.

**What this means in practice:**

- Slow source queries surface directly as slow report interactions.
- Connectivity issues to Azure make the report unusable, not merely stale.
- The available DAX surface is restricted relative to an Import model.

This was an accepted trade for data currency — see
[docs/01-data-architecture.md](docs/01-data-architecture.md) for the reasoning.

---

## 4. Data dependency

The accuracy of every number in the report depends on the quality, completeness and timeliness of
the client-provided data. The report measures what field staff captured, not what was on the
shelf.

Two specific exposures:

- **Capture coverage is uneven.** The *User Analysis* page exists precisely to make this
  visible: image capture is concentrated in a small number of roles, and one in sixteen images
  enters through an unattributed `root` account.
- **Sampling density varies over time.** Early in the programme, individual days contain very few
  store visits, making daily percentages unstable. This is visible as the spiky left-hand
  region of the trend chart.

Neither is a flaw in the report — both are properties of the dataset that the report was built
to expose rather than conceal.

---

## 5. Descriptive, not predictive

The report analyses historical and current product distribution. It does not forecast.

It answers *where is our shelf share weak today* — not *where will it be weak next quarter*, and
not *what would happen if we intervened in these fifty stores*. Forecasting shelf-occupancy
trends, or modelling the expected return on a store intervention, would be a separate piece of
work.

This was a scoped decision, not an oversight: the client needed a measurement layer before any
predictive layer could be justified.

---

## 6. Measurement scope

The three categories — client brand, competitor brands, unbranded — are measured as a share of
*observed* shelf facings in *visited* stores. The report does not claim to represent:

- Stores never visited
- Shelf space outside the photographed area
- Stock held in the back room rather than displayed
- Sell-through, only presence

Share of shelf is a visibility metric. It correlates with commercial outcome but does not
measure it.

---

[← Back to README](README.md) · [Roadmap →](docs/05-roadmap.md)
