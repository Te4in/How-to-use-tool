# RustScan Guide

## What is RustScan?
RustScan is a fast port scanner that works together with Nmap.

Official project:
https://github.com/RustScan/RustScan

---

# Basic Syntax

```bash
rustscan -a TARGET
```

---

# Basic Examples

## Scan all ports

```bash
rustscan -a 192.168.1.10
```

---

## Specify ports

```bash
rustscan -a 192.168.1.10 -p 22,80,443
```

---

## Set batch size

```bash
rustscan -a 192.168.1.10 -b 1000
```

---

## Increase timeout

```bash
rustscan -a 192.168.1.10 --timeout 2000
```

---

## Run Nmap automatically

```bash
rustscan -a 192.168.1.10 -- -sV
```

Everything after `--` is passed to Nmap.

---

# Common Nmap Integration

## Service detection

```bash
rustscan -a 192.168.1.10 -- -sV
```

---

## Default scripts

```bash
rustscan -a 192.168.1.10 -- -sC
```

---

## OS detection

```bash
rustscan -a 192.168.1.10 -- -O
```

---

# Useful Flags

| Flag | Description |
|---|---|
| -a | Target |
| -p | Ports |
| -b | Batch size |
| --timeout | Timeout |
| --ulimit | File descriptor limit |
| -- | Pass arguments to Nmap |

---

# Notes
- RustScan is optimized for speed.
- Large scans may trigger IDS/IPS alerts.
- Use responsibly and only with authorization.
