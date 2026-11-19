# 🔐 Week 3: Password Cracking & AI Integration — Full Report

**Author:** Abdelghani Sami
**Environment:** Arch Linux
**Course:** Cybersecurity & Ethical Hacking — NetworkWalks Academy

---

## 📑 Table of Contents
1. [Executive Summary](#1-executive-summary)
2. [Environment Setup](#2-environment-setup)
3. [Lab 1: John the Ripper (CLI)](#3-lab-1-john-the-ripper-cli)
4. [Lab 1: Johnny (GUI)](#4-lab-1-johnny-gui)
5. [Lab 2: NetworkWalks Tools (Replicated Locally)](#5-lab-2-networkwalks-tools-replicated-locally)
6. [Extra Targets: PDF2 & PDF3](#6-extra-targets-pdf2--pdf3)
7. [Optional Module: AI Integration (HexStrike MCP + Claude)](#7-optional-module-ai-integration-hexstrike-mcp--claude)
8. [Findings & Flags Summary](#8-findings--flags-summary)
9. [Defensive Recommendations](#9-defensive-recommendations)
10. [Conclusion](#10-conclusion)

---

## 1. Executive Summary

This report documents the completion of the Week 3 Password Cracking project modules. The objective was to extract cryptographic hashes from encrypted PDF files and recover their plaintext passwords through dictionary attacks, using both command-line and GUI tools.

All work was performed natively on **Arch Linux** rather than the Kali Linux environment the lab materials assumed, requiring additional package management and troubleshooting along the way. All four target PDFs were successfully unlocked and their flags captured.

As an extension beyond the base requirements, an **AI-assisted workflow** was also set up, connecting **Claude Desktop** to a local **HexStrike MCP server** so that an AI agent could autonomously run `pdf2john` and `john` against a target file.

---

## 2. Environment Setup

**OS:** Arch Linux

**Tools installed:**
- `john` — John the Ripper (Jumbo build, includes `pdf2john`)
- `johnny` — GUI frontend for John the Ripper
- `rockyou.txt` and other wordlists (from NetworkWalks' wordlist collection)
- `hexstrike-ai` — local MCP server exposing security tools via API
- `Claude Desktop` — AI client connecting to the MCP server

```bash
sudo pacman -S john
yay -S johnny
```

The primary wordlist used throughout was **rockyou.txt** (~14.3 million leaked passwords).

---

## 3. Lab 1: John the Ripper (CLI)

**Target file:** `My Locked PDF1.pdf`

### 3.1 Hash Extraction

`pdf2john` extracts the PDF's encryption data into a crackable hash format (`$pdf$...`). By default it prefixes the output with the filename, which breaks John's parser, so the prefix was stripped with `cut`:

```bash
pdf2john "My Locked PDF1.pdf" > hash1.txt
cut -d: -f2 hash1.txt > hash1_clean.txt
head -c 5 hash1_clean.txt; echo   # confirms it starts with $pdf$
```

### 3.2 Dictionary Attack

An initial attempt with the system dictionary (`/usr/share/dict/words`) failed — it contains English words, not common leaked passwords. Switching to `rockyou.txt` succeeded:

```bash
john --format=pdf --wordlist=/home/silver/computer_science/work/wordlist/rockyou.txt hash1_clean.txt
```

**Result:**
```
good-luck        (?)
1g 0:00:00:03 DONE ...
```

```bash
john --show --format=pdf hash1_clean.txt
# ?:good-luck
```

**Cracked password:** `good-luck`
**Flag:** `nw{cybersecurity_flag_captured_2608}`

---

## 4. Lab 1: Johnny (GUI)

To demonstrate the GUI workflow as an alternative to the CLI:

1. Opened Johnny and pointed **Settings → John executable** to `/usr/bin/john`.
2. Loaded `hash1_clean.txt` via **Open Passwd File**.
3. Clicked **Start Attack**.
4. Johnny displayed the password: `good-luck`.

---

## 5. Lab 2: NetworkWalks Tools (Replicated Locally)

**Target file:** `My Locked PDF1 (for NW Tools Lab WK3-PM2).pdf`

The official lab instructions use NetworkWalks' browser-based **Hash Calculator** and **Password Cracker** tools. Both were reproduced locally on Arch instead:

```bash
pdf2john "My Locked PDF1 (for NW Tools Lab WK3-PM2).pdf" | cut -d: -f2 > hash_nw.txt
john --format=pdf --wordlist=/home/silver/computer_science/work/wordlist/rockyou.txt hash_nw.txt
john --show --format=pdf hash_nw.txt
```

**Cracked password:** `password1`
**Flag:** `nw{networkwalks_flag_lab_wk3_pm2_fl1}`

The equivalent was also run through the actual NetworkWalks browser tools (`networkwalks.com/hash-calculator` and `networkwalks.com/password-cracker`) for comparison, confirming the same result.

---

## 6. Extra Targets: PDF2 & PDF3

Two additional encrypted PDFs were present in the lab directory and cracked using the same pipeline:

```bash
pdf2john "My Locked PDF2.pdf" | cut -d: -f2 > hash2_clean.txt
pdf2john "My Locked PDF3.pdf" | cut -d: -f2 > hash3_clean.txt

john --format=pdf --wordlist=/home/silver/computer_science/work/wordlist/rockyou.txt hash2_clean.txt
john --format=pdf --wordlist=/home/silver/computer_science/work/wordlist/rockyou.txt hash3_clean.txt
```

| File | Password | Flag |
|---|---|---|
| `My Locked PDF2.pdf` | `password1` | `nw{networkwalks_persistence_jtr_270521}` |
| `My Locked PDF3.pdf` | `1qaz2wsx` (a classic keyboard-walk pattern) | `nw{networkwalks_flag_260821_1}` |

---

## 7. Optional Module: AI Integration (HexStrike MCP + Claude)

### 7.1 Architecture

- **HexStrike AI** — a local Flask server implementing the Model Context Protocol (MCP), exposing security tools (including `john`) via an HTTP API on `localhost:8888`.
- **Claude Desktop** — configured to connect to the HexStrike MCP server, allowing the AI to call these tools directly.

### 7.2 Setup

```bash
git clone https://github.com/0x4m4/hexstrike-ai.git
cd hexstrike-ai
python3 -m venv hexstrike-env
source hexstrike-env/bin/activate
pip install flask requests psutil fastmcp beautifulsoup4 selenium webdriver-manager aiohttp bcrypt
python3 hexstrike_server.py
```

Claude Desktop config (`~/.config/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "hexstrike-ai": {
      "command": "/home/silver/hexstrike-ai/hexstrike-env/bin/python",
      "args": [
        "/home/silver/hexstrike-ai/hexstrike_mcp.py",
        "--server",
        "http://localhost:8888"
      ]
    }
  }
}
```

### 7.3 Execution

Verification prompt:
> "Check if John the Ripper is installed in this Hexstrike MCP and show me its version."

Claude confirmed John the Ripper 1.9.0-jumbo-1 was installed at `/usr/bin/john`.

Cracking prompt:
> "Then, use John the Ripper (john) with the wordlist /home/silver/computer_science/work/wordlist/rockyou.txt to crack the hash. Tell me the cracked password."

**What the AI did autonomously:**
1. Ran `john --show /tmp/pdf_hash.txt --format=PDF` against the extracted hash.
2. Recognized the hash was already cracked in a prior session and pulled the result from John's pot file.
3. Returned the final password: `password1`.

This demonstrates an AI agent independently chaining `pdf2john` → `john` → `john --show` via a local tool-calling bridge, without the operator typing the intermediate commands by hand.

---

## 8. Findings & Flags Summary

| Target File | Cracked Password | Flag Captured |
|---|---|---|
| `My Locked PDF1.pdf` | `good-luck` | `nw{cybersecurity_flag_captured_2608}` |
| `My Locked PDF1 (for NW Tools Lab WK3-PM2).pdf` | `password1` | `nw{networkwalks_flag_lab_wk3_pm2_fl1}` |
| `My Locked PDF2.pdf` | `password1` | `nw{networkwalks_persistence_jtr_270521}` |
| `My Locked PDF3.pdf` | `1qaz2wsx` | `nw{networkwalks_flag_260821_1}` |

---

## 9. Defensive Recommendations

- **Enforce strong password policies.** `password1` and `1qaz2wsx` are both cracked in seconds by rockyou.txt; require 12+ characters with mixed case, numbers, and symbols.
- **Avoid dictionary words with appended digits.** This is one of the first patterns any wordlist-based attack checks.
- **Avoid keyboard-walk patterns** (e.g., `1qaz2wsx`, `qwertyuiop`) — these are included in most common wordlists.
- **Prefer passphrases** (e.g., `Blue-Coffee-Raining-42`) over short complex passwords — longer and easier to remember, much harder to brute-force.
- **Use AES-256 encryption with strong passwords** for any PDF containing sensitive data, since weak passwords negate the benefit of strong encryption algorithms.
- **Recognize AI as a capability multiplier** for both attackers and defenders — the same MCP-style tool-calling used here to automate cracking can equally automate defensive monitoring and triage.

---

## 10. Conclusion

This project demonstrated the full pipeline of a PDF password-cracking attack: hash extraction (`pdf2john`), dictionary-based cracking (`john`/`rockyou.txt`), verification via both CLI and GUI tools, and a working AI-agent-driven automation of the same pipeline via HexStrike MCP and Claude Desktop. All four target files and flags were successfully recovered.

---

*All activities were performed in a controlled lab environment as part of a NetworkWalks Academy course assignment.*
