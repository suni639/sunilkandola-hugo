+++
# -- Front Matter (Metadata) --
title = "Rebuilding the Lulu & Burdie Financial Model using AI Agents"
date = 2026-06-03
draft = false  # Published case study
description = "How I rebuilt Lulu & Burdie's 6-year pre-launch financial model in Excel using Antigravity AI agents."
summary = "A case study on building a pre-launch financial forecasting model, modeling cash runways and supplier MOQs, and why I chose Excel transparency over Python Streamlit."

# -- Visuals --
featureImage = ""  # You can link a hero image here later

# -- Organization --
tags = ["Financial Modelling", "Excel", "Python", "E-Commerce", "Antigravity", "AI Agents", "Pre-Launch"]
categories = ["Projects"]

# -- Theme Options --
showAuthor = true
showTaxonomies = true
+++

{{< lead >}}
How I modernised the 6-year financial forecasting model for my business, Lulu & Burdie, using AI agents—and why I returned to Excel after trying to move everything to Python and Streamlit.
{{< /lead >}}

## The Problem: Static Projections and Pre-Launch Blind Spots

Like many founders preparing for launch, I relied on a spreadsheet to map our runway. But as we got closer to bringing our products to market, the static formulas in our old 6-year projection model started showing their limits. 

It assumed flat marketing budgets, flat customer acquisition costs (CAC), and had no seasonality—which is a major blind spot for a direct-to-consumer (DTC) e-commerce brand that lives and dies by Q4 spikes and summer slumps. More importantly, it had a classic repeat purchase compounding error: it calculated customer retention across our entire historical pool indefinitely, ignoring the fact that maternity wear has a natural utility window of about nine months. 

For a pre-launch startup, these static models hide the real risks. They mask the pre-launch cash burn, ignore supplier Minimum Order Quantity (MOQ) capital traps, and assume sales start from day one without list building.

---

## The Python Experiment: Streamlit vs. Excel Transparency

My first instinct was to move away from Excel altogether. I wrote a python-first financial engine and built a Streamlit web dashboard to run real-time sensitivity dials. 

{{< mermaid >}}
graph TD
    A[assumptions.yaml<br>Master Inputs] --> B(Python engine/calculations.py<br>Streamlit Dashboard)
    A --> C(openpyxl compiler/generate_maternity_model_6yr.py)
    C --> D[Maternity_Wear_Financial_Model_6Yr.xlsx]
    D --> E[Cover Page & Control Panel]
    D --> F[Executive Charts Dashboard]
    D --> G[Dynamic 3-Statement Model]
{{< /mermaid >}}

The Python dashboard was great for sliding levers and seeing immediate graphs, but I quickly hit a roadblock when it came to sharing the model. 

When you sit down with investors, banks, or manufacturing partners during a pre-launch raise, they don't want to run a local Streamlit server or trust a black-box Python script. They want a spreadsheet. Excel provides a level of row-by-row transparency and auditable cells that python dataframes simply cannot replicate for external stakeholders. 

The compromise? Use Python as the compiler to build a dynamic, formulas-only Excel model.

---

## Rebuilding the Model with AI Agents in Hours, Not Weeks

To achieve this, I used **Antigravity AI agents** to pair-program the rebuild. What would ordinarily take weeks of manual spreadsheet writing, testing, and debugging took us a single afternoon.

I configured a master `assumptions.yaml` file with our unit economics, and had the agents write a compiler script using `openpyxl`. The script generates the entire workbook programmatically, injecting clean, dynamic formulas instead of hardcoded numbers:

*   **Pre-Launch Runway Toggle:** Modeled a configurable pre-launch phase (e.g. 3 months of zero revenue but active fixed costs) to accurately project startup burn before the first product ships.
*   **Rolling Cohort Pools:** Repeat purchases are linked to a rolling 9-month pregnancy window (`=SUM(start_col:prev_col) * repeat_rate`), matching the maternity lifecycle.
*   **Supplier MOQ Modeling:** Replaced smooth inventory assumptions with stepped bulk purchases, exposing the cash flow dips when buying large fabric runs from suppliers.
*   **Dynamic Seasonality & Inflation:** Clicks are seasonal based on monthly indices (`=INDEX(Seasonality, MONTH(Date))`), and CPC inflates over time to represent ad auction fatigue.
*   **Stepped Headcount & COGS Discounts:** Salaries step up by +£1,500/month for every block of 500 monthly orders above our baseline, and fabric material costs automatically trigger a 5% bulk discount when monthly sales exceed 1,000 units.

The final output is a dynamically calculating workbook featuring an elegant cover control sheet, an executive charts dashboard tab (utilising native Excel charts), and integrated Three Statements (P&L, Cash Flow, and Balance Sheet) with corrected sign and debt treatments.

---

## Key Takeaways for Pre-Launch Startups

1. **Model the Cash Valley:** For a pre-launch startup, the primary purpose of a financial model is to find the *Cash Valley*—the lowest point of your cash balance before the business becomes cash-flow positive. This number dictates exactly how much seed capital you need to raise to launch safely.
2. **Pivot Assumptions Instantly:** In the pre-launch phase, supplier terms, shipping rates, and packaging costs change weekly. By compiling the Excel sheet programmatically from a config file, I can rewrite the entire model's logic (like shifting from 60 inventory days to bulk MOQ batches) in seconds, rather than manually editing hundreds of Excel formulas.
3. **Excel Remains the Language of Funding:** Investors and banks run on spreadsheets. A clean, fully formulaic Excel file builds trust because it allows stakeholders to inspect your math cell-by-cell.
