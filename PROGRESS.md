# 📊 Progression du projet — redblue-python-lab

## 🏁 Avancement global

```
██████████████████████████░░░░░░░░░░░░░░ 65%
```

**Phase actuelle** : Phase 1 ✅ COMPLÈTE + Lab Packet Tracer 90% (v2 à venir)

**Dernière mise à jour** : 12 novembre 2026 (soirée)

**Prochaine phase** : 🐍 PHASE 2 — Python (premier script : ping scanner)

---

## 📋 Détail par phase

### Phase 0 — Setup environnement ✅ 100%

- [x] Python 3.13.9 installé sur le PC hôte
- [x] Git 2.51.0 installé et configuré
- [x] VSCode 1.116.0 installé
- [x] Compte GitHub (AntoSecLab) + repo créé
- [x] Clé SSH générée et associée à AntoSecLab
- [x] Fichier `~/.ssh/config` configuré (gestion multi-comptes GitHub)
- [x] Identité Git locale au projet (sans `--global`)
- [x] Repo cloné localement dans `Projects-Perso/`
- [x] VMware Workstation installé
- [x] 4 VMnets créés et renommés : Clients, Servers, Blue, Red
- [x] Documentation réseau (`docs/network-architecture.md`)
- [x] Journal de bord mis en place
- [x] ISOs téléchargées (pfSense, Kali, Metasploitable 2, Windows Server 2025, Windows 10, Ubuntu)

### Phase 1 — Lab de base ✅ 100%

