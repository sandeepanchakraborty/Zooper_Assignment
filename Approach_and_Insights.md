# BI Internship Assignment – Approach, Assumptions & Key Insights

**Submitted by:** Sandeepan  
**Assignment:** Car Insurance – Data Simulation & Analytics  
**Tools Used:** Python (pandas, numpy, matplotlib, seaborn)  
**Date:** March 10, 2026

---

## 1. My Approach

For this assignment I decided to use Python (Jupyter Notebook) instead of Excel or SQL because generating 1 million rows of realistic data is much more manageable with code. It also makes the logic fully reproducible — anyone can re-run the notebook and get the exact same output.

I broke the work into four clear phases:

1. Build the Policy Sales dataset
2. Build the Claims dataset (2025 and 2026 separately)
3. Answer the analytical queries
4. Build visualizations and projections for the bonus questions

---

## 2. Assumptions I Made

### Policy Sales Data

- **2024 is a leap year (366 days).** I distributed all 1,000,000 policies as evenly as possible across every single day. The way I did this: 1,000,000 ÷ 366 gives a base of 2,732 with a remainder of 634. So the first 634 days of the year get 2,733 policies each, and the remaining 332 days get 2,732. The difference is never more than 1, which I think is the fairest interpretation of "evenly distributed."

- **Tenure assignment** was done using `np.random.choice` with the exact probabilities [0.20, 0.30, 0.40, 0.10]. At 1 million records the actual counts come out extremely close to the 20/30/40/10 split.

- **Policy_Start_Date = Purchase Date + 365 days.** The assignment says "365 days after the purchase date" so I used exactly 365 calendar days rather than `DateOffset(years=1)`, which would behave differently in a leap year.

- **Policy_End_Date = Start Date + tenure in full years**, using pandas `DateOffset(years=n)` so it handles year boundaries correctly (e.g. a policy starting Feb 29 ends on Feb 28 of the target year).

- **Premium = ₹100 × Policy_Tenure** and **Vehicle_Value = ₹100,000** for every record as stated.

### Claims Data – 2025

- I filtered for all vehicles where the day-of-month of the purchase date was 7, 14, 21, or 28 in any month of 2024, then randomly sampled 30% of those as claimants.

- **Claim_Date was set to Policy_Start_Date exactly**, as the assignment states "filed exactly on the first date of Policy start."

- I validated that the policy was active at the time of the claim (Start Date ≤ Claim Date < End Date). All 2025 claimants passed this check since the claim date equals the start date by construction.

- **Each vehicle files only one claim in 2025** — the sampling is done once and there is no duplication.

### Claims Data – 2026

- I took all policies with `Policy_Tenure = 4` and sampled 10% randomly.

- The 9,997 sampled claimants were spread across the 59 days from January 1 to February 28, 2026 using the same base+remainder logic as the policy distribution, ensuring the daily count is as even as possible (difference of at most 1 per day).

- **Claim_Type = 2** was assigned to any vehicle that had already filed a claim in 2025; everyone else got **Claim_Type = 1**. Out of the ~9,997 claimants, around 382 were repeat claimants.

- I confirmed the policy was still active for all 2026 claimants. Since 4-year policies starting in early 2025 don't expire until early 2029, all of them passed the active check.

### Claim Amount

- Fixed at ₹10,000 per claim (10% of ₹1,00,000 vehicle value) for all claims in both years.

### Earned Premium (Query 6)

- I calculated a **daily premium rate** for each policy: `Daily Premium = Total Policy Premium ÷ Number of Tenure Days` (where tenure days = Policy_End_Date − Policy_Start_Date in calendar days).

- Earned premium up to Feb 28, 2026 = Daily Premium × number of days the policy was active within that window.

- For policies not yet started by Feb 28, 2026: earned premium = ₹0.

- For the 46-month forward forecast (Mar 2026 onward): I used proper calendar-month boundaries starting March 1, 2026, adding one full month at a time so each window correctly covers the actual number of days in that month.

