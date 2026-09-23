# 🏆 FIRST PLACE WINNER IN CTFZone 2026! 🥇


# 🏴‍☠️ CTF Event: Pirates of the CyberBean
* **Format:** Jeopardy
* **Type:** Private / Solo
* **Date / Time:** Jul 18, 2026, 11:00 AM
* **Status:** 🏁 Ended
* **Participation:**  Registered Players

---

### 📊 Performance Summary (6 Challenges Documented)

| Challenge Name             | Category             | Points             | Level  | Solves              | Status / Score     | Author    |
| :------------------------- | :------------------- | :----------------- | :----- | :------------------ | :----------------- | :-------- |
| **1. Morse Mystery**       | Crypto               | 100/100            | Easy   | 8 solves            | ✅ Solved (100%)    | FRANK     |
| **2. Plain Sight**         | Reverse Engineering  | 120/120            | Easy   | 10 solves           | ✅ Solved (100%)    | FRANK     |
| **3. Office USB**          | MISC / Forensics     | 150/150            | Medium | 8 solves            | ✅ Solved (100%)    | FRANK     |
| **4. Layer By Layer**      | Reverse Engineering  | 200/200            | Medium | Automated           | ✅ Solved (100%)    | FRANK     |
| **5. Lanes of History**    | OSINT                | 100/100            | Easy   | 8 solves            | ✅ Solved (100%)    | FRANK     |
| **6. NEXORA Technologies** | Digital Forensics    | 200/200            | Medium | 10 solves           | ✅ Solved (100%)    | FRANK     |
| **Overall CTF Profile**    | **5 Key Categories** | **870 / 1390 PTS** | mixed  | FIST 1 PLACE WINNER | **6/10 Completed** | **FRANK** |


1. Morse Mystery 

Category: cyrpto
Leval: easy
Point: 100

challenge discription:
### Description

We intercepted a Morse code transmission. Decode it to recover the flag. -.-. - ..-. --.. --- -. . -- ----- .-. ... ...-- ..--.- -.-. ----- -.. ...-- ..--.- -.-. .-. ....- -.-. -.- ...-- -..

how did i solve this challenge
This is moscode challenge

Tools:

1. Cyberchef
2. kali linux

![[Pasted image 20260718233958.png]]

![[Pasted image 20260718234037.png]]





2. Plain Sight

- **Category:** Reverse Engineering
- **Level:** Easy
- **Points:** 120

**Challenge Description**

A forgotten authentication program was recovered from an old system. The password is the key to unlocking the secret, but it was carefully hidden inside the binary. Can you reverse the program and recover the hidden password?


**How I Solved It**

1. **Extraction:** I downloaded `file.zip` and unzipped it to get the `hidden_password` folder.


  ![[Pasted image 20260718235838.png]]


2. **First Attempt:** I tried running `strings hidden_password | grep -i "ctfzone"` but found nothing.
3. **The Breakthrough:** I used `ltrace ./hidden_password` to analyze the binary dynamically.


   ![[Pasted image 20260719000136.png]]
4. **Exploitation:** When it asked for the password, I put `AAAAAAAAAAAAAAAA` (16 characters long since that was the required string length), and `ltrace` revealed the exact hidden password in memory.


**Tools Used**

- Linux Terminal
- ltrace

**Lesson Learned**

Static analysis tools like `strings` can fail when a binary hides its data. Dynamic analysis using `ltrace` makes it easy to intercept keys directly during memory comparison.





  3. CTF Writeup: Office USB

- **Category:** MISC / Digital Forensics
- **Level:** Medium
- **Points:** 150 pts


Challenge Description

An employee at Acme Corp is suspected of leaking sensitive financial data. A USB drive image has been recovered from Jason Holloway's desk (Finance Dept). Your task is to investigate the USB image and recover what is hidden.


What I Learned

- How to use **Sleuth Kit (TSK)** to extract hidden and deleted files from a raw FAT32 disk image.
- How to stitch together clues from internal corporate documents to build a custom password.
- How to recognize and decode standard cryptographic obfuscation like **ROT13**.

![[Pasted image 20260719083235.png]]

Tools Used

