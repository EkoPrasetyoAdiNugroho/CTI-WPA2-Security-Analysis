# 🛡️ Threat Intelligence & WPA2-Personal Security Analysis

> **Course:** Cyber Threat Intelligence (CTI)
> **Study Program:** Informatics Engineering
> **University:** Universitas Muhammadiyah Makassar
> **Author:** Eko Prasetyo Adi Nugroho
> **Student ID:** 105841114223
> **Class:** JK-A

---

## 📌 Project Overview

This project analyzes the security of **WPA2-Personal (PSK)** networks through **wireless packet capture**, **4-Way Handshake analysis**, and **threat intelligence mapping** using the **MITRE ATT&CK** framework.

The assessment was conducted in a controlled personal test environment using **Kali Linux running in Oracle VirtualBox** and an external USB Wi-Fi adapter based on the **AICSemi AIC8800DC chipset**.

The analysis covers wireless network discovery, **Probe Request** capture, **WPA2 4-Way Handshake** capture, detailed inspection of **EAPOL Messages 1–4** using Wireshark, and a **dictionary attack proof of concept** to validate the security of a weak pre-shared key.

> ⚠️ **Disclaimer:** All testing in this project was performed on a controlled test network for educational and security research purposes only. Do not perform security testing against networks or devices without proper authorization.

---

## 🎯 Objectives

The main objectives of this project are to:

* Understand the **WPA2 4-Way Handshake** process.
* Capture and analyze **EAPOL Messages 1–4**.
* Identify potential security threats affecting WPA2-Personal networks.
* Map identified threats to the **MITRE ATT&CK for Enterprise** framework.
* Demonstrate the security impact of a weak PSK through a controlled dictionary attack.
* Provide practical mitigation recommendations for improving wireless network security.

---

## 🧪 Testing Environment

| Component        | Details             |
| ---------------- | ------------------- |
| Operating System | Kali Linux          |
| Virtualization   | Oracle VirtualBox   |
| Wireless Adapter | USB Wi-Fi Adapter   |
| Chipset          | AICSemi AIC8800DC   |
| Target SSID      | `LUFFY`             |
| BSSID            | `80:3C:20:DD:C2:34` |
| Channel          | `11`                |
| Security         | WPA2-PSK / CCMP     |
| Test Client      | `3E:11:06:92:D4:BE` |

---

## 🛠️ Tools Used

* **Wireshark** — packet inspection and EAPOL analysis.
* **Aircrack-ng Suite** — wireless monitoring and packet capture.
* **airodump-ng** — wireless network discovery and packet capture.
* **aircrack-ng** — controlled dictionary attack validation.
* **iw** — manual wireless interface configuration.
* **Oracle VirtualBox** — virtualization environment for Kali Linux.

---

## 🔍 Methodology

### 1. Wireless Adapter Detection

The USB Wi-Fi adapter was first identified using:

```bash
lsusb
```

The system successfully detected the **AICSemi AIC8800DC** wireless adapter.

### 2. USB Passthrough Configuration

Because Kali Linux was running inside Oracle VirtualBox, the physical USB adapter was passed through from the host system to the virtual machine.

### 3. Monitor Mode Configuration

The initial attempt to enable monitor mode using `airmon-ng` resulted in a driver-related error.

Therefore, monitor mode was enabled manually using:

```bash
ip link set wlan0 down
iw dev wlan0 set type monitor
ip link set wlan0 up
```

The interface was verified using:

```bash
iw dev
```

The wireless interface remained:

```text
wlan0
```

### 4. Wireless Network Discovery

Network discovery was performed using:

```bash
airodump-ng wlan0
```

The target test network was identified as:

```text
SSID    : LUFFY
BSSID   : 80:3C:20:DD:C2:34
Channel : 11
```

### 5. Probe Request Analysis

A separate capture was performed to analyze wireless **Probe Request** traffic.

The following Wireshark filter was used:

```text
wlan.fc.type_subtype == 0x04
```

The captured packets were analyzed based on frame fields, transmitter/source addresses, SSID information, and hexadecimal/ASCII representation.

### 6. WPA2 4-Way Handshake Capture

The capture was focused on the target BSSID using:

```bash
airodump-ng -c 11 --bssid 80:3C:20:DD:C2:34 -w captures/wpa2_handshake wlan0
```

A new handshake was captured after the test client performed a re-authentication process with the access point.

### 7. EAPOL Analysis

The captured file was opened in Wireshark and filtered using:

```text
eapol
```

The four EAPOL messages were analyzed individually.

#### Message 1/4 — AP → Client

The Access Point sends the **ANonce (Authenticator Nonce)** to the client.

#### Message 2/4 — Client → AP

The client responds with the **SNonce (Supplicant Nonce)** and **MIC**, along with the RSN Information Element.

