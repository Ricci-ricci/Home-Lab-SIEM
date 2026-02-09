# My Cybersecurity Home Lab Runbook (SIEM + Purple Team)

This repository is my job-focused cybersecurity home lab build journal. I’m documenting everything I do so I can learn practical, “SOC-ready” skills: log collection, detection, investigation, and safe adversary simulation.

> **My principles**
> - **Safety first:** I keep attack traffic off my real home network.
> - **Document everything:** I treat this like a production runbook + portfolio artifact.
> - **Iterate:** I start small (single SIEM VM + a couple endpoints), then add complexity (AD, firewall, detections, IR playbooks).

---

## 1) Objectives (What I’m building and why)

### Primary objective: SOC / Blue Team fundamentals (with Purple Team practice)
I’m building an environment that supports:
- Centralized logging (Windows + Linux, later network/firewall)
- Alerting and dashboards
- Investigation workflows (pivoting on endpoints, timeline, auth events, process activity)
- Simulated in-lab attacks and validation of detections

### Job-relevant skills I’ll practice
- Virtualization and lab network design
- Windows telemetry (event logs + Sysmon)
- Linux telemetry (auth logs, process, file integrity)
- SIEM onboarding and rule tuning
- Writing incident notes and post-incident reviews

---

## 2) My current inventory

### Physical machines I’m using
- **Kali Linux PC (VirtualBox host)**
  - RAM: 20 GB
  - CPU: Intel i5 (6th gen)
  - Role: virtual machine host, and sometimes my admin workstation
- **Parrot OS PC**
  - RAM: 16 GB
  - CPU: Intel i5-6200
  - Role: my attacker workstation (I prefer keeping attacker activity separate from the VM host)

### VirtualBox VMs I already have
- `Windows 10` (endpoint/victim)
- `Ubuntu 24` (endpoint/server)
- `Metasploitable 2` (intentionally vulnerable target)

### New VM I’m adding (in progress)
- `Ubuntu Server 24.04.3` VM for **Wazuh all-in-one** (SIEM)

> Note to self: Wazuh is installed **inside** an Ubuntu Server VM. There isn’t a special “Wazuh OS ISO” for this approach.

---

## 3) My target architecture (v1)

### 3.1 My network goals
- Keep my lab isolated from my home network
- Still allow updates/downloads when I need them (controlled)
- Ensure my SIEM can see endpoints and collect telemetry

### 3.2 My VirtualBox network model (recommended baseline)
- **LAB-INT (Host-Only Network):** endpoint-to-endpoint + SIEM traffic
- **NAT:** outbound internet access for updates (ideally only for the SIEM; endpoints can be temporarily enabled)

This gives me a “mostly isolated” setup:
- My **attack traffic** stays on LAB-INT.
- My environment can **still update** via NAT when I choose.

### 3.3 My lab roles (v1)
- **SIEM/Monitoring:** Wazuh (all-in-one) on Ubuntu Server
- **Endpoints:** Windows 10 + Ubuntu 24 with Wazuh agents
- **Vulnerable target:** Metasploitable 2
- **Attacker workstation:** Parrot OS (preferred), or Kali host as needed

---

## 4) My build order (high level)

1. **Create lab networks** in VirtualBox (Host-Only + NAT)
2. **Build my SIEM VM**
   - Install Ubuntu Server
   - Configure a stable IP on LAB-INT
   - Install Wazuh all-in-one
3. **Onboard my endpoints**
   - Install Wazuh agent on Ubuntu 24
   - Install Wazuh agent on Windows 10
   - (Optional but recommended) Install Sysmon on Windows 10
4. **Generate test events**
   - SSH login attempts, sudo use, new users, suspicious processes
   - Basic port scan from my attacker box
5. **Validate detections + dashboards**
   - Confirm logs arrive
   - Tune noisy rules
   - Create notes and evidence
6. **Iterate**
   - Add Active Directory mini-corp
   - Add firewall/router (OPNsense/pfSense)
   - Add vulnerability scanner (OpenVAS)
   - Write detection content and IR playbooks

---

## 5) VM specs (recommended starting point)

### Wazuh VM (all-in-one)
- OS: Ubuntu Server 24.04.3
- CPU: 2 vCPU
- RAM: 6–8 GB (start at 6 GB; increase if sluggish)
- Disk: 60 GB

### Windows 10 endpoint
- CPU: 2 vCPU
- RAM: 4–6 GB
- Disk: 50–80 GB

### Ubuntu 24 endpoint
- CPU: 1–2 vCPU
- RAM: 2–4 GB
- Disk: 25–40 GB

### Metasploitable 2
- CPU: 1 vCPU
- RAM: 1 GB
- Disk: as-is

---

## 6) Runbook (step-by-step)

### Step 0 — My safety checklist
- [ ] I do not use Bridged networking for target/victim VMs during attack practice.
- [ ] I ensure the lab network is NOT my home LAN.
- [ ] I avoid exposing vulnerable VMs to the internet.

---

### Step 1 — Create my VirtualBox networks

**Goal:** Give myself an isolated lab network plus optional outbound updates.

