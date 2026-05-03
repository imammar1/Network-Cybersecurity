# Home SOC Lab — Traffic Monitoring & Log Analysis

A home lab environment simulating a small internal network to practice entry-level SOC workflows. The lab covers generating authentication events, capturing network traffic, and correlating evidence across host logs and packet captures.

**Tools:** VirtualBox · Ubuntu Server · Kali Linux · Wireshark · OpenSSH

---

## Lab Environment

- **Attacker VM:** Kali Linux — simulates an internal workstation performing recon and authentication attempts
- **Target VM:** Ubuntu Server — runs OpenSSH and logs all authentication outcomes
- **Network:** Host-only adapter (`192.168.56.0/24`), fully isolated with no internet required

---

## Requirements

**Hardware**
- RAM: 16 GB recommended (8 GB minimum)
- CPU: 2+ cores with AMD-V/SVM enabled in BIOS
- Disk: 40–60 GB free space

**Software**
- [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Ubuntu Server LTS](https://ubuntu.com/download/server)
- [Kali Linux](https://www.kali.org/get-kali/)

---

## Setup

**1. Create a Host-Only Network in VirtualBox**

`VirtualBox → Tools → Network → Host-only Networks → Create`

Set IPv4 to `192.168.56.1`, mask `255.255.255.0`, and disable DHCP.

**2. Create Ubuntu-Target VM**

2 GB RAM, 2 CPUs, 20–25 GB disk. Attach Ubuntu ISO and set network adapter to Host-only (vboxnet0). Complete the installer, then remove the ISO and reboot.

**3. Create Kali-Attacker VM**

2–4 GB RAM, 2 CPUs, 20–25 GB disk. Attach Kali ISO and set network adapter to Host-only (vboxnet0). Complete the installer, then remove the ISO and reboot.

---

## Walkthrough

**Verify connectivity**
```bash
ip a                          # Run on both VMs to get IP addresses
ping -c 4 <UBUNTU_IP>         # Run from Kali — expect 0% packet loss
```

**Enable SSH on Ubuntu-Target**
```bash
sudo apt update && sudo apt install -y openssh-server
sudo systemctl enable --now ssh
```

**Simulate normal login from Kali**
```bash
ssh <username>@<UBUNTU_IP>    # Accept host key, enter correct password, then exit
```

**Simulate failed logins (brute-force behavior)**
```bash
ssh <username>@<UBUNTU_IP>    # Enter wrong password 3x, repeat for 6–12 total failures
```

**Investigate logs on Ubuntu-Target**
```bash
sudo grep "Failed password" /var/log/auth.log
sudo grep "Accepted password" /var/log/auth.log
sudo tail -f /var/log/auth.log   # Live monitoring
```

**Run a network scan from Kali**
```bash
sudo nmap -sS <UBUNTU_IP>     # Expect port 22/tcp open
```

**Capture traffic in Wireshark**

Start a capture on Kali's interface and use these display filters:
```
tcp.port == 22
ip.addr == <UBUNTU_IP>
```

Look for the TCP three-way handshake before each SSH session, repeated connection attempts from failed logins, and SYN packets across multiple ports from the Nmap scan.

---

## Skills Practiced

- Virtual network configuration and VM management
- SSH service deployment and authentication monitoring
- Log analysis with `grep` and `auth.log`
- Network traffic capture and analysis with Wireshark
- Traffic and log correlation (core SOC analyst workflow)

---

## Author

**Ibrahim Ammar** — [github.com/imammar1](https://github.com/imammar1)
