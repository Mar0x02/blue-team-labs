# PsExec Hunt Lab — CyberDefenders

> **Platform:** [CyberDefenders](https://cyberdefenders.org/blueteam-ctf-challenges/psexec-hunt/)  
> **Category:** Network Forensics  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-06-26  
> **Time Spent:** ~30 menit  
> **Tags:** `PsExec` `SMB` `Lateral Movement` `Wireshark` `PCAP Analysis`  
> **Tactics:** Execution, Defense Evasion, Discovery, Lateral Movement

---

## 📌 Prolog

Analyze SMB traffic in a PCAP file using Wireshark to identify PsExec lateral movement, compromised systems, user credentials, dan administrative shares. Challenge ini bagus untuk memahami bagaimana PsExec terlihat di level network — mulai dari authentication SMB, penggunaan administrative share, hingga instalasi service secara remote.

---

## 🎯 Scenario

Alert dari Intrusion Detection System (IDS) menandai aktivitas lateral movement yang mencurigakan menggunakan PsExec. Ini mengindikasikan kemungkinan unauthorized access dan pergerakan di dalam network. Sebagai SOC Analyst, tugas kita adalah menginvestigasi file PCAP yang tersedia untuk melacak aktivitas attacker — mengidentifikasi entry point, mesin yang menjadi target, sejauh mana breach terjadi, dan indikator kritis yang mengungkap taktik serta tujuan mereka di dalam environment yang sudah dikompromise.

---

## ❓ Questions

1. To effectively trace the attacker's activities within our network, can you identify the IP address of the machine from which the attacker initially gained access?
2. To fully understand the extent of the breach, can you determine the machine's hostname to which the attacker first pivoted?
3. Knowing the username of the account the attacker used for authentication will give us insights into the extent of the breach. What is the username utilized by the attacker for authentication?
4. After figuring out how the attacker moved within our network, we need to know what they did on the target machine. What's the name of the service executable the attacker set up on the target?
5. We need to know how the attacker installed the service on the compromised machine to understand the attacker's lateral movement tactics. This can help identify other affected systems. Which network share was used by PsExec to install the service on the target machine?
6. We must identify the network share used to communicate between the two machines. Which network share did PsExec use for communication?
7. Now that we have a clearer picture of the attacker's activities on the compromised machine, it's important to identify any further lateral movement. What is the hostname of the second machine the attacker targeted to pivot within our network?

---

## 🔍 Answer & Walkthrough

> 🔄 *Belum diisi — akan dilengkapi setelah pengerjaan selesai.*

---

## 🚨 Key Findings / IOCs

> 🔄 *Belum diisi.*

| Tipe | Value | Keterangan |
|------|-------|------------|
| IP Address | `...` | Attacker source IP |
| Hostname | `...` | First pivot target |
| Hostname | `...` | Second pivot target |
| Username | `...` | Akun yang digunakan attacker |
| Service | `...` | Executable yang diinstall PsExec |
| SMB Share | `...` | Share untuk install service |
| SMB Share | `...` | Share untuk komunikasi |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Lateral Movement | Remote Services: SMB/Windows Admin Shares | T1021.002 | PsExec menggunakan SMB admin share untuk lateral movement |
| Execution | System Services: Service Execution | T1569.002 | PsExec menginstall dan menjalankan service di target |
| Defense Evasion | Masquerading | T1036 | Service PsExec menyamar sebagai proses legitimate |
| Discovery | Network Share Discovery | T1135 | Attacker enumerate network share di target machine |

---

## 📋 Summary — Attacker Behavior & Todo

> 🔄 *Belum diisi — akan dilengkapi setelah analisis selesai.*

### Todo / Follow-up

- [ ] Pelajari lebih dalam cara kerja PsExec di level protokol SMB
- [ ] Eksplorasi detection rule untuk PsExec lateral movement di Sysmon (Event ID 7045)
- [ ] Latihan filter Wireshark untuk SMB traffic: `smb2`, `smb`, `dcerpc`
- [ ] Buat Sigma rule untuk mendeteksi PsExec berdasarkan IOC yang ditemukan

---

## 📚 References

- [MITRE ATT&CK — T1021.002 SMB/Windows Admin Shares](https://attack.mitre.org/techniques/T1021/002/)
- [MITRE ATT&CK — T1569.002 Service Execution](https://attack.mitre.org/techniques/T1569/002/)
- [CyberDefenders — PsExec Hunt Lab](https://cyberdefenders.org/blueteam-ctf-challenges/psexec-hunt/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