**Actions:**
1. I create a **Host-Only Network** (LAB-INT)
   - Suggested CIDR: `192.168.56.0/24`
   - Host IP (VirtualBox host-only adapter): `192.168.56.1`
   - DHCP: Off (I prefer static IPs for lab clarity)
2. I use a per-VM **NAT** adapter for controlled internet access.

**Validation:**
- From my Kali host, I can see a host-only adapter with IP `192.168.56.1`.
- VMs attached to LAB-INT can ping each other (after I set IPs).

**Notes:**
- Host-Only is easier than Internal Network early on because my host can reach the subnet.

---

### Step 2 — Create my `wazuh` VM and install Ubuntu Server 24.04.3

**Goal:** Build a dedicated SIEM VM.

**VM configuration:**
- Name: `wazuh`
- RAM: 6144–8192 MB
- CPU: 2 cores
- Disk: 60 GB
- Network:
  - Adapter 1: Host-Only (LAB-INT)
  - Adapter 2: NAT

**OS install notes:**
- I install OpenSSH server: Yes (recommended)
- I create an admin user (example): `socadmin`

**Validation:**
- I can log in to the Ubuntu VM via console.
- `ip a` shows two interfaces (one for NAT, one for LAB-INT).

---

### Step 3 — Configure a stable IP for LAB-INT on my Wazuh VM

**Goal:** Ensure I always know where to reach my Wazuh dashboard.

**Actions (Ubuntu uses netplan):**
1. I identify interface names: `ip a`
2. I find the netplan file: `ls /etc/netplan/`
3. I configure the LAB-INT NIC with a static IP, e.g.:
   - `192.168.56.10/24`
   - No gateway needed for host-only
4. I apply the config: `sudo netplan apply`

**Validation:**
- `ip a` shows `192.168.56.10/24` on my LAB-INT NIC.
- From my Kali host: I can ping `192.168.56.10` (if ICMP is allowed).

---

### Step 4 — Install Wazuh (all-in-one)

**Goal:** Install Wazuh Manager + Indexer + Dashboard in my Wazuh VM.

**Approach:**
- I install via the official Wazuh all-in-one installation method for Ubuntu.
- I keep a record of:
  - Dashboard URL
  - Initial credentials (I store these securely; I do not commit secrets)

**Validation:**
- The dashboard is reachable from my Kali host browser:
  - `https://192.168.56.10`
- I can log into the dashboard.

> **Security note:** I do not store passwords in this repo. I use a password manager or a `secrets.local.md` that is excluded from version control.

---

### Step 5 — Onboard my endpoints (Windows 10 + Ubuntu 24)

**Goal:** Send endpoint telemetry to Wazuh.

**Actions:**
- I install the Wazuh agent on:
  - `Windows 10`
  - `Ubuntu 24`
- I point the agents to my Wazuh manager IP: `192.168.56.10`
- (Recommended) I install Sysmon on Windows 10 and forward event logs via the Wazuh agent integration.

**Validation:**
- My endpoints show as “active” in Wazuh.
- I can see:
  - Windows security events
  - Linux auth and system events

---

### Step 6 — Generate test events + confirm detections

**Goal:** Practice purple-team loops: do something → see telemetry → detect → investigate.

**Examples:**
- Failed logins (Windows and Linux)
- New local user created
- RDP attempts (if enabled in my lab)
- Port scan from Parrot to Metasploitable (lab-only)

**Validation:**
- Wazuh shows alerts or events corresponding to my actions.
- I document: “What I did”, “What logs showed”, “What alert fired”, “What I learned”.

---

## 7) Build log (chronological)

### 2026-02-09 — Lab kickoff
- I identified the hardware I have available:
  - Kali host (VirtualBox) + Parrot workstation
- I confirmed my existing VMs:
  - Windows 10, Ubuntu 24, Metasploitable 2
- I decided to add a SIEM:
  - Wazuh all-in-one on an Ubuntu Server VM
- I started downloading:
  - Ubuntu Server 24.04.3 ISO
- I installed Ubuntu Server on a new VM:
  - CLI-only environment confirmed (expected for server)

> I’ll add the next entry once I finish Wazuh VM networking and the Wazuh installation.

---

## 8) Operating procedures (lightweight)

### Change management
For every meaningful change, record:
- What changed
- Why
- How to validate
- How to rollback

### Documentation rule
If I didn’t write it down here, I assume it didn’t happen.

---

## 9) Roadmap

### v1 (current)
- [ ] Wazuh all-in-one running
- [ ] Windows 10 agent sending logs
- [ ] Ubuntu 24 agent sending logs
- [ ] Basic attack simulations + evidence

### v2 (enterprise realism)
- [ ] Add Windows Server (Active Directory)
- [ ] Domain-join Windows 10
- [ ] Add OPNsense/pfSense VM for segmentation and logging
- [ ] Add vuln scanning (OpenVAS)
- [ ] Write 3–5 detection rules + tuning notes

---

## 10) Disclaimer
This lab is for educational purposes only. I only test systems I own or have explicit permission to test. I keep vulnerable VMs isolated from the internet and my home network.