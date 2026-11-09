# 📊 Progression du projet — redblue-python-lab

> Vue d'ensemble de l'avancement sur l'ensemble des phases du projet.
> Mis à jour à chaque étape terminée.

**Dernière mise à jour** : 09 novembre 2026

---

## 🏁 Avancement global

```
████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 20%
```

**Phase actuelle** : Phase 0 — Setup environnement (presque terminée)

---

## 📋 Détail par phase

### Phase 0 — Setup environnement ⏳ 95%

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
- [ ] ISOs téléchargées (pfSense, Metasploitable, Kali, Windows Server 2022)

### Phase 1 — Lab de base ⏳ 0%

- [ ] pfSense installé et configuré (routeur + VLANs)
- [ ] Kali Linux installée (double NIC : NAT + Red)
- [ ] Metasploitable 2 installée (sur Servers uniquement)
- [ ] VSCode installé dans Kali
- [ ] Python validé dans Kali
- [ ] Repo cloné dans Kali (via SSH ou HTTPS)
- [ ] **🐍 Première ligne de Python : ping scanner**

### Phase 2 — Port Scanner ⏳ 0%

- [ ] Création du module `scanner/`
- [ ] Port scanner multi-threadé
- [ ] Argparse pour CLI
- [ ] Tests unitaires
- [ ] Documentation d'utilisation

### Phase 3 — Banner Grabbing & Fingerprinting ⏳ 0%

- [ ] Détection des services derrière les ports ouverts
- [ ] Parsing des bannières avec regex
- [ ] Export JSON/CSV des résultats

### Phase 4 — Packet Sniffer (Scapy) ⏳ 0%

- [ ] Installation de Scapy
- [ ] Capture de paquets en temps réel
- [ ] Parsing TCP/UDP/ARP
- [ ] Sauvegarde en .pcap

### Phase 5 — ARP Spoofing & MITM ⏳ 0%

- [ ] ARP spoofer en Python
- [ ] Interception de trafic
- [ ] DNS spoofing

### Phase 6 — Password Tools ⏳ 0%

- [ ] Hash cracker (MD5, SHA1, SHA256)
- [ ] Analyseur d'entropie de mots de passe
- [ ] Password spraying Active Directory
- [ ] Kerberoasting (si AD déployé)

### Phase 7 — Attaques Web ⏳ 0%

- [ ] Déploiement de DVWA via Docker
- [ ] Scanner SQLi basique
- [ ] Scanner XSS basique
- [ ] Analyse de headers HTTP de sécurité

### Phase 8 — 🛡️ Blue Team (SIEM) ⏳ 0%

- [ ] Installation de Wazuh
- [ ] Configuration des agents sur les VMs
- [ ] Création de règles de détection custom
- [ ] Détection de scans Red team en temps réel

### Phase 9 — Durcissement & Dockerisation ⏳ 0%

- [ ] Hardening pfSense (règles firewall strictes)
- [ ] GPO sur Active Directory
- [ ] Dockerisation de la toolbox Red team
- [ ] CLI unifiée en une seule commande

### Phase 10 — Rapport & Portfolio ⏳ 0%

- [ ] Rapport de pentest format pro (PDF)
- [ ] Vidéo démo du lab
- [ ] README final soigné avec screenshots
- [ ] Publication LinkedIn / Medium

---

## 🎯 Prochaines actions prioritaires

1. **Télécharger les ISOs** (pfSense, Metasploitable, Kali, Windows Server 2022)
2. **Créer le dossier** `C:\Users\Anthony\VMs\ISOs\`
3. **Installer pfSense** (première VM du lab)

---

## 📊 Stats

| Indicateur | Valeur |
|---|---|
| Phases complétées | 0 / 11 |
| Phases en cours | 1 (Phase 0) |
| Sessions de travail | 1 |
| Date de début | Septembre 2026 |
| Date de reprise | 09 novembre 2026 |
