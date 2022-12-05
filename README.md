# SIPLah SQL

kumpulan sql untuk export data SIPLah Toko Ladang

# Export VA

## VA Jatim

``` sql
SELECT
	mc.bank_account ->> 'no' AS rek_toko,
	co.order_no AS keterangan,
	( co.payment_transferred - co.tax_fee - co.handling_fee ) AS TRANSFER_KE_MERCHANT,
	co.order_no AS order_no 
FROM
	customer_order co
	JOIN merchant mc ON mc.ID = co.merchant_id
	JOIN bank_mutations bm ON bm.ID = co.bank_mutation_in_id 
WHERE
	co.purchase_status IN ( 'payment_verified' ) 
	AND co.payment_method_id IN ( 2, 4 ) 
	AND co.handling_fee IS NOT NULL 
	AND mc.bank_account ->> 'bank' = 'BANK JATIM' 
	AND bm.updated_at BETWEEN '2022-12-05 00:00:00' 
	AND '2022-12-05 23:59:59' 
	AND mc."id" != 15107 
ORDER BY
	transfer_ke_merchant DESC
```

## BRIVA fro Mass FT

```sql
SELECT
-- 	bm.updated_at TANGGAL_TERVERIFIKASI,
CASE
		co.payment_method_id 
		WHEN 1 THEN
		'transfer' 
		WHEN 2 THEN
		'BRIVA' 
		WHEN 3 THEN
		'kode_bayar_BPDaja' 
		WHEN 4 THEN
		'VA Jatim' 
		WHEN 5 THEN
		'PAC' 
		WHEN 6 THEN
		'BJB' ELSE'tidak tau' 
	END AS BANK_LADANG,
	('041201001176308')::TEXT AS rek_ladang,
	mc.bank_account ->> 'no' AS rek_toko,
	mc.bank_account ->> 'name'AS accont_name,
	mc.email AS email_address,
	( co.payment_transferred - co.tax_fee - co.handling_fee ) AS amount,
	('IDR')::TEXT AS currency,
	
	co.order_no AS remark,
	co."id" AS ref_number
FROM
	customer_order co
	JOIN merchant mc ON mc.ID = co.merchant_id
	JOIN bank_mutations bm ON bm.ID = co.bank_mutation_in_id 
WHERE
	co.purchase_status IN ( 'payment_verified' ) -- 	AND co.payment_method_id = 4
	
	AND co.handling_fee IS NOT NULL 
	AND mc.bank_account ->> 'bank' = 'BRI' 
	AND bm.updated_at BETWEEN '2022-11-30 00:00:00' 
	AND '2022-11-30 23:59:59' 
	AND co.payment_method_id IN(3)
ORDER BY
	bm.updated_at ASC
```

## BPD Aja

```sql
SELECT
	( '20221205' ) :: TEXT AS rek_ladang,
	mc.bank_account ->> 'no' AS rek_toko,
	mc.bank_account ->> 'name' AS accont_name,
	co.order_no AS remark,
	( co.payment_transferred - co.tax_fee - co.handling_fee ) AS amount 
FROM
	customer_order co
	JOIN merchant mc ON mc.ID = co.merchant_id
	JOIN bank_mutations bm ON bm.ID = co.bank_mutation_in_id 
WHERE
	co.purchase_status IN ( 'payment_verified' ) -- 	AND co.payment_method_id = 4
	
	AND co.handling_fee IS NOT NULL 
	AND mc.bank_account ->> 'bank' = 'BRI' 
	AND bm.updated_at BETWEEN '2022-12-05 00:00:00' 
	AND '2022-12-05 23:59:59' 
	AND co.payment_method_id IN ( 3 ) --bpdaja
	
ORDER BY
	bm.updated_at ASC
```

---

# Pajak

```sql
SELECT
	tc.collected_at AS tanggal_pungut,
	tc.customer_order_id AS order_id,
	co.order_no AS nomor_po,
	( co.tax_fee_details -> 0 ->> 'amount' ) :: INTEGER AS pph,
	( co.tax_fee_details -> 1 ->> 'amount' ) :: INTEGER AS ppn,
	tc.amount AS total_pajak 
FROM
	"tax_collectors" tc
	INNER JOIN customer_order co ON tc.customer_order_id = co."id" 
WHERE
	collected_at BETWEEN '2022-11-01 00:00:00' 
	AND '2022-11-30 23:59:59' 
ORDER BY
	tc.collected_at ASC
```
