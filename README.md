# 🛡️ Red Blue Python Lab

> Laboratoire d'apprentissage **Python appliqué à la cybersécurité** offensive et défensive, dans un environnement virtualisé isolé (VMware).

![Status](https://img.shields.io/badge/Status-En%20cours-yellow)
![Phase](https://img.shields.io/badge/Phase-1%20%2F%2012-blue)
![Progress](https://img.shields.io/badge/Progress-45%25-orange)
![Python](https://img.shields.io/badge/Python-3.13-blue)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 🎯 Objectifs du projet

Ce projet a un **double objectif pédagogique** :

1. **Apprendre le langage Python** en profondeur, from scratch
2. **Comprendre les attaques réseau** pour mieux les **défendre** — philosophie Red Team pour bâtir du Blue Team

À travers la construction d'un **lab réseau d'entreprise réaliste** (VLANs, pare-feu, Active Directory, SIEM), je développe mes propres outils Python offensifs puis mets en place les défenses associées.

---

## ⚠️ Avertissement légal

Tous les outils de ce repo sont destinés à être utilisés **UNIQUEMENT** sur ce laboratoire virtuel isolé (VMnets Host-Only).

Les utiliser sur un réseau qui ne vous appartient pas ou sans autorisation écrite est **illégal** en France (article 323-1 du Code pénal — jusqu'à 2 ans de prison et 60 000 € d'amende).

---

## 📚 Documentation du projet

| Fichier | Description |
|---|---|
| [`JOURNAL.md`](./JOURNAL.md) | Journal chronologique de bord — chaque session documentée |
| [`PROGRESS.md`](./PROGRESS.md) | Checklist détaillée de toutes les phases |
| [`docs/network-architecture.md`](./docs/network-architecture.md) | Architecture réseau complète avec schéma |
| [`docs/troubleshooting.md`](./docs/troubleshooting.md) | Toutes les pannes rencontrées + solutions |

---

## 🗺️ Feuille de route

- [x] **Phase 0** — Setup environnement (Python, VSCode, Git, SSH, VMware)
- [ ] **Phase 1** — Lab de base (pfSense, Kali, Metasploitable) ⏳ 85%
- [ ] **Phase 2** — Premier script Python : Ping Scanner 🐍
- [ ] **Phase 3** — Port Scanner multi-threadé
- [ ] **Phase 4** — Banner Grabbing & détection de services
- [ ] **Phase 5** — Packet Sniffer (Scapy)
- [ ] **Phase 6** — ARP Spoofing & MITM
- [ ] **Phase 7** — Password Tools (hash cracker, analyseur d'entropie)
- [ ] **Phase 8** — Web Vulnerability Scanner (SQLi, XSS)
- [ ] **Phase 9** — 🛡️ Blue Team : SIEM Wazuh + détection de scans
- [ ] **Phase 10** — Durcissement + Dockerisation
- [ ] **Phase 11** — Rapport final + portfolio

---

## 🖥️ Architecture du lab

```
                  🌍 INTERNET (via NAT VMware)
                            │
                  ┌─────────▼─────────┐
                  │  🛡️ pfSense Router │
                  │   (5 interfaces)   │
                  └─────┬──────────┬───┘
                        │          │
      ┌─────────────────┼──────────┼────────────────┐
      │                 │          │                │
  🟢 CLIENTS       🔵 SERVERS    💙 BLUE         🔴 RED
 192.168.10.0/24  192.168.20.0  192.168.30.0  192.168.99.0
      │                 │          │                │
  💻 Windows 10    🪟 Win Server  🛡️ Wazuh       🐉 Kali Linux
                  🎯 Metasploitable (SIEM)      (attaquant)
```

**Voir [docs/network-architecture.md](./docs/network-architecture.md) pour les détails complets.**

---

## 📦 Stack technique

### Infrastructure
- **VMware Workstation** — hyperviseur
- **pfSense CE 2.7.2** — routeur / firewall
- **Kali Linux 2026.3** — plateforme d'attaque
- **Metasploitable 2** — cible vulnérable d'entraînement
- **Windows Server 2025** — serveur AD/DNS/DHCP (à venir)
- **Windows 10** — poste client (à venir)
- **Wazuh** — SIEM Blue Team (Phase 9)

### Développement
- **Python 3.13** — langage principal
- **VSCode** — éditeur (installé dans Kali)
- **Git + GitHub** — versioning
- **Docker** — pour les cibles web (DVWA, Juice Shop)

---

## 🎓 Compétences acquises à ce jour

### Infrastructure & Réseau
- Configuration VMware (VMnets, host-only, NAT)
- Installation et configuration pfSense (routeur, firewall, DHCP, règles)
- Segmentation réseau en VLANs
- Debug réseau avancé (ARP, ICMP, HTTPS, tcpdump)

### Linux / Système
- Installation Kali Linux
- Gestion des interfaces réseau avec NetworkManager (`nmcli`)
- SSH (génération de clés, config multi-comptes)

### Sécurité
- Principe "deny by default, allow by exception"
- Hardening : changement des mots de passe par défaut
- Mindset Red Team / Blue Team

### DevOps / Workflow
- Git + GitHub (SSH, multi-comptes, Conventional Commits)
- Documentation technique (Markdown, diagrammes SVG)
- Journal de bord structuré

---

## 👤 Auteur

**Anthony** — étudiant en Master Cybersécurité

- GitHub : [@AntoSecLab](https://github.com/AntoSecLab)

---

## 📄 Licence

MIT — voir le fichier [LICENSE](./LICENSE)
