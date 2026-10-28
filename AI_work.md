# Twin Hearth Studios — AI Workflow Documentation

## 1. Objective

I used AI as an analytical assistant to understand the assessment requirements, develop SQL queries, document the semantic model, draft DAX measures, and validate business metrics. AI-generated outputs were treated as hypotheses and checked against the warehouse and the metric brief before being included in the scorecard.

## 2. Tools Used and Why

| Tool | Purpose |
|---|---|
| Chat GPT - Claude - Gemini| Interpret business requirements, suggest query logic, explain modeling decisions, draft DAX, and help investigate discrepancies. |

## 3. Example AI Prompts and Outcomes

### Prompt 1 — Understand the warehouse

**Prompt:**  
"Inspect the available warehouse tables and columns. Explain the grain of each fact table and identify the relationships needed to calculate title revenue, studio revenue, refunds, and paying players."

**Outcome:**  
The warehouse contains order and refund fact tables, title and studio dimensions, a title-studio bridge, player and platform dimensions, and a date dimension. The order and refund tables must be handled separately because revenue recognition, settlement, and refund cash flow use different dates.

### Prompt 2 — Interpret revenue rules

**Prompt:**  
"Using the metric brief, explain how to calculate net revenue without double-subtracting platform fees, and distinguish revenue recognition from cash settlement."

**Outcome:**  
The `net_usd` and `net_refund_usd` fields already account for platform fees. Revenue reporting subtracts refunds based on the original order date, while cash reporting uses the settlement date for orders and refund date for refunds.

### Prompt 3 — Calculate quarterly revenue

**Prompt:**  
"Write a SQL query to calculate FY2026 Q2 net revenue by title and company total, counting each company transaction only once."

**Outcome:**  
The query filters orders to January–March 2026 and subtracts refunds associated with orders from that period. The company total is calculated independently of studio attribution to avoid double-counting co-developed titles.

### Prompt 4 — Calculate studio revenue shares

**Prompt:**  
"Calculate each Twin Hearth Studios title's share of the studio's FY2026 net revenue. Ensure the title shares reconcile to 100%."

**Outcome:**  
The title shares were recalculated using title-level net revenue after refunds and reconciled to the studio's FY2026 total. The final shares are 74.22% for Hollow Crown and 25.78% for Hollow Crown: Ashen Tide.

### Prompt 5 — Review the semantic model

**Prompt:**  
"Review the semantic model and DAX measures for date-role conflicts, studio attribution, refund treatment, and company-versus-studio totals."

**Outcome:**  
The review highlighted the need to distinguish order date from settlement date and refund date. Measures using inactive date relationships need to prevent the active original-date relationship from filtering the same fact table at the same time.

## 4. Incorrect AI Output and How It Was Caught

An initial AI tool calculation for Twin Hearth Studios' title revenue shares produced figures that did not reconcile to the studio's total. The resulting shares added up to more than 100%, which indicated a problem in the calculation.

I did not accept those results. I reviewed the query logic, simplified the calculation, and checked the refund treatment against the metric brief. The corrected title revenues were:

- Hollow Crown: $679,729.98
- Hollow Crown: Ashen Tide: $236,166.69
- Twin Hearth Studios total: $915,896.67

The corrected shares are 74.22% and 25.78%, respectively, and add up to 100.00%. As it should be

This validation showed why AI-generated SQL should be checked against reconciliation rules and independently calculated totals before being used in reporting.

## 5. Conclusion

AI accelerated query development, documentation, and review. The most important quality control was validating the output against the metric brief, the warehouse, and expected reconciliation behavior. Discrepancies were investigated and corrected rather than accepted without verification.
