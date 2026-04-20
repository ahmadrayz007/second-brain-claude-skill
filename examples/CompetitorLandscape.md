---
type: research
topic: "Geospatial analytics competitors"
created: 2026-04-15
updated: 2026-04-20
status: active
tags:
  - research/competitor
related:
  - [[Datamapan]]
  - [[2026-04-20-adopt-postgres]]
---

# Geospatial analytics competitors

## Context
Datamapan is preparing a Q3 launch. Before finalizing pricing and positioning we need a clear view of who else sells geospatial analytics to mid-market retailers in SEA. This note feeds into the positioning decision and the sales enablement doc.

Related: [[Datamapan]]

## Sources
- [CartoDB pricing page](https://carto.com/pricing) — starts at $300/mo, enterprise-focused
- [Kepler.gl docs](https://kepler.gl) — open source, Uber-backed, visualization only
- [Esri ArcGIS Online](https://www.esri.com) — dominant incumbent, heavy enterprise
- [Mapbox studio](https://www.mapbox.com) — strong for devs, weak for non-technical users

## Key Findings

1. **No player owns the "non-technical retail analyst" segment.** CartoDB and Esri require GIS expertise; Kepler assumes developers.
2. **Pricing floor is high.** Median entry is ~$300/mo — we can realistically price at $99 and still profit at volume.
3. **SEA-specific data is weak across all competitors.** Foot-traffic and demographic overlays for Indonesia are either missing or outdated (2019 data).
4. **Interactive dashboarding is table-stakes.** Every competitor ships it; we cannot skip it in MVP.

## Implications
- Positioning: "GIS for retail, without a GIS team."
- Pricing: $99 entry tier is defensible and differentiates.
- Roadmap: SEA data partnerships become a strategic moat, not a feature.
- MVP scope must include interactive dashboards (no deferring to v2).

## Open Questions
- [ ] Is there a credible SEA-only player we're missing? Check local search + LinkedIn.
- [ ] What's CartoDB's churn like in SEA specifically?
- [ ] Does Esri have a cheaper SKU we haven't seen?

## Next Steps
- [ ] Interview 5 retail analysts on tooling pain points → spawn [[RetailAnalystInterviews]]
- [ ] Draft positioning 1-pager
- [ ] Revisit pricing after interviews

## Log

### 2026-04-15
- Initial scan, 4 competitors identified.

### 2026-04-20
- Added pricing analysis. Confirmed with [[Farah Putri]] that SEA data gap is real based on her prior role at [Client X].