- [x] pfSense installé et configuré (5 interfaces, DHCP activé sur 3 LANs)
- [x] Interfaces renommées (SERVERS/BLUE/RED au lieu d'OPT1/2/3)
- [x] Kali Linux installée (double NIC : NAT + Red)
- [x] Connexions NetworkManager dédiées (`NAT-Internet`, `Red-Lab`)
- [x] Internet validé sur Kali (ping 8.8.8.8)
- [x] DHCP pfSense distribue des IPs (Kali reçoit 192.168.99.101)
- [x] Décochage des host adapters sur toutes les VMnets (fix conflit IP VMnet3/5)
- [x] Accès interface web pfSense depuis Red (https://192.168.99.1)
- [x] Mot de passe admin pfSense changé
- [x] Archi propre à 2 NICs sur Kali (eth2 temporaire supprimé)
- [x] Documentation du troubleshooting (`docs/troubleshooting.md`)
- [x] Schéma visuel de l'infra (`docs/network-diagram.svg`)
- [x] Création des règles firewall pfSense pour RED (3 règles)
  - [x] Allow RED to pfSense HTTPS (admin) — port 443 TCP
  - [x] Allow RED ICMP (ping) — any destination
  - [x] Allow RED to Servers (pentest) — Network 192.168.20.0/24
- [x] Metasploitable 2 installée et démarrée
- [x] IP statique Metasploitable configurée (192.168.20.100)
- [x] Fix routage Kali : ajout route statique vers 192.168.20.0/24 via 192.168.99.1
- [x] **Validation finale : ping Kali → pfSense → Metasploitable OK (ttl=63, 0% loss)**

### 🎓 Lab Packet Tracer parallèle ✅ 90% _(Dernière maj : 12/11/2026)_

- [x] Topologie physique (routeur 1941 + switch 2960 + 4 PCs)
- [x] Sécurisation devices (enable, console, VTY, password-encryption)
- [x] Création des 4 VLANs (CLIENTS, SERVERS, BLUE, RED)
- [x] Ports access assignés aux VLANs
- [x] Trunk 802.1Q entre switch et routeur
- [x] Router-on-a-stick (sous-interfaces dot1Q)
- [x] DHCP serveur sur routeur (4 pools)
- [x] ACL extended bloquant RED → SERVERS
- [x] Tests ping validés (OK/KO selon ACL)
- [x] Documentation complète (README dans `packet-tracer/`)
- [ ] (Futur v2) Ajout d'un Red externe (simulateur hacker depuis internet)

### Phase 2 — Premier script Python : Ping Scanner ⏳ 0%

- [ ] Rendre persistantes : route statique Kali + IP statique Meta (survie au reboot)
- [ ] Python validé dans Kali (`python3 --version`)
- [ ] Création du module `scanner/`
- [ ] Première ligne de Python 🐍
- [ ] Ping scanner simple (hôte unique)
- [ ] Ping scanner d'un subnet complet (/24)
- [ ] `argparse` pour paramétrer le range depuis la CLI
- [ ] Multi-threading pour accélérer
- [ ] Détection de l'OS via TTL (Windows=128, Linux=64)
- [ ] Export des résultats en JSON
- [ ] Documentation du script (README dédié)
- [ ] Fiches Anki : vocabulaire Python de base

### Phase 3 — Port Scanner ⏳ 0%

- [ ] Création du module `portscanner/`
- [ ] Port scanner TCP basique
- [ ] Port scanner multi-threadé
- [ ] Argparse pour CLI (options --ports, --threads, --output)
- [ ] Tests unitaires
- [ ] Documentation d'utilisation

### Phase 4 — Banner Grabbing & Fingerprinting ⏳ 0%

- [ ] Détection des services derrière les ports ouverts
- [ ] Parsing des bannières avec regex
- [ ] Export JSON/CSV des résultats

### Phase 5 — Packet Sniffer (Scapy) ⏳ 0%

- [ ] Installation de Scapy
- [ ] Capture de paquets en temps réel
- [ ] Parsing TCP/UDP/ARP
- [ ] Sauvegarde en .pcap

### Phase 6 — ARP Spoofing & MITM ⏳ 0%

- [ ] ARP spoofer en Python
- [ ] Interception de trafic
- [ ] DNS spoofing

### Phase 7 — Password Tools ⏳ 0%

- [ ] Hash cracker (MD5, SHA1, SHA256)
- [ ] Analyseur d'entropie de mots de passe
- [ ] Password spraying Active Directory
- [ ] Kerberoasting (si AD déployé)

### Phase 8 — Attaques Web ⏳ 0%

- [ ] Déploiement de DVWA via Docker
- [ ] Scanner SQLi basique
- [ ] Scanner XSS basique
- [ ] Analyse de headers HTTP de sécurité

### Phase 9 — 🛡️ Blue Team (SIEM) ⏳ 0%

- [ ] Installation de Wazuh
- [ ] Configuration des agents sur les VMs
- [ ] Création de règles de détection custom
- [ ] Détection de scans Red team en temps réel

### Phase 10 — Durcissement & Dockerisation ⏳ 0%

- [ ] Hardening pfSense (règles firewall strictes)
- [ ] GPO sur Active Directory
- [ ] Dockerisation de la toolbox Red team
- [ ] CLI unifiée en une seule commande

### Phase 11 — Rapport & Portfolio ⏳ 0%

- [ ] Rapport de pentest format pro (PDF)
- [ ] Vidéo démo du lab
- [ ] README final soigné avec screenshots
- [ ] Publication LinkedIn / Medium

---

## 🎯 Prochaines actions prioritaires

1. **Rendre persistantes** la route statique Kali + l'IP Meta (sinon tout casse au reboot)
2. **🐍 PHASE 2** : premières lignes de Python — ping scanner
3. **Enrichir Packet Tracer v2** avec un "Red externe" (hacker simulé depuis Internet)
4. Préparer la base pour Phase 3 (Port Scanner)

---

## 📊 Stats

| Indicateur | Valeur |
|---|---|
| Phases complétées | 2 / 12 (Phase 0 + Phase 1) + Lab PT (90%) |
| Phases en cours | 0 (prêt pour Phase 2 Python) |
| Sessions de travail | 5 |
| Documents produits | 2 cheatsheets PDF + JOURNAL + PROGRESS + architecture + 2 troubleshooting + README PT |
| Fiches Anki créées | ~85 (+30 en session #5 sur routing/firewall/debug) |
| Lab Packet Tracer | 1 (v1 interne, v2 externe à venir) |
| Lab VMware | 1 (pfSense + Kali + Metasploitable, 100% fonctionnel) |
| Règles firewall créées | 3 (sur interface RED) |
| Date de début | Septembre 2026 |
| Date de reprise | 09 novembre 2026 |
| Dernière session | 12 novembre 2026 (soirée) |
