# MODEL.md — Twin Hearth Studios Analytics

## 1. Model objectives

The semantic model supports publishing leadership's FY2026 reporting requirements, including net revenue, cash settlements, refunds, paying players, year-over-year growth, and studio-attributed performance.

The model follows a star-schema approach where practical, with separate order and refund fact tables. Measures must respect the distinction between revenue recognition dates, cash settlement dates, and refund transaction dates.

## 2. Tables and grain

| Table | Grain | Purpose |
|---|---|---|
| `fact_order` | One row per order (`order_id`) | Order values, platform fees, order status, order date, and settlement date |
| `fact_refund` | One row per refund (`refund_id`) | Refund amounts, original order date, refund date, and refund reason |
| `dim_date` | One row per calendar date | Calendar and fiscal reporting |
| `dim_title` | One row per title (`title_id`) | Title name, genre, launch date, lead studio, and base price |
| `dim_player` | One row per player (`player_id`) | Player and acquisition attributes |
| `dim_platform` | One row per platform (`platform_code`) | Platform name and settlement lag |
| `dim_studio` | One row per studio (`studio_id`) | Studio name, region, and founding year |
| `bridge_title_studio` | One row per title–studio credit | Maps titles to all credited studios, including co-development |

The order and refund facts must remain separate when aggregating their amounts. Joining both facts directly at transaction level can multiply rows and inflate revenue.

## 3. Relationships and filter direction

Use one-to-many relationships with single-direction filtering from dimensions toward facts wherever possible.

| One side | Many side | Cardinality | Filter direction |
|---|---|---|---|
| `dim_title[title_id]` | `fact_order[title_id]` | 1:* | Dimension to fact |
| `dim_title[title_id]` | `fact_refund[title_id]` | 1:* | Dimension to fact |
| `dim_player[player_id]` | `fact_order[player_id]` | 1:* | Dimension to fact |
| `dim_player[player_id]` | `fact_refund[player_id]` | 1:* | Dimension to fact |
| `dim_platform[platform_code]` | `fact_order[platform_code]` | 1:* | Dimension to fact |
| `dim_platform[platform_code]` | `fact_refund[platform_code]` | 1:* | Dimension to fact |
| `dim_date[date]` | `fact_order[order_date]` | 1:* | Active; dimension to fact |
| `dim_date[date]` | `fact_order[settlement_date]` | 1:* | Inactive; dimension to fact |
| `dim_date[date]` | `fact_refund[order_date]` | 1:* | Active; dimension to fact |
| `dim_date[date]` | `fact_refund[refund_date]` | 1:* | Inactive; dimension to fact |
| `dim_studio[studio_id]` | `bridge_title_studio[studio_id]` | 1:* | Dimension to bridge |
| `dim_title[title_id]` | `bridge_title_studio[title_id]` | 1:* | Dimension to bridge |

If Power BI requires a different active date relationship for a report page, use explicit date-role measures or inactive relationships with `USERELATIONSHIP`. Do not enable bidirectional filtering indiscriminately.

### Studio attribution

The bridge represents a many-to-many business attribution: a title may be credited to multiple studios, and each credited studio receives 100% of that title's revenue.

Keep the bridge relationships single-direction to avoid ambiguous filter paths. Studio-attributed measures should transfer the selected studio's title IDs to `dim_title` using `TREATAS` or an equivalent explicit filter pattern.

Company-wide measures must not apply studio attribution. Studio totals are non-additive and may sum to more than company revenue.

## 4. Date table and fiscal calendar

Use `dim_date` as the canonical calendar and mark it as the Power BI date table using its unique `date` column.

The fiscal year begins October 1:

- FY2026: October 1, 2025–September 30, 2026.
- FY2026 Q1: October–December 2025.
- FY2026 Q2: January–March 2026.
- FY2026 Q3: April–June 2026.
- FY2026 Q4: July–September 2026.

Use the existing `fiscal_year`, `fiscal_quarter`, `fiscal_quarter_no`, and `fiscal_month_no` columns rather than recreating fiscal logic in every measure.

## 5. Revenue and cash business logic

### Net revenue

Order and refund amounts are already net of the platform's cut:

- Order revenue: `fact_order[net_usd]`.
- Refund revenue: `fact_refund[net_refund_usd]`.
- Net revenue: order net revenue minus refund net refunds, with both filtered by the original order date.

Do not subtract `platform_fee_usd` again.

### Date roles

- Revenue is recognized on `fact_order[order_date]`.
- Revenue reductions from refunds are attributed to `fact_refund[order_date]`.
- Cash receipts from orders use `fact_order[settlement_date]`.
- Treasury refund outflows use `fact_refund[refund_date]`.

These dates answer different questions and must not be treated interchangeably.

### Refund status

`fact_order[order_status] = "refunded"` means the order was fully refunded. Partially refunded orders remain `"completed"`. Use the refund fact to calculate refund amounts; never infer refund amounts from order status.

## 6. Measures and exposed fields

Expose business-facing dimensions and measures needed for leadership reporting.

Recommended visible dimension fields:

- Date: fiscal year, fiscal quarter, fiscal month, calendar date.
- Title: title name and genre.
- Studio: studio name and region.
- Platform: platform name.
- Player: acquisition channel and country, where appropriate.

Hide technical keys from ordinary report consumers when they are not needed for analysis, including surrogate or business IDs, bridge keys, and internal transaction identifiers. Keep necessary relationship columns in the model even if hidden from report view.

Hide raw amount columns from casual report authors where practical, and encourage use of certified measures to keep business logic consistent.

## 7. Paying-player definition

For this model, a paying player is a distinct player whose net revenue after refunds attributed to orders in the selected period is greater than zero.

Net revenue per paying player is:

`Net Revenue / Paying Players`

Document this definition in the report because other definitions, such as any player with at least one purchase before refunds, can produce a different count.

## 8. Row-level security intent

Studio leads should only see titles attributed to their own studio. The Head of Publishing Operations and CFO require company-wide access.

Implement RLS using a user-to-studio access mapping and a controlled studio filter path through `bridge_title_studio`. Studio-level measures should retain the selected studio context. Company-wide measures and totals should only be exposed to authorized leadership roles.

Validate RLS by testing each role with titles that are shared across studios, since co-developed titles are credited in full to each studio.

## 9. Performance at 50x order volume

At 50 times the current order volume, prioritize:

1. Import mode and columnar storage for the large fact tables where refresh and capacity constraints permit.
2. Incremental refresh partitioned by an appropriate date column, with historical corrections handled explicitly.
3. Source-side aggregation and pre-aggregation for common leadership queries when detail-level analysis is not required.
4. Integer or compact key columns for relationships and reduced cardinality in large facts.
5. Measures that aggregate each fact independently rather than joining order and refund rows at transaction level.
6. Performance Analyzer and query-plan testing on realistic volume before adding complex calculated columns or bidirectional relationships.

The current model is designed around the supplied dataset. If order volume grows significantly, validate refresh times and query responsiveness using a representative larger dataset. Consider incremental refresh or pre-aggregated tables if testing shows they are necessary.

Keep the model simple and measure-driven. Any optimization must preserve the fiscal date logic, refund recognition rules, and full studio-credit business rule.
