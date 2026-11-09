# 📘 Journal de bord — redblue-python-lab

> Journal chronologique de mon apprentissage de Python appliqué à la cybersécurité, à travers la construction d'un lab Red Team vs Blue Team.
>
> Chaque entrée documente une session de travail : contexte, objectifs, décisions techniques, difficultés rencontrées, apprentissages.

---

## 🎯 À propos du projet

- **Objectif pédagogique** : apprendre Python from scratch dans un contexte cybersécurité
- **Objectif technique** : construire un lab réseau d'entreprise réaliste (VLANs, AD, DNS, DHCP) et développer des outils offensifs (Red) puis défensifs (Blue) en Python
- **Approche choisie** : tout le développement se fait **directement dans les VMs** (Kali principalement), pour une immersion totale dans l'environnement cyber réel

---

## 📅 Session #1 — 09 novembre 2026 (reprise du projet)

### Contexte
Reprise du projet après plusieurs mois d'interruption. Objectif de cette session : refaire le point sur l'existant, réparer les éléments cassés, et poser les bases pour la suite.

### Objectifs
- [x] Vérifier l'état des 4 VMnets VMware (Clients, Servers, Blue, Red)
- [x] Réparer le VMnet Red qui affichait une IP APIPA (169.254.x.x)
- [x] Mettre en place les fichiers de suivi (JOURNAL.md + PROGRESS.md)
- [ ] Préparer la liste des ISOs à télécharger

### Décisions techniques
- **Approche "Full VM"** retenue : le développement et l'exécution se feront intégralement dans les VMs. VSCode, Python, Git seront installés dans Kali, pas utilisés en Remote SSH depuis le PC hôte.
- **Justification** : immersion dans l'environnement Linux pour maximiser l'apprentissage des outils cyber natifs.

### Difficultés rencontrées
- **VMware non lancé en admin** → tous les boutons de Virtual Network Editor étaient grisés. Solution : bouton "Change Settings" → UAC → fenêtre se réouvre avec les privilèges admin.
- **VMnet Red bloqué en APIPA (169.254.x.x)** → la carte réseau virtuelle Windows n'avait pas récupéré l'IP configurée. Fix : décocher puis recocher "Connect a host virtual adapter to this network" dans Virtual Network Editor.

### Apprentissages du jour
- **IP en 169.254.x.x = APIPA** = signe que le DHCP ou la config statique a échoué. Signal d'alarme réseau à reconnaître.
- **VMware doit TOUJOURS être lancé en admin** pour modifier la config réseau. Astuce : clic droit sur raccourci → Propriétés → Compatibilité → cocher "Exécuter en tant qu'administrateur".
- **"Change Settings" dans Virtual Network Editor** = demande d'élévation de privilèges en cours de session, sans relancer VMware.

### Prochaine session
- Télécharger les ISOs nécessaires : pfSense CE, Metasploitable 2, Kali Linux, Windows Server 2022 Evaluation.
- Créer le dossier `C:\Users\Anthony\VMs\ISOs\` pour centraliser les images d'installation.
- Commencer la Phase 1 : installation de pfSense (le routeur du lab).

---
