# PrivEsc Checklists

**Linux PrivEsc:**
[Checklist - Linux Privilege Escalation - HackTricks](https://book.hacktricks.wiki/en/linux-hardening/linux-privilege-escalation-checklist.html)
[Checklist - Linux Privilege Escalation - PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)

**Windows PrivEsc:**
[Checklist - Local Windows Privilege Escalation - HackTricks](https://book.hacktricks.wiki/en/windows-hardening/checklist-windows-privilege-escalation.html)
[Checklist - Local Windows Privilege Escalation - PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)

# Enumeration Scripts

**Linux:**
[LinEnum: Scripted Local Linux Enumeration & Privilege Escalation Checks](https://github.com/rebootuser/LinEnum)
[linuxprivchecker -- a Linux Privilege Escalation Check Script](https://github.com/sleventyeleven/linuxprivchecker)

**Windows:**
[Seatbelt](https://github.com/GhostPack/Seatbelt)
[JAWS - Just Another Windows (Enum) Script](https://github.com/411Hall/JAWS)

# User Privileges

Once we find a particular application we can run with `sudo`, we can look for ways to exploit it to get a shell as the root user. [GTFOBins](https://gtfobins.github.io/) contains a list of commands and how they can be exploited through `sudo`. We can search for the application we have `sudo` privilege over, and if it exists, it may tell us the exact command we should execute to gain root access using the `sudo` privilege we have.

[LOLBAS](https://lolbas-project.github.io/#) also contains a list of Windows applications which we may be able to leverage to perform certain functions, like downloading files or executing commands in the context of a privileged user.

