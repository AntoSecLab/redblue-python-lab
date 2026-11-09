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

## 📅 Session #2 — 09-10 novembre 2026 (nuit) — Construction du lab

### Contexte
Suite de la session #1 du matin. Objectif : passer des configurations réseau théoriques à la construction réelle du lab, en commençant par le routeur pfSense puis l'attaquant Kali.

### Objectifs
- [x] Réparer définitivement le VMnet Red (encore bloqué en APIPA au début)
- [x] Télécharger les ISOs restantes (pfSense, Kali, Metasploitable 2)
- [x] Organiser les ISOs dans `C:\Users\Anthony\VMs\ISOs\`
- [x] Créer et installer la VM pfSense-Router (5 cartes réseau)
- [x] Configurer les 5 interfaces pfSense (WAN + 4 LANs)
- [x] Activer le DHCP server sur Clients, Blue, Red (statique pour Servers)
- [x] Créer la VM Kali-Linux-Red (2 cartes réseau : NAT + Red)
- [ ] Installation Kali en cours au moment de cette entrée

### Décisions techniques
- **pfSense** choisi comme routeur central : open-source, basé FreeBSD, standard de l'industrie pour les labs cyber.
- **Architecture 5 interfaces pfSense** :
  - WAN (em0) → VMnet8 NAT → 192.168.56.229/24 (DHCP VMware)
  - LAN (em1) → VMnet10 Clients → 192.168.10.1/24 (DHCP activé, plage .100-.200)
  - OPT1 (em2) → VMnet3 Servers → 192.168.20.1/24 (pas de DHCP, IPs statiques)
  - OPT2 (em3) → VMnet4 Blue → 192.168.30.1/24 (DHCP activé)
  - OPT3 (em4) → VMnet5 Red → 192.168.99.1/24 (DHCP activé)
- **Kali avec 2 NICs** : eth0 sur NAT pour les updates/git, eth1 sur Red pour le lab isolé.
- **Partitionnement Kali** : "Assisté - disque entier - tout dans une seule partition", choix débutant adapté à un lab.
- **Pack d'outils Kali** : top10 + default (environ 600 outils), largement suffisant pour nos 10 phases.
- **Accès à l'interface web pfSense** : choisi de passer par Kali (approche réaliste pentester) plutôt que depuis Windows (évite le conflit d'IP entre VMnet10 côté Windows et LAN pfSense).

### Difficultés rencontrées
- **Piège Netgate** : le site pfSense pousse vers "Netgate Installer" (outil de maintenance hardware) au lieu du vrai pfSense CE. Solution : passer par le miroir direct https://atxfiles.netgate.com/mirror/downloads/
- **Confusion DHCP client vs DHCP server** dans la config pfSense console. "Configure IPv4 via DHCP?" = recevoir une IP (à répondre n pour les LANs), "Enable DHCP server?" = distribuer des IPs (à répondre y pour les LANs clients).
- **Erreur "Invalid interface!"** : tapé une IP (192.168.10.1) au lieu du numéro d'interface (2) dans le menu Set interface IP address. Corrigé en relançant proprement l'option 2.
- **VMware non lancé en admin** au départ → boutons grisés dans Virtual Network Editor. Fix : bouton "Change Settings" en bas pour demander l'élévation.

### Apprentissages du jour
- **Les interfaces Ethernet dans Linux (eth0, eth1)** suivent l'ordre d'ajout des cartes réseau dans VMware. Idem avec `em0, em1` dans pfSense (FreeBSD).
- **DHCP client ≠ DHCP server** : notion fondamentale. Un routeur peut être les deux à la fois (client sur WAN, server sur LAN).
- **Convention de plage DHCP** : garder les premières IPs (.1 à .99) pour le statique, utiliser .100-.200 pour le dynamique. Standard d'entreprise.
- **Segmentation réseau** = défense n°1 en entreprise. Nos 4 VLANs (Clients/Servers/Blue/Red) reproduisent une architecture réelle.
- **ISO ≠ VM pré-faite** : Metasploitable 2 est fournie comme VM complète (fichiers .vmx + .vmdk), à ouvrir directement dans VMware, pas à installer.

### Prochaine session
- Finaliser l'installation Kali (configuration réseau post-install)
- Accéder à l'interface web pfSense (https://192.168.10.1) depuis Kali
- Installer et démarrer Metasploitable 2
- Premier test de connectivité : ping entre Kali et Metasploitable via pfSense
- **Préparer la Phase 2 : écriture du premier script Python (ping scanner)**

---