- `binwalk` – For initial file carving and detecting embedded zip structures.
- `fls` – For listing the filesystem directory and finding file inodes.
- `icat` – For extracting the exact files from the raw image.
- `unzip` – For unlocking the final protected archive.


How I Solved It

1. Digging Into the USB

I started by scanning the `usb.img` file with `binwalk` and discovered a hidden, encrypted ZIP file. To explore further, I switched to the Sleuth Kit tools to list the entire FAT32 directory structure:

bash

```
fls -f fat32 -r -p usb.img
```

Use code with caution.

This revealed several internal company documents, which I extracted using `icat`.
![[Pasted image 20260719083322.png]]

2. Gathering the Clues

By reading through the recovered files, I gathered the pieces needed to build a master password based on a corporate policy template (`[PROJECT_REFERENCE]_[EMPLOYEE_ID]_[SECRET_PHRASE]`):

- **Project Reference:** Found inside `Q4_budget_draft.txt` → **`ACME-Q4-NF`**
- **Employee ID:** Found Jason's ID inside `team_directory.txt` → **`FIN-4201`**
- **Secret Phrase:** Found an obfuscated string `qbag_sbetrg_gb_oerngur` inside Jason's carved notes. Decoding it from **ROT13** gave → **`dont_forget_to_breathe`**

3. Cracking the Zip

I combined the three pieces with underscores to form the full password and unlocked the core archive:

![[Pasted image 20260719083430.png]]

bash

```
unzip -P "ACME-Q4-NF_FIN-4201_dont_forget_to_breathe" financial_data.zip
```

Use code with caution.

This extracted `flag.txt`, giving me the final flag!
![[Pasted image 20260719083118.png]]

**Flag:** `ctfzone{usb_f0r3ns1cs_1nv3st1g4t10n_r3c0v3r_d3l3t3d_d4t4_d3c0d3_r0t13_unl0ck_z1p_g3t_fl4g}`



---


## 4. NEXORA Technologies - Corporate Data Breach

- **Category:** Digital Forensics / Network Analysis
- **Level:** Medium
- **Points:** 200 pts
- **Solves:** 10 solves

### Challenge Description
An employee workstation is suspected of communicating with an internal host to exfiltrate corporate files. Your task is to analyze the provided network packet capture (`capture.pcap`), reconstruct the network transmission, bypass any protection layers, and recover the hidden flag.

#### What I Learned
- **Automated Object Extraction:** Using `tshark` object exporting flags to carve active application-layer file payloads directly from pcap structures without heavy GUI utilities.
- **Zip Password Auditing:** Utilizing `zip2john` and the `John the Ripper` cracking engine to perform dictionary-based password auditing against compressed archives.
- **Metadata Forensics & C2PA Manifests:** Parsing deep metadata properties via `exiftool` to investigate automated generation indicators like the IPTC NewsCodes digital source types (`trainedAlgorithmicMedia`).

---

### How I Solved This Challenge

#### 1. Initial Triage & Automated Carving
I unzipped the main file `given.zip` to discover the underlying target capture asset named `capture.pcap`. Rather than loading the capture inside a standard Wireshark viewer, I executed an automated HTTP object-exporting routing command using **`tshark`** to extract file payloads into a clean directory:

![[Pasted image 20260719131329.png]]

```bash
tshark -r capture.pcap --export-objects http,extracted-file
```

Navigating into the `extracted-file` directory revealed two newly recovered file fragments:
* A debug trace file named `'%3fdebug=ctfzone%7BHahahahahhah7D'`
* An encrypted zip archive named `orion.zip`

---

#### 2. Brute-Forcing the Orion Archive
I attempted to run standard unzipping parameters on `orion.zip` but realized that its internal contents (`_Project Orion.pdf`, `financial report.pdf`, and `logo.png`) were password-protected. To recover the passphrase, I extracted the archive's hash structure via `zip2john` and processed it against the standard `rockyou.txt` dictionary using **John the Ripper**:

```bash
zip2john orion.zip > orion_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt orion_hash.txt
```

The cracking utility successfully bypassed the zip wrapping and exposed the plaintext credential instantly:


![[Pasted image 20260719131436.png]]
```text
orion.zip:nexoras::orion.zip
```
**Password Found:** `nexoras`

Using the recovered credential `nexoras`, I unzipped the contents safely into a clean extraction folder named `orion/`.

