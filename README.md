<div align="center">

```
 ██████╗████████╗██████╗ ███████╗    ███╗   ██╗ ██████╗ ████████╗███████╗███████╗
██╔════╝╚══██╔══╝██╔══██╗██╔════╝    ████╗  ██║██╔═══██╗╚══██╔══╝██╔════╝██╔════╝
██║        ██║   ██████╔╝███████╗    ██╔██╗ ██║██║   ██║   ██║   █████╗  ███████╗
██║        ██║   ██╔═══╝ ╚════██║    ██║╚██╗██║██║   ██║   ██║   ██╔══╝  ╚════██║
╚██████╗   ██║   ██║     ███████║    ██║ ╚████║╚██████╔╝   ██║   ███████╗███████║
 ╚═════╝   ╚═╝   ╚═╝     ╚══════╝    ╚═╝  ╚═══╝ ╚═════╝    ╚═╝   ╚══════╝╚══════╝
```

# HTB CPTS — Study Notes

### *by [Omar Almasri](https://github.com/OmarAlmasri)*

<br>

![HTB](https://img.shields.io/badge/Platform-Hack%20The%20Box-9fef00?style=for-the-badge&logo=hackthebox&logoColor=black)
![CPTS](https://img.shields.io/badge/Certification-CPTS-9fef00?style=for-the-badge&logo=hackthebox&logoColor=black)
![Status](https://img.shields.io/badge/Status-In%20Progress-orange?style=for-the-badge)
![Notes Type](https://img.shields.io/badge/Notes-Module%20Content-blue?style=for-the-badge)
![License](https://img.shields.io/badge/License-Attribution%20Required-red?style=for-the-badge)

</div>

---

## 📌 What Is This Repo?

This repository contains my personal study notes for the **Hack The Box Certified Penetration Testing Specialist (HTB CPTS)** certification path, organized module by module as I work through the curriculum.

> **These are NOT cheatsheets.**
>
> The content in this repository is **copied directly from HTB Academy modules** as part of my active learning process. I take these notes to reinforce concepts, build structured reference material, and support my exam preparation — not to summarize or shortcut the material.

If you're looking for condensed cheatsheets, they **may be added later** if I complete the path ahead of schedule. Until then, what you find here is raw module content structured for study purposes.

---

## ⚠️ Disclaimer

- These notes are **not professional-grade documentation**. They are a personal learning resource, written and maintained at an intermediate skill level.
- Do **not** treat these notes as authoritative references for real-world engagements without independently verifying each technique.
- HTB Academy's original course material is the true source of truth — always refer back to it.
- If any content here was inadvertently misrepresented or simplified, it's the nature of live note-taking, not intentional misinstruction.

---

## 📂 Path Coverage

The [HTB Penetration Tester Job Role Path](https://academy.hackthebox.com/) consists of **28 modules** spanning the full penetration testing lifecycle — from pre-engagement and reconnaissance all the way through to Active Directory attacks, reporting, and attacking enterprise networks.

Modules are organized by phase/domain as covered in the path. Progress will be tracked below.

| #   | Module                                 | Status |
| --- | -------------------------------------- | ------ |
| 01  | Penetration Testing Process            | ✅      |
| 02  | Getting Started                        | ✅      |
| 03  | Network Enumeration with Nmap          | ✅      |
| 04  | Footprinting                           | ✅      |
| 05  | Information Gathering - Web Edition    | ✅      |
| 06  | Vulnerability Assessment               | ✅      |
| 07  | File Transfers                         | ✅      |
| 08  | Shells & Payloads                      | ✅      |
| 09  | Using the Metasploit Framework         | ✅      |
| 10  | Password Attacks                       | ✅      |
| 11  | Attacking Common Services              | ✅      |
| 12  | Pivoting, Tunneling & Port Forwarding  | 🔄     |
| 13  | Active Directory Enumeration & Attacks | ⬜      |
| 14  | Using Web Proxies                      | ⬜      |
| 15  | Attacking Web Applications with Ffuf   | ⬜      |
| 16  | Login Brute Forcing                    | ⬜      |
| 17  | SQL Injection Fundamentals             | ⬜      |
| 18  | SQLMap Essentials                      | ⬜      |
| 19  | Cross-Site Scripting (XSS)             | ⬜      |
| 20  | File Inclusion                         | ⬜      |
| 21  | File Upload Attacks                    | ⬜      |
| 22  | Command Injections                     | ⬜      |
| 23  | Web Attacks                            | ⬜      |
| 24  | Attacking Common Applications          | ⬜      |
| 25  | Linux Privilege Escalation             | ⬜      |
| 26  | Windows Privilege Escalation           | ⬜      |
| 27  | Documentation & Reporting              | ⬜      |
| 28  | Attacking Enterprise Networks          | ⬜      |

> **Legend:** ✅ Done &nbsp;|&nbsp; 🔄 In Progress &nbsp;|&nbsp; ⬜ Not Started

---

## 🧠 About the Author

I'm a professional penetration tester on a journey toward becoming a Red Team Operator. My day-to-day spans external and internal network assessments, client advisory work, and offensive security research. This repo is part of my structured approach to building deep, documented knowledge as I work through the CPTS path.

> *"Taking notes is how I learn. Sharing them is how I contribute."*

---

## 📜 Redistribution & Attribution

These notes are shared openly for the benefit of the community. If you wish to use, redistribute, or build upon this material in any form — including personal notes, blog posts, YouTube videos, or other public repos — **you must credit the original author**:

```
Original notes by Omar Almasri — https://github.com/OmarAlmasri/HTB-CPTS-Notes
```

Attribution isn't just a formality. It's respect for the time and effort that goes into building structured learning material. Please don't strip authorship when sharing.

> All original course content belongs to **Hack The Box Academy**. This repo makes no claim of ownership over HTB's intellectual property.

---

## 🔗 Useful Resources

- 🌐 [HTB Academy — Penetration Tester Path](https://academy.hackthebox.com/path/preview/penetration-tester)
- 🎓 [HTB CPTS Certification Info](https://academy.hackthebox.com/preview/certifications/htb-certified-penetration-testing-specialist)
- 💬 [HTB Discord Community](https://discord.gg/hackthebox)
- 📖 [HTB Blog — CPTS Launch Article](https://www.hackthebox.com/blog/certified-penetration-testing-specialist-cpts)

---

<div align="center">

*Good notes aren't written once — they're refined forever.*

**⭐ Star the repo if it helps your journey**

</div>