#### Message 3/4 — AP → Client

The Access Point sends the ANonce again, together with the MIC and encrypted key data containing the **GTK (Group Temporal Key)**.

#### Message 4/4 — Client → AP

The client sends the final confirmation indicating that the key installation process has been completed and encrypted communication can begin.

---

## 🔐 Dictionary Attack — Proof of Concept

To validate the captured handshake and demonstrate the risk associated with weak WPA2-PSK passwords, a controlled **offline dictionary attack** was performed using **Aircrack-ng** and `rockyou.txt`.

The test successfully identified the PSK used by the lab network.

### Test Result

```text
ESSID  : LUFFY
Attack : Offline Dictionary Attack
Result : KEY FOUND!
Time   : 00:01:15
```

The result demonstrates that a weak and predictable PSK can be vulnerable to an offline dictionary attack even when the network uses **WPA2/CCMP**.

---

## ⚠️ Threat Intelligence & MITRE ATT&CK Mapping

| Threat Vector                             | MITRE ATT&CK / CVE | Risk Level     |
| ----------------------------------------- | ------------------ | -------------- |
| WPA2 Handshake Capture & Offline Cracking | `T1110.001`        | 🔴 High        |
| ARP Spoofing / Adversary-in-the-Middle    | `T1557.002`        | 🟠 Medium-High |
| KRACK — Key Reinstallation Attack         | `CVE-2017-13077`   | 🔴 High        |
| Passive Network Sniffing                  | `T1040`            | 🟡 Medium      |

### T1110.001 — Offline Dictionary Attack

A captured WPA2 handshake can be used to perform password testing offline against weak PSKs using dictionary-based techniques.

### T1557.002 — ARP Spoofing

After gaining access to the network, an attacker may manipulate ARP communication to redirect local traffic and potentially perform an Adversary-in-the-Middle attack.

### CVE-2017-13077 — KRACK

The **Key Reinstallation Attack (KRACK)** is associated with the WPA2 key installation process and can exploit retransmission behavior during the handshake.

### T1040 — Network Sniffing

A wireless adapter operating in monitor mode can capture wireless traffic and metadata from networks within radio range.

---

## 🛡️ Mitigation Recommendations

### 1. Strong Password Policy

Use a strong WPA2-PSK with at least **16–20 characters**, combining:

* Uppercase and lowercase letters
* Numbers
* Special characters

This significantly increases resistance against dictionary-based password attacks.

### 2. Firmware and System Updates

Regularly update the Access Point firmware and client operating systems to reduce exposure to known vulnerabilities such as KRACK.

### 3. Migration to WPA3-Personal

Consider migrating to **WPA3-Personal**, which uses **Simultaneous Authentication of Equals (SAE)** to improve resistance against offline password attacks.

### 4. Wireless Monitoring / WIPS

Implement wireless monitoring or **Wireless Intrusion Prevention Systems (WIPS)** to help detect suspicious wireless activity, rogue access points, and unauthorized scanning.

---

## 📂 Repository Structure

```text
CTI-WPA2-Security-Analysis/
│
├── captures/
│   └── wpa2_handshake_sample.pcap
│
├── screenshots/
│   ├── adapter_detection.png
│   ├── monitor_mode.png
│   ├── network_discovery.png
│   ├── probe_request.png
│   ├── handshake_capture.png
│   ├── eapol_message_1.png
│   ├── eapol_message_2.png
│   ├── eapol_message_3.png
│   ├── eapol_message_4.png
│   └── aircrack_result.png
│
├── references.md
└── README.md
```

---

## 📸 Documentation

The repository contains documentation and evidence covering:

* USB Wi-Fi adapter detection
* VirtualBox USB passthrough configuration
* Monitor mode activation
* Wireless network discovery
* Probe Request analysis
* WPA2 4-Way Handshake capture
* EAPOL Message 1–4 analysis
* Dictionary attack proof of concept

---

## 📊 Key Findings

The assessment demonstrates that **WPA2-Personal remains vulnerable to offline password attacks when a weak PSK is used**.

Although WPA2 with AES-CCMP provides strong cryptographic protection, the security of WPA2-Personal is still highly dependent on the strength of the configured pre-shared key.

The assessment therefore highlights the importance of:

* Strong and unpredictable passwords
* Regular firmware and system updates
* Wireless security monitoring
* Migration to WPA3-Personal where possible

---

## 👨‍💻 Author

**Eko Prasetyo Adi Nugroho**

**Cyber Threat Intelligence — Informatics Engineering**
**Universitas Muhammadiyah Makassar**

---

## 📜 License

This project was created for **academic and educational cybersecurity research purposes**.

Unauthorized testing of networks, systems, or devices is strictly prohibited.