---

## 3. Key Insights

### Insight 1 – Total Premium Collected in 2024 is ~₹2.4 Crore

The 3-year tenure bucket contributes the most to total premium because it has both the largest customer count (~400,000 policies) and a decent per-policy premium of ₹300. The 4-year bucket earns the most per customer (₹400) but only has ~100,000 customers.

### Insight 2 – 2025 Claims Are the Dominant Cost Driver

The 7th/14th/21st/28th purchase-day rule catches about 13% of the year's sales — roughly 130,000+ vehicles are eligible. Sampling 30% of those gives ~39,000+ claimants. At ₹10,000 a claim, this is the biggest single claims event. The claims land across all 12 months of 2025 (spread by when vehicles were purchased), but each month's spike is noticeable.

### Insight 3 – 2026 Activity Is Smaller but Concentrated

The 2026 claims only affect 4-year-tenure policies — roughly 10,000 vehicles spread over 59 days. The monthly cost is flat because of the even distribution. It's a manageable event compared to 2025 but it does pile onto policies that may have already claimed in 2025 (those 382 vehicles with Claim_Type = 2).

### Insight 4 – 3-Year Policies Are the Most Profitable Overall, But 1-Year Policies Have the Best Loss Ratio

In absolute rupee terms, the 3-year tenure generates the highest net profit simply because of volume — 400,000 customers × ₹300 each is a large premium pool. However, when you look at the **loss ratio** (claims ÷ premium), 1-year policies fare the best. This is because 1-year policies are not exposed to the 2026 claims at all (they only have a 1-year tenure and all expire before January 2026), and the 2025 claim exposure is proportional across tenures.

### Insight 5 – 4-Year Policies Carry the Most Risk

4-year policies are the only group affected by both the 2025 defective-day claims and the 2026 bulk claims. This double exposure makes them the highest-risk tenure in the portfolio. Their loss ratio is higher than any other group, and they require the most reserving attention.

### Insight 6 – The Claim-to-Premium Gap Is Very Wide (By Design)

One claim pays out ₹10,000. A 1-year policy brings in only ₹100 in premium. That means a single claim consumes 100 years' worth of premium from that policy. This is a simulation setup as given in the assignment, but it reflects a real-world truth: if an insurer doesn't price risk correctly, even a modest claim frequency can make the portfolio deeply unprofitable. In this portfolio the loss ratio runs into the hundreds of percent.

### Insight 7 – Significant Unrealised Liability Remains

As of February 28, 2026, most of the 1 million policies are still active and have never filed a claim. If every unclaimed active policy eventually files exactly one claim, the total payout liability runs into the tens of crores. This tail risk needs to be provisioned in the company's reserves — it can't be ignored just because claims haven't happened yet.

### Insight 8 – A 5% Annual Growth in Claim Frequency Makes Things Worse Every Year

Using 2025 as the base year (claim frequency ~3.93%), a 5% annual increase means by 2030 the frequency reaches ~5%. Since premiums are fixed, the loss ratio grows each year. Without a premium revision, tighter underwriting, or reinsurance, the portfolio's financial position will continue deteriorating. The right response is either to raise premiums in line with observed claim frequency or to exit the riskiest segments (e.g. vehicles from defect-prone batches).

---

## 4. Files Submitted

| File                            | Description                                                                |
| ------------------------------- | -------------------------------------------------------------------------- |
| `BI_Insurance_Assignment.ipynb` | Full Python notebook with all data simulation, queries, and visualizations |
| `Policy_Sales_Data.csv`         | 1,000,000 policy records for the year 2024                                 |
| `Claims_Data.csv`               | All 2025 and 2026 claim records with Claim_Type                            |
| `BI_Dashboard.png`              | 6-panel analytics dashboard (claim trends, loss ratios, tenure mix)        |
| `Loss_Ratio_Projection.png`     | 5-year loss ratio forecast under 5% annual claim frequency growth          |
| `Approach_and_Insights.md`      | This document                                                              |
