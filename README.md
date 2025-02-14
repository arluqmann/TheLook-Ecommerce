<a name="readme-top"></a>

> [!NOTE]
> <b>The Look E commerce - Public Dataset Big Query</b>

<h1 align="left">
  <span style="color: #627254; font-size: 44px; font-weight: bold;">
    Laporan Penjualan Produk pada Salah Satu E-commerce dengan <br> 
    Pendapatan Tertinggi Setiap Bulannya
  </span>
</h1>

---

**Author:** [Arief Luqman Hakiem](https://www.linkedin.com/in/ariefluqman/) <br>
**Data Source:** [Google Big Query - Public Dataset](https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce?project=cool-benefit-286000) <br>

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
```

</div>



<div style="text-align: left;">
  <h2 style="background-color: #2C3E3D; color: white; padding: 10px 20px; margin-bottom: 20px; border-radius: 8px; font-weight: bold; display: inline-block; font-size: 30px">Penjelasan Syntax</h2>
</div>

---

<div style='text-align: justify'>

<strong>Bagian I. Membuat tabel sementara untuk agregasi penjualan produk per bulan</strong>

- `WITH report_monthly_orders_product_agg AS (...)` — membuat tabel sementara (Common Table Expression - CTE) yang menyimpan data agregasi penjualan produk per bulan sebelum dilakukan peringkat.
- `DATE_TRUNC(DATE(o.created_at), MONTH) AS year_month` — mengubah tanggal transaksi `(created_at)` menjadi format bulanan `(YYYY-MM-01)`.
- `p.id AS product_id, p.name AS product_name, p.brand, p.category` — mengambil informasi unik dari produk.
- `COUNT(*) AS total_items_sold` — menghitung jumlah produk yang terjual dalam satu bulan.
- `SUM(o.sale_price) AS total_revenue` — menghitung total pendapatan dari penjualan produk dalam satu bulan.
- `JOIN dengan tabel products` — menghubungkan order_items dengan products untuk mendapatkan detail produk.
- `WHERE o.status = 'Complete'` — memastikan hanya transaksi yang sudah selesai yang diambil.
- `GROUP BY` — mengelompokkan data berdasarkan bulan dan produk untuk agregasi total penjualan.

<strong>Bagian II. Membuat peringkat produk berdasarkan total revenue tiap bulan</strong>

- `RANK() OVER (...) AS rank` — memberikan peringkat (rank) kepada setiap produk.
- `PARTITION BY year_month` — peringkat diberikan dalam lingkup setiap bulan.
- `ORDER BY total_revenue DESC` — mengurutkan pendapatan produk dari tertinggi ke terendah.

<strong>Bagian III. Menampilkan 5 produk setiap bulan berdasarkan total revenue (penjualan tertinggi)</strong>

- `WHERE rank <= 5` — hanya mengambil 5 produk dengan revenue tertinggi setiap bulan.
- `ORDER BY year_month, rank` — mengurutkan hasil berdasarkan bulan dan peringkat produk dalam bulan tersebut.

</div>

<br><hr>
[🔼 Back to top](#readme-top)
