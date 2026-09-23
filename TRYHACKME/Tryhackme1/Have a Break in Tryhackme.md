Breaking down "Have a Break" — TryHackMe OSINT Writeup

Executive Summary

This writeup walks through the TryHackMe **Have a Break** challenge. The objective is to investigate the insider-facilitated theft of a major shipment of KitKat products en route from Italy to Poland. By combining digital forensics, log analysis, email header evaluation, and Open Source Intelligence (OSINT), we trace the culprit's trail from an anonymous email back to their true identity.

---

Challenge Briefing

Background

- **Incident Date:** 26 March 2026
- **The Consignment:** 413,793 units of Nestlé KitKat confectionery (approx. 12 tonnes)
- **The Route:** Central Italy to Poland via Austria and the Czech Republic
- **The Subcontractor:** TransEuro Logistics s.r.o. (Brno Office)
- **The Core Clue:** An anonymous insider email leaked to a local media outlet (_Brno Regional News_) claiming internal system sabotage the night before departure.

---

🛠️ Toolset & Techniques

- **File & Type Analysis:** `file`, `ls`
- **Text Extraction & Filtering:** `cat`, `grep`
- **Network & Infrastructure Intelligence:** `whois`
- **Identity OSINT Search Engines:** Epieos / GHunt

---

🧭 Step-by-Step Investigation Workflow

Step 1: Initial Decompression and Extraction

We download the challenge package, verify the file structure, and decompress the archive to access our raw evidence.

bash

```
# Verify the file type
file ecta-case-1775237862109.zip

# Unzip the contents
unzip ecta-case-1775237862109.zip

# List the extracted investigative files
ls -l ~/Downloads/THM
```

Use code with caution.

_Output files found:_ `ecta_memo.html.pdf`, `exhibit_a.eml`, `exhibit_b.png`, and the directory `transeuro_data/`.

![[Pasted image 20260707133341.png]]
---

Step 2: Email Header Forensics (`Exhibit A`)

The anonymous tip received by the journalist is saved as an email file (`exhibit_a.eml`). We inspect its raw parameters using `cat` to trace the email source infrastructure.

bash

```
cat exhibit_a.eml
```

Use code with caution.

Key Findings from Header Triangulation:

- **The Originating IP Address:** Hidden inside the deepest `Received:` hop, we locate the client IP address:
    
    text
    
    ```
    Received: from () by smtp.gmail.com ...
    ```
    
    Use code with caution.
    
- **The Timestamp:** Thu, 27 Mar 2026 23:14:47 +0100

We pivot to the terminal to discover the provider hosting this IP block:

bash

```
whois 193.32.249.132
```

Use code with caution.

- **WHOIS Results:** The address routes directly to **31173 Services AB** located in Amsterdam, Netherlands. This provider infrastructure exclusively hosts **Mullvad VPN** exit nodes.

---

Step 3: Server Access Log Analysis

We move into the subpoenaed directory (`transeuro_data/`) to parse company infrastructure interactions and cross-reference our VPN timeline.

bash

```
cd transeuro_data/
cat access_log.csv
```

Use code with caution.

Correlating the Malicious Timeline:

Looking for anomalous activity the night before the vehicle departed (March 25th), we find a high-severity indicator:

text

```
2026-03-25,22:14:09,BR-0291,ROUTE_IT_PL_Q1_2026.pdf,EXPORT
```

Use code with caution.

- **Suspicious Timestamp:** `22:14:09`
- **The Action:** An un-vetted file `EXPORT` of the exact distribution corridor map (`ROUTE_IT_PL_Q1_2026.pdf`).
- **The Offending Employee Account:** **`BR-0291`**

Identifying the Whistleblower:

The challenge also tracks who discovered this leak. Later that night, a separate employee logged in to manage scheduling:

text

```
2026-03-25,23:41:17,BR-0312,DRIVER_SCHEDULE_WK13.xlsx,EDIT
```

Use code with caution.

Account **`BR-0312`** spotted the irregularities and sent the initial defensive anonymous tip.

---

Step 4: Internal Communications & Operational Intel

We audit `comms_export.txt` to uncover any personal markers or security notifications tied to account `BR-0291`.

bash

```
cat comms_export.txt
```

Use code with caution.

An entry logged on **2026-03-24 at 09:11** by the IT systems administrator flags a critical corporate security policy breach:

> _"This morning a request was received from an external address (**kraliknovak09@gmail.com**) to access files in the route planning shared folder. The request was blocked."_

Cross-checking `employees.csv` confirms that **`BR-0291`** handles the precise role of **Route Planner** in the Brno operational hub.

---

Step 5: Unmasking the Identity via OSINT

To tie everything together, we take the non-corporate email target discovered in the chat history (`kraliknovak09@gmail.com`) and run an active Google Profile OSINT lookup using **Epieos** or **GHunt**:

bash

```
# Terminal alternative using GHunt
ghunt email kraliknovak09@gmail.com
```

Use code with caution.

The live metadata query resolves the linked Google profile name immediately to the true actor profile.

---

🏆 Challenge Solutions & Flag Parameters

- **Which VPN service was used to send the anonymous email?**  
    `Mullvad VPN`
- **At what time did the suspicious action take place on March 25th, 2026?**  
    `22:14:09`
- **What is the employee ID of the person who sent the anonymous email?**  
    `BR-0312`
- **What is the employee ID of the employee responsible for leaking the shipment details?**  
    `BR-0291`
- **What is the leaker's full name?**  
    `Radovan Blšťák`

---

🎯 Key Takeaways & Defensive Lessons

1. **Header Analysis Integrity:** EML headers provide immutable path validation. Even when attackers tunnel traffic through anonymous VPN routes, corporate logs capture the baseline operational anomalies.
2. **Inspecting Infiltration Footprints:** Attackers often test access limits with personal accounts (`kraliknovak09@gmail.com`) before shifting to compromised credentials over a VPN.
3. **Data Loss Prevention (DLP):** Restricting and monitoring `EXPORT` rights on critical planning PDF templates can disrupt operations before material loss happens.
