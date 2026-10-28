# Dashboard — Publishing Performance

## 1. Objective and audience

The goal is to give the Head of Publishing Operations, CFO, and four studio leads a one-page view of revenue performance, cash settlement, refunds, and studio contributions.

The dashboard prioritizes the questions in the metric brief. It separates revenue recognized by order date from cash settled by settlement date and refunds recognized by refund date. All financial amounts are shown in USD and rounded to two decimal places.

## 2. One-page dashboard wireframe

The following ASCII wireframe represents the proposed dashboard layout. It is a design specification, not a screenshot of an implemented Power BI report.

```text
+--------------------------------------------------------------------------+
| PUBLISHING PERFORMANCE                                      Currency: USD |
| FY2026 Q2: Jan 1 - Mar 31, 2026           Fiscal period: FY2026 Q2        |
+--------------------------------------------------------------------------+
| COMPANY NET REVENUE | YOY REVENUE GROWTH | PAYING PLAYERS | REV / PLAYER |
| $315,862.09         | +120.55%           | 12,498         | $25.27       |
+--------------------------------------+-----------------------------------+
| NET REVENUE BY TITLE                 | CASH & REFUNDS                    |
| FY2026 Q2                            | December 2025                     |
|                                      |                                   |
| Hollow Crown              $145,510.77| Net revenue settled               |
| Hollow Crown: Ashen Tide   $44,894.38| $103,907.40                       |
| Emberwatch Tactics         $51,037.14|                                   |
| Saltmarsh Rally            $47,119.73| Refunds recognized                 |
| Lantern & Lock             $27,300.05| $5,009.66                         |
+--------------------------------------+-----------------------------------+
| FY2026 STUDIO-ATTRIBUTED REVENUE     | SELECTED STUDIO'S TITLE MIX       |
|                                      | Twin Hearth Studios (FY2026)      |
| Twin Hearth Studios       $915,896.67|                                   |
| Emberwatch Interactive    $424,406.54| Hollow Crown                      |
| Nine Lanterns             $309,959.94| $679,729.98  | 74.22%            |
| Saltpine Games            $215,396.86|                                   |
|                                      | Hollow Crown: Ashen Tide          |
| Company net revenue:    $1,441,253.46| $236,166.69  | 25.78%            |
|                                      |                                   |
| Studio rows are NON-ADDITIVE because| Title shares sum to 100.00%       |
| co-developed titles receive full    | of Twin Hearth attributed revenue.|
| credit for each credited studio.    |                                   |
+--------------------------------------+-----------------------------------+
| NOTES: Revenue recognized by order date. Settlement cash and refunds use  |
| their respective dates. Studio-attributed totals are not additive.        |
+--------------------------------------------------------------------------+
```

## 3. What is on the dashboard, in order, and why

### A. Header and reporting period

The header identifies the report and explicitly states that the primary revenue period is FY2026 Q2, covering January 1 through March 31, 2026. This prevents readers from confusing the quarterly revenue results with the December 2025 cash and refund figures or the FY2026 annual studio figures.

### B. Four headline KPIs

The first row shows company net revenue, year-over-year growth, paying players, and net revenue per paying player.

These KPIs answer the highest-level questions first: how much revenue was generated, how performance changed, how many players contributed positive net revenue, and how much revenue was generated per paying player. The company-wide KPI cards are visible only to authorized company-level users.

### C. Net revenue by title

The title section lists net revenue for all five titles in FY2026 Q2. It helps publishing leadership identify which titles contribute most to quarterly revenue and compare performance across the portfolio.

Revenue is based on order date, with net refunds attributed to their original order date. The displayed title amounts are rounded individually; the company total is calculated from the unrounded underlying values.

### D. Cash settled and refunds

This section reports December 2025 cash settlement and refunds as separate measures:

- **Net revenue settled:** $103,907.40, based on settlement date.
- **Refunds recognized:** $5,009.66, based on refund date, following Treasury's reporting basis.

They are kept separate because revenue recognition, settlement, and refund timing are different business events. Combining them into a single net figure could obscure their different date definitions.

### E. FY2026 studio-attributed revenue

This section compares the revenue attributed to each studio across FY2026. The company net revenue of $1,441,253.46 is displayed separately from the sum of the studio rows, which is $1,865,660.01.

The studio figures are explicitly labelled **non-additive** because co-developed titles receive full credit for each credited studio. Company revenue counts each transaction only once.

### F. Selected studio's title mix

The final analytical section shows each title's share of its credited studio's FY2026 revenue. The example shown is Twin Hearth Studios: Hollow Crown contributes 74.22%, while Hollow Crown: Ashen Tide contributes 25.78%.

The section helps a studio lead understand their studio's revenue concentration without implying that title shares across different studios can be added together. For a studio lead, the section is filtered to their authorized studio rather than showing another studio's title mix.

## 4. How a studio lead sees only their studio

The intended implementation uses Power BI row-level security (RLS) with an authorization mapping that associates each user's login with their permitted studio.

The security filter should flow through `dim_studio`, `bridge_title_studio`, and the relevant title and revenue calculations. For studio leads, title-level visuals and studio-attributed measures are restricted to titles credited to their authorized studio. Company-wide KPIs, other studios' revenue, and company-level comparisons are hidden from studio leads and remain available only to authorized company-level users.

Co-developed titles require special care: a studio lead may see a title that is credited to their studio, but must not thereby gain access to the other studios' broader financial data. Revenue attribution rules and security rules serve different purposes and must be tested independently.

RLS is a design requirement described here; the wireframe itself does not implement or prove that the security configuration works. Before release, test the report under a company-level role and each studio lead role to verify both visible totals and drill-down behavior.

## 5. What is deliberately left off

| Left off | Why it was excluded | What would justify adding it |
|---|---|---|
| Daily or hourly revenue trends | The brief focuses on fiscal-period results and a one-page executive view. | Leadership needs to detect short-term changes or monitor a launch. |
| Country, platform, and acquisition-channel breakdowns | They add detail but are not necessary to answer the listed questions. | A specific market, platform, or marketing decision requires them. |
| Refund reasons and refund rates by title | The requested Treasury metric is the total refund amount by refund date. | Finance or publishing wants to investigate refund drivers or unusual refund patterns. |
| Detailed order and player records | The audience needs aggregated metrics, not individual-level data. | An authorized analyst needs a governed drill-through workflow for an investigation. |
| Forecasts and revenue targets | No approved targets or forecasting assumptions were supplied. | Leadership provides approved targets, forecast methodology, and reporting expectations. |
| Operational metrics such as retention, conversion, and lifetime value | The brief does not define these measures or request them. | The business defines the metrics and confirms the underlying data is reliable. |