---

#### 3. Deep Metadata Analysis & Flag Recovery
I navigated inside the extracted `orion/` directory containing the target files:
```bash
cd orion/
ls
# Output: 'financial report.pdf'  logo.png  '_Project Orion.pdf'
```

To extract hidden parameters from the images, I performed a deep inspection of the `logo.png` metadata layout using **`exiftool`**:

![[Pasted image 20260719131517.png]]
```bash
exiftool logo.png
```

The output exposed highly distinct cryptographic and systemic structures:
* **Software Agent Name:** `gpt-image` (version 2.0)
* **Digital Source Type:** `http://cv.iptc.org/newscodes/digitalsourcetype/trainedAlgorithmicMedia` (Indicating the file was an algorithmically generated asset via AI models).
* **C2PA Manifest / Certificate Block:** A large base64-encoded string structure holding organizational verification logs (`Trufo Inc.`, `Trufo C2PA Claim Signing CA`).

Looking closely at the initial file carving outputs and the debug signatures, the flag format parameters combined to give the final solution!

![[Pasted image 20260719131721.png]]

#### Tools Used
- `tshark` (Automated CLI network analysis)
- `John the Ripper` & `zip2john` (Credential cracking engine)
- `exiftool` (Deep metadata parser)

**Flag:** `ctfzone{nex0ra_pcap_f0rens1cs_breach_isolated}`



## 5. Lanes of History

- **Category:** OSINT
- **Level:** Easy
- **Points:** 100 pts

### Challenge Description
The image shows a historic bank facility. Identify the exact building, locate its historical record/article, and determine the original architectural details of its drive-through banking lanes.

#### What I Learned
- **Geointelligence & Landmark Pivoting:** How to use distinct architectural features to pivot search queries and pinpoint obscure historical structures.
- **Historical Record Digging:** Navigating public property archives, historical preservation articles, and local news archives to retrieve specific past architectural specifications.

### How I Solved It

1. **Landmark Identification:** By analyzing the provided image, I identified the structure as the historic **Texas Bank building** located at **281 N Graham Street in Stephenville, Texas**.

2. **Open-Source Intelligence (OSINT) Archival Search:** I conducted a targeted search using the exact address and building name to uncover local historical documentations and architectural articles related to the facility's construction and evolution.

   ![[Pasted image 20260719091530.png]]

3. **Extracting the Clues:** I located the official historical article detailing the facility's legacy. The records explicitly stated that:
   - The motor-bank facility was originally constructed in **1981**.
   - It originally featured **eight drive-through banking lanes** facing west along N. Graham Street.


![[Pasted image 20260719091549.png]]

#### Tools Used
- Google Search / Advanced Dorking
- Google Maps / Street View
- Local Historical Archives

**Flag:** `ctfzone{8}`




6 CTF Writeup: Layer By Layer

- **Category:** Reverse Engineering
- **Level:** Medium
- **Points:** 200 pts


Challenge Description

Layer by layer, check me, and I got a gift for you.


What I Learned

- **Do not trust `strings` outputs blindly:** Challenge authors often include fake flags or honeypots to deceive analysts.
- **Anti-Debugging awareness:** Programs can dynamically change their execution path when they detect debugging or basic string extraction tools.
- **Algebraic Reversing:** How to break down automated byte-level obfuscation loops (`XOR`, `Addition`, `Bit Rotations`) by building an exact mathematical inverse script in Python. 


      #!/usr/bin/env python3

def ror8(val, r_bits):
    """Inafanya rotation ya bits kwenda kulia (Rotate Right 8-bit)."""
    return ((val & 0xff) >> r_bits) | ((val << (8 - r_bits)) & 0xff)

def main():
    # Zile constants 4 kutoka kwenye objdump yako
    constants = [
        0xf466d7c1b52e140, 
        0x136bc12235a1158, 
        0x2fbedd42924247e0, 
        0x37d68382938cd158
    ]

    # Geuza kwenda little-endian byte array (Jumla byte 32)
    target = []
    for c in constants:
        target.extend(list(c.to_bytes(8, byteorder="little")))

    # Invert layers zote tatu nyuma
    flag = ""
    for i in range(len(target)):
        v = (target[i] - 0x2a) & 0xff
        v = v ^ (((7 * i) + 13) & 0xff)
        c = ror8(v, 3)
        flag += chr(c)

    print(f"\n[+] SUCCESS! FLAG HALISI NI: {flag}\n")

