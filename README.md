# sql-to-ml-pipeline-mistakes

Real-world SQL mistakes that silently corrupt ML training data — with examples and fixes.

---

## Contents

- Scenario
- Mistake #1: Duplicate Rows from JOIN
- Mistake #2: Silent NULL Handling
- Mistake #3: Distorted Aggregations
- Why This Matters

---

## 🎯 Scenario

A churn prediction model started degrading.

- No hyperparameters changed  
- No architecture changes  
- No retraining issues  

The problem?

Upstream SQL logic.

Machine learning models learn from the dataset they’re given.

If the query layer is flawed, the model output will be too.

---

## 🚨 Mistake #1: Duplicate Rows from JOIN

### ❌ Problem

```sql
SELECT *
FROM customers c
JOIN orders o
ON c.customer_id = o.customer_id;
```

If customers have multiple orders, this duplicates customer rows — inflating training data.

This silently biases models and distorts feature distributions.

### ✅ Fix

```sql
SELECT
    c.customer_id,
    COUNT(o.order_id) AS total_orders
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id
GROUP BY c.customer_id;
```

Aggregate intentionally before training.

---

## 🚨 Mistake #2: Silent NULL Handling

### ❌ Problem

```sql
SELECT income
FROM customers;
```

If NULL values exist and aren’t handled deliberately, model features become skewed.

NULL ≠ 0  
NULL ≠ missing at random  

It changes distributions and impacts feature integrity.

### ✅ Fix

```sql
SELECT
    COALESCE(income, 0) AS income
FROM customers;
```

Or better: define and document an intentional imputation strategy before model training.

---

## 🚨 Mistake #3: Distorted Aggregations

### ❌ Problem

```sql
SELECT AVG(transaction_amount)
FROM transactions;
```

Global averages hide segmentation.

Models trained on distorted aggregates often underperform in production.

### ✅ Fix

```sql
SELECT
    customer_id,
    AVG(transaction_amount) AS avg_txn
FROM transactions
GROUP BY customer_id;
```

Align aggregation logic with the model objective.

---

## 📌 Why This Matters

Many ML failures blamed on “model accuracy” are actually upstream data logic issues.

Strong ML systems require strong SQL foundations.

This repository highlights common silent data corruption patterns that impact:

- Feature engineering
- Model training
- Production stability
- Decision integrity

Upstream data logic determines downstream model behavior.

Data pipelines are part of the model architecture — not just preprocessing.

Strong models are built on strong data contracts.
