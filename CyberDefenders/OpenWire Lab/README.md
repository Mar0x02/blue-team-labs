# OpenWire Lab — CyberDefenders

> **Platform:** CyberDefenders
> **Category:** Network Forensics
> **Difficulty:** Medium
> **Status:** 🔄 In Progress
> **Date:** 2026-08-14
> **Time Spent:** ~1 jam 30 menit

---

## 📌 Prolog

Investigasi Java deserialization vulnerability di Apache ActiveMQ yang memungkinkan remote code execution lewat insecure class loading.

**Tools:** Wireshark | Zui | Network Miner

**Tactics yang tercakup:** Initial Access | Execution | Command and Control

**Catatan:** lab ini berstatus *Retired* di platform CyberDefenders.

---

## 🎯 Scenario

Selagi shift jadi tier-2 SOC analyst, lo dapet eskalasi dari tier-1 analyst soal public-facing server. Server ini keflag karena bikin outbound connection ke beberapa IP yang mencurigakan. Sebagai respons, lo jalanin standard incident response protocol — termasuk isolasi server dari network buat cegah potential lateral movement atau data exfiltration, dan ambil packet capture dari NSM utility buat dianalisis. Tugas lo: analisis pcap-nya dan assess tanda-tanda malicious activity.

---

## ❓ Questions

1. By identifying the C2 IP, we can block traffic to and from this IP, helping to contain the breach and prevent further data exfiltration or command execution. Can you provide the IP of the C2 server that communicated with our server?
2. Initial entry points are critical to trace the attack vector back. What is the port number of the service the adversary exploited?
3. Following up on the previous question, what is the name of the service found to be vulnerable?
4. The attacker's infrastructure often involves multiple components. What is the IP of the second C2 server?
5. Attackers usually leave traces on the disk. What is the name of the reverse shell executable dropped on the server?
6. What Java class was invoked by the XML file to run the exploit?
7. To better understand the specific security flaw exploited, can you identify the CVE identifier associated with this vulnerability?
8. The vendor addressed the vulnerability by adding a validation step to ensure that only valid Throwable classes can be instantiated, preventing exploitation. In which Java class and method was this validation step added?

---

## 🔍 Answer & Walkthrough

> 🔄 *Belum diisi — akan dilengkapi setelah pengerjaan selesai.*

---

## 🚨 Key Findings / IOCs

> 🔄 *Belum diisi.*

| Tipe | Value | Keterangan |
|------|-------|------------|
| IP Address | `...` | ... |
| File Hash | `...` | ... |
| Domain | `...` | ... |

---

## 🗺️ MITRE ATT&CK Mapping

> 🔄 *Belum diisi.*

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| ... | ... | ... | ... |

---

## 📋 Summary — Attacker Behavior & Todo

> 🔄 *Belum diisi — akan dilengkapi setelah analisis selesai.*

### Attacker Behavior

🔄 Belum diisi.

### Todo / Follow-up

- [ ] Analisis pcap via Wireshark/Zui/NetworkMiner buat identifikasi C2 IP pertama & kedua (pertanyaan #1 & #4)
- [ ] Trace initial entry point — port & service yang di-exploit (pertanyaan #2 & #3)
- [ ] Cari reverse shell executable yang di-drop ke disk (pertanyaan #5)
- [ ] Identifikasi Java class di XML exploit payload (pertanyaan #6)
- [ ] Cross-check CVE ID buat ActiveMQ deserialization vulnerability ini (pertanyaan #7)
- [ ] Cek vendor patch — class & method mana yang nambahin validasi Throwable (pertanyaan #8)

---

## 📚 References

- [CyberDefenders — OpenWire Lab](https://cyberdefenders.org/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
