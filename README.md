# 📊 Overdue Accounts Analysis (SQL)

## 📌 Objective

This project analyzes overdue customer accounts to support collection prioritization and recovery strategies.

---

## 🛠 Tools

* SQL (PostgreSQL)
* Data Analysis

---

## 📊 Business Problem

Collection teams need to identify customers with outstanding debts and prioritize actions based on risk and value.

This analysis answers:

* Which customers are overdue?
* What is the amount of debt?
* When was the last payment made?
* How many delay-related blocks exist?

---

## 🧮 SQL Analysis

```sql
-- 📊 Overdue Accounts Analysis
-- Objective: Identify customers in debt and support collection prioritization

WITH customers AS (
    SELECT
        tc.id_cliente,
        tc.nm_cliente,
        tr.nm_fantasia
    FROM decisionscard.t_cliente tc
    JOIN decisionscard.t_rede tr 
        ON tc.id_origem_comercial = tr.id_rede
),

overdue_invoices AS (
    SELECT 
        tf.id_cliente,
        tf.dt_vencimento::date AS due_date,
        tf.vl_fatura AS overdue_amount
    FROM decisionscard.t_fatura tf
    WHERE 
        tf.fl_ativa = 'S'
        AND tf.vl_fatura > 0
        AND NOT EXISTS (
            SELECT 1 
            FROM decisionscard.t_pagamento_fatura tpf
            WHERE tpf.id_fatura = tf.id_fatura
        )
),

last_payment AS (
    SELECT DISTINCT ON (id_cliente)
        id_cliente,
        dt_pagamento::date AS last_payment_date,
        vl_pagamento AS last_payment_amount
    FROM decisionscard.t_pagamento_fatura
    ORDER BY id_cliente, dt_pagamento DESC
),

blocks AS (
    SELECT
        tbc.id_cliente,
        COUNT(*) AS delay_blocks
    FROM decisionscard.t_bloqueio_cliente tbc
    JOIN decisionscard.t_tipo_bloqueio_cliente ttbc
        ON tbc.id_tipo_bloqueio_cliente = ttbc.id_tipo_bloqueio_cliente
    WHERE ttbc.id_tipo_bloqueio_cliente IN (2, 9, 25)
    GROUP BY tbc.id_cliente
)

SELECT
    c.id_cliente,
    c.nm_cliente,
    c.nm_fantasia,
    COALESCE(b.delay_blocks, 0) AS qtd_bloqueios_atraso,
    oi.due_date,
    oi.overdue_amount,
    lp.last_payment_date,
    lp.last_payment_amount
FROM customers c
JOIN overdue_invoices oi 
    ON c.id_cliente = oi.id_cliente
LEFT JOIN last_payment lp 
    ON c.id_cliente = lp.id_cliente
LEFT JOIN blocks b 
    ON c.id_cliente = b.id_cliente
ORDER BY oi.overdue_amount DESC;
```

---

## 📈 Metrics

* Overdue invoice value
* Due date
* Last payment date and amount
* Number of delay-related blocks

---

## 🧠 Key Insights

* Customers with higher overdue amounts should be prioritized
* Lack of recent payments indicates higher risk
* Frequent blocks may indicate chronic delinquency

---

## 🚀 Conclusion

This analysis helps collection teams focus on high-impact cases and improve recovery efficiency.

---

## 👨‍💻 Author

Manoel Sousa Gomes

🔗 LinkedIn: https://www.linkedin.com/in/manoel-sousa-712a6b240/
🔗 GitHub: https://github.com/Maelsousa