if __name__ == "__main__":
    main()


Tools Used

- `unzip` – To extract the initial challenge files.
- `file` – To fingerprint the binary type.
- `strings` – For rapid initial data gathering (which revealed the decoy path).
- `objdump` – To perform static disassembly and extract raw memory constants.
- `python3` – For writing the final custom cryptographic solver script.



How I Solved It

1. Initial Triage & The Trap

I started by extracting the downloaded attachment using the `unzip` command. This gave me an unstripped binary file named `layerbylayer`. I ran the `file` command to inspect it: 
bash

```
file layerbylayer
```

Use code with caution.

The output confirmed it was a standard **64-bit ELF executable**. 

My very first instinct was to run the `strings` command to see if the flag was exposed in cleartext. The binary outputted a flag, but submitting it failed. I quickly realized this was a **fake flag** acting as a decoy routine designed to trick automated scripts and lazy analysts. 

   ![[Pasted image 20260719085612.png]]

2. Static Disassembly with Objdump

Knowing that I could not trust dynamic analysis or basic string scraping, I turned to static disassembly using `objdump` to inspect the underlying validation routine.

I needed to find the hidden data constraints that the program validates inputs against. I ran a targeted regex filter to look for long data movements into the CPU registers:

bash

```
objdump -d layerbylayer | grep -E "movabs|mov.*\$0x"
```

Use code with caution.

This successfully extracted four large 64-bit little-endian constants directly from the assembly:

- `0xf466d7c1b52e140`
- `0x136bc12235a1158`
- `0x2fbedd42924247e0`
- `0x37d68382938cd158`


![[Pasted image 20260719085719.png]]

3. Reversing the Layers with Python

The program encoded characters using a three-layer formula: a left bit rotation (`rol8`), an index-based `XOR`, and an integer `addition`.

To extract the real flag, I mapped the four 64-bit constants into an ordered 32-byte array and wrote a quick Python script to invert every single layer mathematically in reverse order (Subtract `0x2a` -> Reverse `XOR` -> Right bit rotation `ror8`):

```
def ror8(val, r_bits):
    return ((val & 0xff) >> r_bits) | ((val << (8 - r_bits)) & 0xff)

constants = [0xf466d7c1b52e140, 0x136bc12235a1158, 0x2fbedd42924247e0, 0x37d68382938cd158]

target = []
for c in constants:
    target.extend(list(c.to_bytes(8, byteorder="little")))

flag = ""
for i in range(len(target)):
    v = (target[i] - 0x2a) & 0xff
    v = v ^ (((7 * i) + 13) & 0xff)
    flag += chr(ror8(v, 3))

print(flag




---
## 🏁 Conclusion & Lessons Learned

Participating in the **Pirates of the CyberBean** CTF provided great hands-on experience across multiple domains. Key takeaways include:

1. **Look Beyond the Surface:** Static indicators like `strings` can be deceptive honeypots. Deep disassembly (`objdump`) is vital for real validation.
2. **Leverage Dynamic Tracing:** Tools like `ltrace` are highly effective at intercepting keys and hidden data directly from memory during execution.
3. **Structured Forensics:** Combining `binwalk` with **The Sleuth Kit (TSK)** (`fls`, `icat`) makes it easy to carve out deleted artifacts and stitch hidden clues together.

Ultimately, this event proved that combining custom scripting with a solid understanding of low-level system behaviors is far more effective than relying on automated scanners alone.

---

### 💡 Core Values & Driving Philosophy
My journey in cybersecurity is driven by:
* **Consistency:** Compounding knowledge every single day.
* **Curiosity:** Diving deep to understand how systems function underneath.
* **Passion:** Loving the thrill of the hunt and solving complex puzzles.
* **Persistence:** Refusing to quit until the vulnerability is found and the flag is captured.
* **Knowledge Sharing:** Consistently writing and sharing detailed writeups to elevate the community.

**Author:** FRANK  
**Club:** Cyber Club IAA  
**Date:** July 19, 2026
 

