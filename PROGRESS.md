# 📊 Progression du projet — redblue-python-lab

> Vue d'ensemble de l'avancement sur l'ensemble des phases du projet.
> Mis à jour à chaque étape terminée.

**Dernière mise à jour** : 10 novembre 2026

---

## 🏁 Avancement global

```
██████████████████░░░░░░░░░░░░░░░░░░░░░░ 45%
```

**Phase actuelle** : Phase 1 — Lab de base (85%)

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

### Phase 1 — Lab de base ⏳ 85%

- [x] pfSense installé et configuré (5 interfaces, DHCP activé sur 3 LANs)
- [x] Interfaces renommées (SERVERS/BLUE/RED au lieu d'OPT1/2/3)
- [x] Kali Linux installée (double NIC : NAT + Red)
- [x] Connexions NetworkManager dédiées (`NAT-Internet`, `Red-Lab`)
- [x] Internet validé sur Kali (ping 8.8.8.8)
- [x] DHCP pfSense distribue des IPs (Kali reçoit 192.168.99.101)
- [x] Règle firewall RED → anywhere créée
- [x] Accès interface web pfSense depuis Red (https://192.168.99.1)
- [x] Mot de passe admin pfSense changé
- [x] Archi propre à 2 NICs sur Kali (eth2 temporaire supprimé)
- [x] Documentation du troubleshooting (`docs/troubleshooting.md`)
- [x] Schéma visuel de l'infra (`docs/network-diagram.svg`)
- [ ] Metasploitable 2 installée et démarrée
- [ ] IP statique Metasploitable configurée (192.168.20.100)
- [ ] Règle firewall RED → SERVERS (pour atteindre Metasploitable)
- [ ] Premier ping Kali → Metasploitable via pfSense (validation routage)

### 🎓 Lab Packet Tracer parallèle ✅ 90%  Dernière maj : 12/11/2026

- [x] Topologie physique (routeur 1941 + switch 2960 + 4 PCs)
- [x] Sécurisation devices (enable, console, VTY, password-encryption)
- [x] Création des 4 VLANs (CLIENTS, SERVERS, BLUE, RED)
- [x] Ports access assignés aux VLANs
- [x] Trunk 802.1Q entre switch et routeur
- [x] Router-on-a-stick (sous-interfaces dot1Q)
- [x] DHCP serveur sur routeur (4 pools)
- [x] ACL extended bloquant RED → SERVERS
- [x] Tests ping validés (OK/KO selon ACL)
- [x] Documentation complète (README dans packet-tracer/)
- [ ] (Futur v2) Ajout d'un Red externe (simulateur hacker depuis internet)


### Phase 2 — Premier script Python : Ping Scanner ⏳ 0%

- [ ] Python validé dans Kali (`python3 --version`)
- [ ] Création du module `scanner/`
- [ ] Première ligne de Python 🐍
- [ ] Ping scanner simple (hôte unique)
- [ ] Ping scanner d'un subnet complet (/24)
- [ ] Multi-threading pour accélérer
- [ ] Export des résultats en JSON

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

1. **Installer Metasploitable 2** (VM pré-faite, 5 min)
2. **Configurer son IP statique** (192.168.20.100)
3. **Créer règle firewall RED → SERVERS** (pour que Kali puisse atteindre Metasploitable)
4. **Premier ping multi-VLAN** (validation du lab complet)
5. **🐍 PHASE 2 : premières lignes de Python**

---

## 📊 Stats

| Indicateur | Valeur |
|---|---|
| Phases complétées | 1 / 12 (Phase 0) |
| Phases en cours | 1 (Phase 1 à 85%) |
| Sessions de travail | 3 |
| Date de début | Septembre 2026 |
| Date de reprise | 09 novembre 2026 |
| Dernière session | 10 novembre 2026 |