# 🔴 Advanced Malware Research Project
## NT140.Q13.ANTT - Group 12

> **⚠️ EDUCATIONAL PURPOSE ONLY**  
> This project is developed for academic research and cybersecurity education as part of the NT140 Network Security course. **NEVER deploy this software on systems you do not own or have explicit permission to test.**

---

## 📋 Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Components](#components)
- [Attack Chain](#attack-chain)
- [Build Instructions](#build-instructions)
- [Testing Environment](#testing-environment)
- [Safety Guidelines](#safety-guidelines)
- [Technical Details](#technical-details)
- [Legal Disclaimer](#legal-disclaimer)

---

## 🎯 Overview

This project demonstrates a complete **Advanced Persistent Threat (APT)** simulation, showcasing multiple sophisticated malware techniques including:

- **Network Worm Propagation** via SMB vulnerabilities
- **Logic Bomb** with conditional triggers
- **Trojan Dropper** with payload delivery
- **Command & Control (C&C)** infrastructure
- **Botnet** with cryptojacking capabilities
- **Destructive Payload** (Wiper)
- **Stealth Techniques** (File hiding, LNK traps, privilege escalation)
- **Persistence Mechanisms** (Scheduled tasks)

### 🎓 Educational Objectives
- Understanding modern malware architecture
- Learning attack vectors and propagation methods
- Studying C&C communication protocols
- Analyzing persistence and evasion techniques
- Exploring system destruction mechanisms

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         ATTACK FLOW                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. Initial Infection (Worm.exe)                                 │
│     │                                                             │
│     ├─► SMB Network Scan                                         │
│     ├─► Credential-based Propagation                             │
│     ├─► LNK Trap Generation (.docx.lnk)                          │
│     └─► Payload Deployment                                       │
│                                                                   │
│  2. Conditional Trigger (LogicBomb.exe)                          │
│     │                                                             │
│     ├─► Monitor Windows Defender Status                          │
│     ├─► Wait for "Defender Disabled" (3 checks)                  │
│     └─► Decrypt & Launch Trojan                                  │
│                                                                   │
│  3. Payload Delivery (Trojan.exe)                                │
│     │                                                             │
│     ├─► Scan Network for C&C Server                              │
│     ├─► Download payload.zip                                     │
│     ├─► Extract BotClient + Wiper                                │
│     └─► Install Persistence (Scheduled Tasks)                    │
│                                                                   │
│  4. Botnet Operation (BotClient.exe)                             │
│     │                                                             │
│     ├─► Connect to C&C Server                                    │
│     ├─► Await Commands (idle/cryptojack/wipe/recon)             │
│     ├─► Execute Cryptojacking (SHA-256 mining)                   │
│     ├─► Monitor Defender (Logic Bomb #2)                         │
│     └─► Launch Wiper if Triggered                                │
│                                                                   │
│  5. Command & Control (C&CServer + AttackerControlPanel)         │
│     │                                                             │
│     ├─► Track Active Bots                                        │
│     ├─► Issue Commands                                           │
│     ├─► Collect Mining Results                                   │
│     └─► Receive Recon Reports                                    │
│                                                                   │
│  6. System Destruction (Wiper.exe)                               │
│     │                                                             │
│     ├─► Delete Critical System Files                             │
│     ├─► Corrupt Disk Structures (MBR/GPT)                        │
│     ├─► Destroy User Data                                        │
│     └─► Trigger BSOD                                             │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🧩 Components

### 🔧 **Builder.exe**
**Purpose:** Automated build system  
**Features:**
- Compiles all projects (Worm, LogicBomb, Trojan, BotClient, C&C Server, etc.)
- Generates random encryption keys
- Encrypts Trojan.exe → `bomb.encrypted`
- Creates two output folders:
  - `product/` - Contains Worm deployment package
  - `attack/` - Contains attacker toolkit (C&C Server + Control Panel)

**Output Structure:**
```
Builder/
├── product/
│   ├── Worm.exe
│   ├── SharedCrypto.dll
│   └── payload/
│       ├── LogicBomb.exe
│       ├── bomb.encrypted
│       ├── key.dat
│       └── SharedCrypto.dll
└── attack/
    ├── C&CServer.exe (+ dependencies)
    ├── control panel/
    │   └── AttackerControlPanel.exe
    └── wwwroot/
        └── payload.zip (contains BotClient + Wiper)
```

---

### 🪱 **Worm.exe**
**Purpose:** Self-propagating network worm  
**Propagation Method:** SMB file shares with hardcoded credentials

**Key Features:**
- **Network Scanning:** Ping sweep on local subnet (x.x.x.1-254)
- **SMB Authentication:** Connects using `user:user` credentials
- **Payload Delivery:** Copies entire malware package to `\\target\SharedFolder`
- **Remote Execution:** Uses WMI to execute Worm.exe on target
- **Re-encryption:** Generates new encryption key for each victim
- **LNK Traps:** Creates malicious shortcuts disguised as documents
- **Stealth:** Hides console window, marks files as Hidden+System
- **Persistence:** Creates scheduled task `Malicious_Worm` (SYSTEM-level, runs at startup)

**Hardcoded Credentials:**
```
Username: user
Password: user
Share: SharedFolder
```

---

### 💣 **LogicBomb.exe**
**Purpose:** Conditional trigger mechanism  
**Trigger Condition:** Windows Defender Real-Time Protection disabled for 30+ seconds (3 consecutive checks at 10s intervals to avoid false positive)

**Key Features:**
- Monitors Real-time Protection status via WMI + Registry
- Loads decryption key from `key.dat`
- Decrypts `bomb.encrypted` → `Trojan.exe`
- Launches Trojan payload
- **Self-destruct:** Deletes encrypted payload, key file, and itself
- Removes scheduled task `Malicious_LogicBomb`

---

### 🐴 **Trojan.exe**
**Purpose:** Secondary dropper and payload delivery system

**Key Features:**
- **C&C Discovery:** Scans local network (port 8000) to find C&C Server
- **Payload Download:** Fetches `payload.zip` from `http://[server]:8000/payload.zip`
- **Extraction:** Unpacks BotClient.exe and Wiper.exe
- **Persistence:** Creates scheduled tasks for all executables in payload
- **Immediate Execution:** Launches payloads in background
- **Self-deletion:** Removes itself after deployment

---

### 🤖 **BotClient.exe**
**Purpose:** Multi-function bot with C&C communication

**Capabilities:**

#### 1️⃣ Command Execution
- **IDLE:** Stop all tasks, minimize CPU usage
- **CRYPTOJACK:** SHA-256 hash mining (simulated cryptocurrency mining)
- **WIPE:** Trigger destructive Wiper payload
- **RECON:** System reconnaissance and data exfiltration

#### 2️⃣ Cryptojacking
- Adaptive CPU usage (Stealth Mode vs. Aggressive Mode)
  + When CPU usage is low (<50%), enter Stealth mode, mining at reduced speed
  + When CPU usage is high (>=50%), enter Aggressive mode, mining at maximum speed
- Batch processing to avoid detection
- Submits results to C&C Server
- Performance counter monitoring

#### 3️⃣ Logic Bomb (Secondary)
- Monitors Windows Defender status
- 3-strike rule: Launches Wiper if Real-time Protection is back online for 30+ seconds (3 consecutive checks at 10s intervals)

#### 4️⃣ Reconnaissance
- Collects hardware info (CPU, RAM, Disk)
- File system scanning (Desktop, Documents)
- Sends detailed reports to C&C Server

#### 5️⃣ Wiper Trigger
- Launches `Wiper.exe` from `payload/wiper/` subfolder
- Executes with admin privileges (UAC prompt)
- Self-terminates after activation

**Persistence:**
- Single-instance mutex (prevents multiple bots)
- Scheduled task for automatic startup
- File logging to `bot_log.txt`

---

### 🎮 **AttackerControlPanel.exe**
**Purpose:** Interactive command interface for the attacker

**Features:**
- Real-time bot status monitoring
- Command issuance (cryptojack/idle/wipe/recon)
- View mined hashes
- Browse reconnaissance reports
- Safety confirmation for destructive commands

**Menu:**
```
[1] Command: CRYPTOJACK (Start mining)
[2] Command: IDLE (Stop all tasks)
[3] Command: WIPE (!!! DESTROY TARGETS !!!)
[4] Command: RECON (Gather Hardware & Files)
[5] List all active Bots
[6] View collected Results
[7] Refresh Menu
[0] Exit Panel
```

---

### 🖥️ **C&CServer.exe**
**Purpose:** Command & Control server (ASP.NET Core Web API)  
**Port:** 8000

**Endpoints:**
- `POST /bot/checkin` - Bot registration and status updates
- `GET /bot/getcommand` - Retrieve current command
- `POST /bot/setcommand` - Set new command (from Control Panel)
- `GET /bot/list` - List all active bots
- `POST /bot/submitresult` - Submit mining results
- `GET /bot/results` - Retrieve all mining results
- `POST /bot/submitrecon` - Submit recon reports
- `POST /bot/log` - Bot logging endpoint
- `GET /payload.zip` - Payload download (served from `wwwroot/`)

**Data Storage:**
- In-memory bot registry (ConcurrentDictionary)
- File-based result logging (`mined_hashes.txt`)
- Timestamped recon reports (`recon_[IP]_[timestamp].txt`)

---

### 🧨 **Wiper.exe**
**Purpose:** Destructive payload for system destruction

**⚠️ EXTREME DANGER - This component causes PERMANENT DATA LOSS**

**Destruction Phases:**

**Phase 1: Critical System Files**
- `ntoskrnl.exe` (Windows kernel)
- `winload.exe` (Boot loader)
- `hal.dll` (Hardware Abstraction Layer)
- `ntfs.sys` (File system driver)
- Registry hives (SAM, SYSTEM, SOFTWARE)
- Disk drivers (volmgr.sys, partmgr.sys)

**Phase 2: Disk Structure Corruption**
- MBR/GPT destruction via `diskpart clean`
- Partition table wiping

**Phase 3: User Data Destruction**
- `C:\Users` (All user profiles)
- `C:\ProgramData`
- `C:\Windows\System32\config`

**Phase 4: System Crash**
- Triggers Blue Screen of Death (BSOD) via `NtRaiseHardError`
- Kills critical system processes (csrss, smss, wininit)

**Safety Features:**
- `_enableDestruction` flag (set to `false` for simulation mode)
- Detailed logging to `destruction_log.txt`
- Requires Administrator privileges

---

### 🔐 **SharedCrypto.dll**
**Purpose:** Encryption/decryption library

**Features:**
- AES-256 encryption
- SHA-256 key derivation
- Random key generation (GUID-based)
- Separate IV derivation with salt
- File encryption/decryption utilities

**Key Generation:**
```csharp
string key = CryptoUtils.GenerateRandomKey(); // Returns GUID
byte[] aesKey = DeriveKeyFromString(key);     // SHA-256 hash
byte[] iv = DeriveIVFromString(key);          // SHA-256(key + salt)
```

---

## 🔗 Attack Chain

```
┌─────────────────────────────────────────────────────────────────┐
│                    COMPLETE INFECTION TIMELINE                   │
└─────────────────────────────────────────────────────────────────┘

T+0:00   │ Attacker deploys Worm.exe on initial target
         │
T+0:10   │ Worm scans network, finds vulnerable SMB shares
         │
T+0:30   │ Worm propagates to exposed machines
         │ Payload deployed: LogicBomb.exe + bomb.encrypted + key.dat
         │
T+1:00   │ On infected machines, LogicBomb monitors Windows Defender status, waiting for deployment
         │
T+5:00   │ Victim disables Defender to speed up system, or to play a cracked video game
         │
T+5:30   │ LogicBomb triggers after 3 checks (30 seconds)
         │ Decrypts and launches Trojan.exe
         │
T+5:35   │ Trojan scans network, finds C&C Server at 192.168.1.100
         │
T+5:40   │ Trojan downloads payload.zip, extracts and run BotClient + Wiper
         │ Creates persistence via scheduled tasks
         │
T+5:45   │ BotClient connects to C&C Server
         │ Status: "idle" - Awaiting commands
         │
T+10:00  │ Attacker issues "cryptojack" command via Control Panel
         │
T+10:05  │ All bots begin SHA-256 mining
         │ Adaptive mining will determine the mining speed accordingly to avoid suspicion
         │
T+20:00  │ Attacker issues "recon" command
         │ Bots collect system info, file listings
         │
T+20:30  │ Recon reports received on C&C Server
         │ Attacker analyzes victim data
         │
T+30:00  │ User turns Windows Defender back online
         | BotClient detects Defender re-enabled (Strike 1/3)
         │
T+30:30  │ Defender still active (Strike 2/3)
         │
T+31:00  │ Defender still active (Strike 3/3)
         │ → LOGIC BOMB TRIGGERED
         │ → Wiper.exe launched with admin rights
         │
T+31:05  │ Wiper destroys critical system files
         │ MBR/GPT corrupted, user data deleted
         │
T+31:10  │ BSOD triggered on all infected machines
         │ → SYSTEMS DESTROYED
         │
         │ ATTACK COMPLETE
```

---

## 🔨 Build Instructions

### Prerequisites
- **Operating System:** Windows 10/11
- **Development Environment:**
  - Visual Studio 2022 (Community Edition or higher)
  - .NET Framework 4.8 SDK
  - .NET 8.0 SDK
  - MSBuild (included with Visual Studio)
- **NuGet Packages:** (Auto-restored)
  - IWshRuntimeLibrary (COM Interop for LNK creation)
  - ASP.NET Core 8.0 (for C&C Server)

### Step 1: Clone Repository
```bash
git clone https://github.com/CatFishee/GROUP-13_NT140.Q13.ANTT.git
cd GROUP-13_NT140.Q13.ANTT
```

### Step 2: Build with Builder.exe
```bash
cd Builder
dotnet run
```

**Builder will automatically:**
1. Locate MSBuild.exe
2. Generate random encryption key
3. Compile all projects
4. Encrypt Trojan.exe
5. Package everything into `product/` and `attack/` folders

### Step 3: Output Verification
Check that the following folders exist:
```
Builder/
├── product/          ← Worm deployment package
└── attack/           ← Attacker toolkit
```

---

## 🧪 Testing Environment

### ⚠️ MANDATORY SAFETY REQUIREMENTS

**NEVER test this malware on:**
- Production systems
- Systems you don't own
- Networks without explicit permission
- Your main computer

**ONLY test in:**
- Isolated virtual machines (VMs)
- Air-gapped lab networks
- Sandboxed environments

---

### Recommended Lab Setup

#### Network Topology
```
┌─────────────────────────────────────────────────────────┐
│                  ISOLATED VIRTUAL NETWORK                │
│                     (No Internet Access)                 │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  ┌──────────────┐      ┌──────────────┐                 │
│  │ Attacker VM  │      │  Victim VM 1 │                 │
│  │              │      │              │                 │
│  │ C&C Server   │◄────►│ Target       │                 │
│  │ Control Panel│      │ (Worm Entry) │                 │
│  └──────────────┘      └──────────────┘                 │
│         │                      │                         │
│         │                      │                         │
│         ▼                      ▼                         │
│  ┌──────────────┐      ┌──────────────┐                 │
│  │ Network      │      │  Victim VM 2 │                 │
│  │ Switch       │◄────►│              │                 │
│  │ (vSwitch)    │      │ Propagation  │                 │
│  └──────────────┘      │ Target       │                 │
│                        └──────────────┘                 │
│                                                           │
└─────────────────────────────────────────────────────────┘
```

#### VM Specifications (Minimum)

**Attacker Machine:**
- OS: Windows 10/11 Pro
- RAM: 4GB
- Disk: 50GB
- Network: NAT + Host-Only Adapter
- Software: .NET Runtime, C&C Server, Control Panel

**Victim Machines (We need 2 victims, but only need 1 to setup SMB server):**
- OS: Windows Server 2016 (Or any kind of Windows 10)
- RAM: 2GB each
- Disk: 40GB each
- Network: Host-Only Adapter (same subnet as Attacker)
- Setup:
  - Create local user: `user` / `user`
  - Create SMB share: `C:\SharedFolder` (Full Control for user)
  - Enable remote powershell execution (Enable-PSRemoting -Force to enable) (winrm quickconfig to check)
  - Place sample documents (.docx, .pdf, .txt) in shared folders

#### Network Configuration 
- **Subnet:** 192.168.56.0/24
- **Attacker IP:** 192.168.56.100
- **Victim 1 IP:** 192.168.56.101 (SMB server)
- **Victim 2 IP:** 192.168.56.102
- **No Gateway** (Isolated network)
- **No DNS** (Prevent external communication)
- (IPs are recommended but not enforced, since the virus does not look for hard-coded IPs, just make sure all the VMs are in the same network)
---

### Testing Procedure

#### Phase 1: Setup C&C Infrastructure
1. On Attacker VM, extract `attack/` folder
2. Run `C&CServer.exe` (Port 8000 should open)
3. Run `AttackerControlPanel.exe`
4. Verify Control Panel connects to server

#### Phase 2: Initial Infection
1. On Attacker VM, extract `product/` folder
2. Execute `Worm.exe` as current user, NOT admin to ensure task scheduling and logic bomb fail, so attacker don't get infected by their own virus
3. Monitor `Worm_log_[timestamp].log`
4. Verify network scanning begins

#### Phase 3: Propagation
1. Worm should discover Victim 1's SMB share
2. Check Victim 1: `C:\SharedFolder` should contain Worm package
3. Worm auto-executes on Victim 1 via WMI
4. Check Worm's log inside the shared folder
5. Open Task scheduler to see the new scheduled tasks that worm installed 

#### Phase 4: Logic Bomb Testing
1. Disable Windows Real-time Protection (RTP) on Victim 1
2. Wait 30 seconds
3. `LogicBomb.exe` should decrypt and launch `Trojan.exe`
4. Check `result/` folder for `Trojan.exe` and the logs

#### Phase 5: Botnet Activation
1. Trojan scans network, finds Attacker VM (port 8000)
2. Downloads `payload.zip`, extracts BotClient, install persistence and run it
3. In Control Panel, verify bot appears in List
4. Issue "cryptojack" command
5. Monitor CPU usage on victim machines (should spike)

#### Phase 6: Recon Testing
1. Issue "recon" command from Control Panel
2. Wait 30-60 seconds
3. Check C&C Server folder for `recon_[IP]_[timestamp].txt`
4. Review collected system information

#### Phase 7: Wiper Testing (⚠️ DESTRUCTIVE)
**WARNING:** This will destroy the VM. Take a snapshot first!
1. Take VM snapshot: "Pre-Wiper Test"
2. Issue "wipe" command OR let turn on Window's Real-time Protection to trigger BotClient's logic bomb
3. Observe system destruction
4. VM will crash (BSOD), or it might not if the OS has built-in wiper protection like Window Server, but File Explorer and other vital programs will be deleted
5. Revert to snapshot

#### Extra: SMB Server to Client Worm Propagation
1. On Victim 2's VM, connect to Victim 1's SMB sharedfolder
2. Try opening one of the sample documents (notice that the name and icon is the same, but file exension and file type is different)
3. An UAC prompt will pop up, and if you agree, the document will open normally while worm and logic bomb run in the background 
---

## 🛡️ Safety Guidelines

### FOR RESEARCHERS AND STUDENTS

#### ✅ DO:
- Run ONLY in isolated VMs
- Use air-gapped networks
- Take frequent snapshots
- Document all findings
- Disable network adapters after testing
- Delete all malware artifacts when finished

#### ❌ DON'T:
- Deploy on physical hardware
- Connect VMs to the Internet
- Share malware binaries publicly
- Use on employer/school networks
- Test without proper authorization
- Underestimate the damage potential

### Emergency Procedures

**If malware escapes the lab:**
1. Immediately disconnect all network cables
2. Power off infected machines
3. Notify IT security team
4. Do NOT attempt to "clean" systems yourself
5. Restore from clean backups

**If Wiper is accidentally triggered:**
- There is NO recovery
- All data will be permanently lost
- Restore from snapshot or reinstall OS

---

## 🔬 Technical Details

### Encryption Scheme
```
Key Generation:
  - Random GUID → "e4f5a8b2-1234-5678-9abc-def012345678"
  
Key Derivation:
  - AES Key = SHA256(GUID) [First 32 bytes]
  - IV = SHA256(GUID + "_IV_SALT_2025") [First 16 bytes]
  
Encryption:
  - Algorithm: AES-256-CBC
  - Input: Trojan.exe (Plaintext)
  - Output: bomb.encrypted (Ciphertext)
  
Unique Per Victim:
  - Worm generates NEW key for each infected machine
  - Stored in payload/key.dat
  - LogicBomb reads key.dat to decrypt
```

### Network Communication
```
Protocol: HTTP/1.1 (REST API)
Port: 8000
Content-Type: application/json

Bot → Server:
  - POST /bot/checkin         (Every 15-30s, randomized)
  - GET /bot/getcommand       (Polling for commands)
  - POST /bot/submitresult    (Mining results)
  - POST /bot/submitrecon     (Recon reports)
  - POST /bot/log             (Status messages)

Attacker → Server:
  - POST /bot/setcommand      (Command issuance)
  - GET /bot/list             (Bot inventory)
  - GET /bot/results          (Mining results)

Bot Identification:
  - IP-based (No BotID in payload, extracted from HTTP request)
```

### Persistence Mechanisms
```
Method: Windows Task Scheduler

Tasks Created:
  1. Malicious_Worm
     - Trigger: At system startup
     - User: SYSTEM
     - Privilege Level: Highest
     - Action: Run Worm.exe
  
  2. Malicious_LogicBomb
     - Trigger: At system startup
     - User: SYSTEM
     - Privilege Level: Highest
     - Action: Run LogicBomb.exe
  
  3. Malicious_BotClient (created by Trojan)
     - Trigger: At system startup
     - User: SYSTEM
     - Privilege Level: Highest
     - Action: Run BotClient.exe [C&C_IP]

Removal Requirement:
  - Administrative privileges
  - Manual task deletion via Task Scheduler
  - Or: schtasks.exe /Delete /TN [TaskName] /F
```

### Stealth Techniques
```
1. Console Hiding:
   - ShowWindow(GetConsoleWindow(), SW_HIDE)
   - All executables run with no visible window
   
2. File Attributes:
   - Malware files marked as Hidden + System
   - Explorer.exe hides them by default
   - Requires "Show hidden files" + "Show system files"
   
3. LNK Traps:
   - Original document hidden (Hidden attribute)
   - Malicious .lnk created with same name + icon
   - User double-clicks .lnk → Worm executes → Document opens
   - User sees document, unaware of infection
   
4. Mutex (Single Instance):
   - BotClient uses Global\\MyMalwareBot_Unique_Mutex_String
   - Prevents multiple instances
   - Avoids high CPU usage from duplicate processes
   
5. Adaptive CPU Usage:
   - Cryptojacking monitors system CPU
   - High system load → Aggressive mining (100k hashes/burst)
   - Low system load → Stealth mining (10k hashes/burst)
   - Makes detection harder
```

### Logic Bomb Conditions

**Trigger 1 (LogicBomb.exe):**
```
Condition: Windows Defender Real-Time Protection = Disabled
Duration: 30 seconds (3 consecutive checks @ 10s intervals)
Action: Decrypt bomb.encrypted → Launch Trojan.exe → Self-destruct
```

**Trigger 2 (BotClient.exe):**
```
Condition: Windows Defender Real-Time Protection = Enabled
Duration: 30 seconds (3 consecutive checks @ 10s intervals)
Logic: If Defender detected 3 times → System is being monitored
Action: Launch Wiper.exe → Destroy evidence
```

**Detection Methods:**
1. WMI Query: `root\Microsoft\Windows\Defender → MSFT_MpComputerStatus.RealTimeProtectionEnabled`
2. Registry Fallback: `HKLM\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection\DisableRealtimeMonitoring`

---

## 📊 Project Statistics

- **Total Components:** 9
- **Lines of Code:** ~3,500+
- **Languages:** C# (.NET Framework 4.7.2, .NET 8.0)
- **Attack Vectors:** SMB, WMI, LNK Social Engineering
- **Persistence Methods:** Scheduled Tasks (SYSTEM-level)
- **Encryption:** AES-256-CBC
- **C&C Protocol:** HTTP REST API
- **Destructive Capability:** EXTREME (Wiper can destroy OS)

---

## 📚 Learning Resources

### Recommended Reading
- **MITRE ATT&CK Framework:** Tactics and techniques used
  - T1021.002: SMB/Windows Admin Shares
  - T1047: Windows Management Instrumentation
  - T1053.005: Scheduled Task/Job
  - T1027: Obfuscated Files or Information
  - T1485: Data Destruction
  - T1496: Resource Hijacking (Cryptojacking)

### Related CVEs
- CVE-2017-0143 (EternalBlue) - SMB exploitation
- CVE-2020-0796 (SMBGhost) - SMBv3 RCE

### Notable Real-World Malware
- **WannaCry** (Worm + Ransomware)
- **NotPetya** (Wiper disguised as ransomware)
- **Mirai** (IoT Botnet)
- **Zeus** (Banking Trojan with C&C)

---

## ⚖️ Legal Disclaimer

```
┌─────────────────────────────────────────────────────────────────┐
│                    LEGAL WARNING                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  This software is provided for EDUCATIONAL AND RESEARCH          │
│  PURPOSES ONLY as part of the NT140 Network Security course.    │
│                                                                   │
│  UNAUTHORIZED USE OF THIS SOFTWARE IS STRICTLY PROHIBITED        │
│  and may violate:                                                │
│                                                                   │
│  • Computer Fraud and Abuse Act (CFAA) - USA                    │
│  • Computer Misuse Act - UK                                      │
│  • Cybercrime Convention - International                         │
│  • Local laws in your jurisdiction                              │
│                                                                   │
│  The authors and contributors assume NO LIABILITY for misuse     │
│  of this software. By using this code, you agree to:            │
│                                                                   │
│  1. Use ONLY in isolated lab environments                       │
│  2. Obtain proper authorization before testing                  │
│  3. Not deploy on systems you don't own                         │
│  4. Accept full legal responsibility for your actions           │
│                                                                   │
│  VIOLATORS WILL BE PROSECUTED TO THE FULLEST EXTENT OF LAW      │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 👥 Contributors

**Group 12 - NT140.Q13.ANTT**  
Course: Network Security (NT140)  
Institution: University of Information Technology (UIT)  
Academic Year: 2025-2026

---

## 📞 Contact

For academic inquiries only:
- GitHub: [CatFishee/GROUP-12_NT140.Q13.ANTT](https://github.com/CatFishee/GROUP-12_NT140.Q13.ANTT)
- Course: NT140.Q13.ANTT
- Gmail: thienphubui2803@gmail.com, giangcam2005@gmail.com

**DO NOT contact for:**
- Help deploying malware
- Bypassing security systems
- Illegal activities

---


## 🙏 Acknowledgments

- NT140 Course Instructors
- Cybersecurity research community
- Open-source security tools
- MITRE ATT&CK Framework

---

<div align="center">

**⚠️ USE RESPONSIBLY - EDUCATION ONLY ⚠️**

*"With great power comes great responsibility"*

</div>
