<a name="readme-top"></a>

> [!NOTE]
> <b>The Look E commerce - Public Dataset Big Query</b>

<h1 align="center">
  <span style="color: #627254; font-size: 44px; font-weight: bold;">
    Laporan Penjualan Produk dengan Pendapatan Tertinggi Setiap Bulannya
  </span>
</h1>

---

<div style="text-align: left;">
  <h2 style="background-color: #2C3E3D; color: white; padding: 10px 20px; margin-bottom: 20px; border-radius: 8px; font-weight: bold; display: inline-block; font-size: 30px">SQL Syntax</h2>
</div>

---

<div style='text-align: justify'>

```sql
-- Membuat tabel sementara untuk agregasi penjualan produk per bulan
WITH report_monthly_orders_product_agg AS (
  SELECT
    DATE_TRUNC(DATE(o.created_at), MONTH) AS year_month,
    p.id AS product_id,
    p.name AS product_name,
    p.brand,
    p.category,
    COUNT(*) AS total_items_sold, -- Total unit produk yang terjual
    SUM(o.sale_price) AS total_revenue -- Total pendapatan dari penjualan produk
  FROM
    `bigquery-public-data.thelook_ecommerce.order_items` AS o
  JOIN
    `bigquery-public-data.thelook_ecommerce.products` AS p
    ON o.product_id = p.id
  WHERE
    o.status = 'Complete' -- Hanya mengambil transaksi yang sudah selesai
  GROUP BY
    year_month, product_id, product_name, brand, category
),

-- Membuat peringkat produk berdasarkan total revenue tiap bulan
ranked_products AS (
  SELECT 
    *,
    RANK() OVER (PARTITION BY year_month ORDER BY total_revenue DESC) AS rank
  FROM report_monthly_orders_product_agg
)

-- Menampilkan 5 produk setiap bulan berdasarkan total revenue (penjualan tertinggi)
SELECT
  *
FROM ranked_products
WHERE rank <= 5 -- Mengambil hanya 5 produk teratas
ORDER BY year_month, rank;

</div>
