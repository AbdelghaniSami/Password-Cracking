# 🔐 Password Cracking with John the Ripper & AI Integration

This repository documents my completion of the Week 3 Project Modules for the **Cybersecurity & Ethical Hacking** course at **NetworkWalks Academy**. 

The project covers manual PDF hash extraction, dictionary attacks using John the Ripper, and an advanced Optional Module integrating an AI Agent (Claude Desktop) with a local MCP server (HexStrike) to automate the cracking process.

---

## 🎯 Project Objectives
- Extract `$pdf$` hashes from encrypted PDF files using `pdf2john`.
- Crack hashes using John the Ripper (CLI and Johnny GUI) with dictionary attacks (`rockyou.txt`).
- Replicate NetworkWalks web-based Hash Calculator and Password Cracker tools locally.
- Integrate John the Ripper with an AI Agent using **Claude Desktop** and the **HexStrike MCP Server**.

## 🖥️ Environment
- **OS:** Arch Linux
- **Tools:** John the Ripper (Jumbo), pdf2john, Johnny GUI, Claude Desktop, HexStrike AI MCP Server.
- **Wordlists:** `rockyou.txt`, `JTR_default_password.txt`.

---

## 🚀 Lab Walkthrough & Results

### Lab 1: JTR (CLI & GUI)
Extracted the PDF hash and cracked it using John. Also verified the crack visually using the Johnny GUI.

```bash
pdf2john "My Locked PDF1.pdf" | cut -d: -f2 > hash1.txt
john --format=pdf --wordlist=/path/to/rockyou.txt hash1.txt
