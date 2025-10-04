# Direct-Response Ad Creative Analysis: From Tagged Data to Actionable ROAS Insights

## Project Summary

This project demonstrates a full data science workflow designed to optimize a high-volume direct-response ad spend by analyzing structured creative tag data.

**The Goal:** To move beyond simple performance reporting and identify the specific **creative blueprints (tag combinations)** that drive the highest **Weighted Average ROAS (Return on Ad Spend)**, providing immediate, statistically-backed hypotheses for a creative team.

**Tech Stack Demonstrated:**
* **SQL (BigQuery Syntax):** For high-volume data joining and aggregation.
* **Python:** Pandas, NumPy for data manipulation; SciPy for statistical validation; Matplotlib/Seaborn for visualization.
* **Business Focus:** Prioritizing **Weighted ROAS** and developing a systematic feedback loop.

---

## 1. BigQuery/SQL for Data Aggregation

The analysis begins by aggregating simulated `creative_tags` and `performance_metrics` tables. To ensure we prioritize insights from high-volume, meaningful tests, we use **Weighted Average ROAS** ($\text{SUM}(\text{Revenue}) / \text{SUM}(\text{Spend})$) and filter out low-spend noise.

### SQL Query for Weighted Average ROAS (BigQuery Syntax)

```sql
SELECT
    t.hook_style,
    t.cta_type,
    -- Core Metric: Weighted Average ROAS (SUM(Revenue) / SUM(Spend))
    SAFE_DIVIDE(SUM(p.revenue_usd), SUM(p.spend_usd)) AS weighted_avg_roas,
    SUM(p.spend_usd) AS total_spend_usd,
    COUNT(DISTINCT p.ad_id) AS total_unique_ads
FROM
    -- Simulated Join of Tags and Performance Data
    `project_id.dataset.creative_tags` t
INNER JOIN
    `project_id.dataset.performance_metrics` p ON t.ad_id = p.ad_id
WHERE
    p.date >= DATE_SUB(CURRENT_DATE(), INTERVAL 90 DAY) -- Focus on recent performance
    AND t.hook_style IS NOT NULL
GROUP BY 1, 2
-- Only analyze combinations with significant testing volume (Spend > $5,000)
HAVING SUM(p.spend_usd) > 5000
ORDER BY weighted_avg_roas DESC
LIMIT 15;
