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
- [x] Préparer la liste des ISOs à télécharger

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
- [x] Installation Kali en cours au moment de cette entrée

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

## 📅 Session #3 — 10 novembre 2026 (après-midi) — Accès admin pfSense via Red

### Contexte
Suite de la nuit. Kali était installée et fonctionnelle, mais impossible d'accéder à l'interface web pfSense depuis le VLAN Red. Objectif : comprendre pourquoi, débloquer l'accès admin depuis Red, et nettoyer l'architecture Kali pour revenir à 2 NICs.

### Objectifs
- [x] Diagnostiquer pourquoi pfSense ne répond pas sur 192.168.99.1 depuis Kali
- [x] Configurer NetworkManager pour gérer proprement les 2 cartes réseau Kali
- [x] Ajouter temporairement une 3e carte réseau sur Kali (VLAN Clients) pour accéder à pfSense
- [x] Se connecter à l'interface web pfSense pour la première fois
- [x] Renommer les interfaces pfSense (OPT1→SERVERS, OPT2→BLUE, OPT3→RED)
- [x] Créer la règle firewall "Allow RED to anywhere" pour débloquer l'admin depuis Red
- [x] Changer le mot de passe admin par défaut
- [x] Supprimer la 3e carte réseau Kali (retour à l'archi propre à 2 NICs)
- [x] Validation finale : accès complet à pfSense depuis Red (eth1)

### Décisions techniques
- **Création de 2 connexions NetworkManager dédiées** sur Kali (`NAT-Internet` pour eth0 et `Red-Lab` pour eth1) au lieu d'une connexion partagée qui créait des conflits (dès qu'on connectait eth0, eth1 se déconnectait, et vice-versa).
- **Approche "ajout temporaire d'une 3e NIC"** pour bootstrapper l'accès pfSense : la règle anti-lockout de pfSense n'autorise que le LAN natif pour l'admin par défaut. On a ajouté eth2 sur VLAN Clients pour arriver à l'interface web, créer la règle firewall autorisant RED, puis retiré eth2.
- **Règle firewall RED → anywhere** choisie plutôt qu'une règle restrictive (RED → pfSense HTTPS uniquement). Justification : le VLAN Red est celui de l'attaquant, il doit pouvoir tout faire. Les restrictions s'appliquent aux autres VLANs (Clients, Servers).
- **Mot de passe admin changé** : `pfsense` (default) → `adminkalired`. Les mots de passe par défaut sont une des vulnérabilités les plus exploitées en cyber.

### Difficultés rencontrées
- **Conflit NetworkManager "Wired connection 1"** : une connexion générique essayait d'être assignée à plusieurs interfaces simultanément. Fix : suppression + création de connexions dédiées par interface.
- **pfSense ne répondait pas au ping depuis Kali** malgré un ARP qui fonctionnait → c'était bien le firewall qui droppait ICMP sur les interfaces OPT par défaut. Comportement correct en sécurité (deny by default).
- **Rate-limiting / scan detection pfSense** : après plusieurs tests nmap rapprochés, les ports étaient affichés "filtered" au 2e scan. pfSense détecte les scans aggressifs et met temporairement l'IP en quarantaine. Le fix : attendre 5 min ou scanner avec `nmap -T2` (slow scan).
- **Login pfSense refusé après changement de mdp** : probable erreur de frappe dans le champ password invisible. Fix : option 3 de la console pfSense pour reset le mdp à default `pfsense`, puis changement propre via l'interface web avec copier-coller.
- **Caractère `|` (pipe) impossible à taper dans la console VMware** (clavier QWERTY par défaut, incompatible AZERTY). Fix : utiliser SSH depuis Kali pour taper dans le shell pfSense, ou utiliser des commandes sans pipe.

### Apprentissages du jour
- **ARP vs ICMP vs HTTPS = 3 couches réseau différentes** (L2/L3/L7). Un hôte peut répondre à ARP mais bloquer ICMP (classique en sécurité). Diagnostiquer couche par couche.
- **La règle anti-lockout de pfSense** n'autorise par défaut que le LAN natif pour l'admin. Important à comprendre pour tout nouveau VLAN.
- **Les tables PF (`pfctl -t <table>`)** stockent les IPs bannies temporairement (virusprot, sshguard, bruteforce, webConfiguratorlockout). `pfctl -ss` montre les connexions actives.
- **Enable SSH sur pfSense (option 14)** = réflexe pro pour administrer à distance avec un vrai clavier Linux.
- **NetworkManager recrée automatiquement une connexion** sur toute interface vue "libre". Pour vraiment supprimer une interface : la retirer côté hyperviseur.
- **`nmcli connection add/delete/up`** = commandes de base pour gérer les profils réseau sur Kali/Ubuntu moderne.
- **"Deny by default, allow by exception"** = principe fondamental du firewall. Chaque règle qu'on ajoute doit être justifiée.

### Prochaine session
- Installer et démarrer Metasploitable 2 sur VLAN Servers
- Configurer son IP statique (192.168.20.100)
- Créer une règle firewall RED → SERVERS pour autoriser Kali à atteindre Metasploitable
- Premier ping Kali → Metasploitable via pfSense (validation du routage multi-VLAN)
- **🐍 PHASE 2 : écrire le premier script Python — un ping scanner**

---
## 📅 Session #4 — 12 novembre 2026 — Lab Packet Tracer en parallèle

### Contexte
Décision prise de construire un lab Cisco Packet Tracer en parallèle de l'infra VMware réelle. Objectif : approfondir la compréhension des concepts réseau (VLANs, routing, firewall) côté Cisco IOS, qui est le standard du marché. Les 2 labs sont indépendants mais se renforcent pédagogiquement.

### Objectifs
- [x] Créer la topologie physique (routeur 1941 + switch 2960 + 4 PCs)
- [x] Sécuriser routeur et switch (enable secret, console, VTY, service password-encryption)
- [x] Créer les 4 VLANs (CLIENTS, SERVERS, BLUE, RED)
- [x] Assigner les ports access aux VLANs
- [x] Configurer le trunk 802.1Q (Fa0/1 vers le routeur)
- [x] Implémenter le router-on-a-stick (sous-interfaces dot1Q)
- [x] Configurer DHCP sur le routeur (4 pools)
- [x] Implémenter une ACL extended pour bloquer RED → SERVERS
- [x] Valider tous les tests (pings OK/KO selon ACL)
- [x] Documenter l'ensemble dans un README dédié
- [x] Push du lab sur GitHub

### Décisions techniques
- **Choix Packet Tracer v9.0** (dernière version) pour accéder aux features modernes.
- **Routeur 1941** choisi comme équivalent pfSense — standard CCNA et compatible avec tous les tutos.
- **ACL numérotée 150** (extended, plage 100-199) pour pouvoir filtrer sur src+dest+protocole.
- **Approche "deny Red→Servers + permit all"** pour simuler un durcissement réaliste sans tout bloquer.
- **DHCP excluded-address 1-99** sur chaque VLAN pour laisser la plage statique pour les équipements critiques.

### Difficultés rencontrées
- **Confusion initiale "Reply from X"** : le message "Reply from 192.168.99.1: Destination host unreachable" peut faire croire que le ping passe, alors que c'est le routeur qui dit "je refuse". Clé : regarder `Received = 0` dans les stats.
- **ACL qui semblait inactive au début** : résolu avec un `no ip access-group` + `ip access-group` pour force le reload sur l'interface.
- **Faute de frappe "temrinal"** au lieu de "terminal" dans la CLI : Cisco affiche un petit `^` sous le caractère erroné → feature de debug super pratique.

### Apprentissages du jour
- **ARP vs ICMP** : ping n'est pas ARP, mais ARP est nécessaire une fois avant le premier ping pour résoudre la MAC de la gateway.
- **TTL dans ping** : Windows=128, Linux=64, décrémenté de 1 à chaque routeur traversé. Permet d'identifier l'OS cible en pentest.
- **Router-on-a-stick** : un seul câble entre routeur et switch peut gérer N VLANs avec les sous-interfaces 802.1Q.
- **DHCP client vs server** : un routeur peut être les deux à la fois (client sur WAN, serveur sur LAN).
- **MAC spoofing** : les adresses MAC peuvent être modifiées côté OS, donc les filtres basés sur MAC ne sont pas fiables.
- **Les ACL ont un deny implicite** : toujours finir par `permit ip any any` si on veut juste bloquer un flux ciblé.
- **"Destination host unreachable"** = routeur qui bloque. **"Request timed out"** = silence (firewall silencieux, plus discret en prod).
- **Protocoles maîtrisés** : DHCP (67/68 UDP), DNS (53 UDP), HTTP (80 TCP), HTTPS (443 TCP), ARP (L2), ICMP (L3).

### Documentation produite
- PDF cheatsheet "Mots de passe Cisco IOS" (6 pages) dans `docs/cheatsheets/cisco_passwords.pdf`
- PDF cheatsheet "Protocoles réseau essentiels" (6 pages + vocabulaire anglais) dans `docs/cheatsheets/protocoles_reseau.pdf`
- Fichier d'import Anki avec ~55 fiches de révision (vocabulaire français/anglais)
- Fichier `packet-tracer/lab-v1-vlans.pkt` avec le lab complet
- `packet-tracer/README.md` détaillé documentant l'architecture et les configs

### Prochaine session
- Retour VMware : installer Metasploitable 2 sur le VLAN Servers
- Configurer son IP statique (192.168.20.100)
- Premier ping Kali → Metasploitable via pfSense (validation du routage réel)
- **🐍 PHASE 2 : écrire le PREMIER SCRIPT PYTHON — un ping scanner**
- Extension futures Packet Tracer : ajout d'un "Red externe" (attaquant depuis internet simulé)

---

## 📅 Session #5 — 12 novembre 2026 (soirée) — Validation routage inter-VLAN Kali ↔ Metasploitable

### Contexte
Session de troubleshooting intense pour finaliser le lab VMware. Objectif de validation : un ping fonctionnel entre Kali (VLAN Red) et Metasploitable (VLAN Servers) en passant par pfSense. Session marathon de 3h qui a nécessité plusieurs cycles de debug (firewall, routage, table de routes Linux).

### Objectifs
- [x] Résoudre le blocage Kali → pfSense (ping 192.168.99.1)
- [x] Installer et configurer Metasploitable 2 sur le VLAN Servers
- [x] Attribuer une IP statique à Meta (192.168.20.100)
- [x] Valider le routage pfSense entre VLANs Red et Servers
- [x] Obtenir un ping fonctionnel Kali → Meta (via pfSense)
- [x] Comprendre et documenter chaque problème rencontré

### Décisions techniques
- **Choix "deny by default" assumé** : plutôt que créer une règle "Allow RED to any", on crée 3 règles granulaires (HTTPS admin, ICMP, Servers pentest) pour appliquer le principe du moindre privilège.
- **Route statique côté Kali plutôt que suppression du default gateway** : solution plus propre qu'un reconfiguration DHCP complète, et plus pédagogique.
- **Metasploitable en IP statique** : pas de DHCP sur l'interface SERVERS de pfSense (pour l'instant), configuration manuelle via `ifconfig` + `route add`.
- **Pas de règle sortante sur SERVERS pour ce soir** : respect de la logique "un serveur n'initie pas de connexions vers un attaquant", à ajouter plus tard uniquement si nécessaire (Meta → Internet pour updates, Meta → Blue pour logs Wazuh).

### Difficultés rencontrées

#### 🔴 Problème #1 : Firefox fonctionne vers pfSense mais pas le ping
- **Symptôme** : `curl` et navigateur sur 192.168.99.1 → OK, mais `ping 192.168.99.1` → 100% loss.
- **Cause** : pfSense bloque TOUT par défaut sur les interfaces OPT. Seule une règle "Allow HTTPS admin" avait été créée (port 443 TCP), autorisant l'accès web mais pas l'ICMP.
- **Solution** : Création de la règle "Allow RED ICMP (ping)" avec Protocol=ICMP, Destination=any.

#### 🔴 Problème #2 : Kali ping pfSense OK mais pas Metasploitable
- **Symptôme** : `ping 192.168.99.1` depuis Kali → OK, mais `ping 192.168.20.100` → 100% loss.
- **Hypothèses testées** :
  1. Règle firewall "Allow RED to Servers" manquante → créée, toujours KO
  2. Metasploitable mal configuré → vérifié `ifconfig` + `route -n`, IP et gateway OK
  3. pfSense ne route pas entre VLANs → diagnostic ping depuis pfSense (Diagnostics → Ping, Source=RED) → **OK 4/4**
- **Breakthrough** : `tcpdump -i eth0 icmp` sur Metasploitable → **AUCUN paquet n'arrivait**. Donc le paquet se perd AVANT d'atteindre Meta.
- **Investigation table de routage Kali** : `ip route show` révèle **DEUX default gateways** :
  - `default via 192.168.56.2 dev eth0 metric 100` (NAT)
  - `default via 192.168.99.1 dev eth1 metric 101` (Red)
- **Cause racine** : Linux priorise la route avec la metric la plus basse (eth0 metric 100). Donc tout trafic vers un réseau inconnu (192.168.20.0/24) partait vers le NAT au lieu de pfSense Red.
- **Solution** : Ajout d'une route statique spécifique :
  ```bash
  sudo ip route add 192.168.20.0/24 via 192.168.99.1 dev eth1
  ```

#### 🔴 Problème #3 : Login pfSense refusé après changement de mot de passe
- **Symptôme** : impossible de se reconnecter au Web UI après changement du mdp admin.
- **Cause** : probable erreur de saisie lors du changement (caractères spéciaux, clavier).
- **Solution** : reset via console (option 3), puis changement du mdp avec copier-coller.

### Apprentissages du jour

#### 🧠 Firewall
- **pfSense = deny by default** sur toutes les interfaces OPT. Chaque flux légitime DOIT être autorisé par une règle explicite.
- Un **firewall stateful** (comme pfSense) autorise automatiquement les paquets de **retour** d'une connexion initiée. Donc une règle unidirectionnelle (A→B) suffit pour le ping aller-retour.
- Les **logs firewall** (Status → System Logs → Firewall) sont l'outil n°1 pour voir ce qui est bloqué en temps réel.
- **Diagnostics → Ping** avec `Source address` custom permet de tester un routage depuis une interface spécifique du pfSense : outil pro de validation.

#### 🧠 Réseau Linux
- **Table de routage** = GPS du système. Chaque paquet consulte cette table pour savoir par quelle interface sortir.
- **Metric** = score de priorité d'une route. **Plus petit = plus prioritaire**. En cas de 2 default gateways, Linux prend celle avec la metric la plus basse.
- Une **route statique** (`ip route add`) force un réseau spécifique à passer par une gateway/interface donnée, même si d'autres routes existent.
- **Attention** : les routes ajoutées à la volée avec `ip route add` sont **en RAM uniquement**. Au reboot, elles disparaissent. Pour les rendre persistantes, il faut éditer `/etc/network/interfaces` ou utiliser NetworkManager.

#### 🧠 Debug
- **tcpdump est roi**. Un `tcpdump -i <interface> icmp` qui ne montre RIEN pendant un ping → le paquet n'arrive JAMAIS → le problème est en amont (routage source).
- **TTL dans ping** : décrémente de 1 à chaque routeur traversé. Un ping avec `ttl=63` au lieu de 64 prouve mathématiquement que le paquet a traversé **1 routeur**.
- **Dichotomie du debug** : tester chaque segment indépendamment (A→Router OK ? Router→B OK ? Alors A→B devrait l'être. Si non → problème de routage ou firewall entre A et Router).

#### 🧠 Vocabulaire anglais
| Anglais | Français |
|---|---|
| routing table | table de routage |
| default gateway | passerelle par défaut |
| metric | métrique (score de route) |
| static route | route statique |
| deny by default | bloquer par défaut |
| stateful firewall | pare-feu à état |
| inter-VLAN routing | routage inter-VLAN |
| packet loss | perte de paquets |
| echo request / echo reply | requête / réponse ICMP |

### Documentation produite
- Schémas ASCII de la topologie et du routage (problème + solution)
- Diagrammes de table de routage Kali (pré/post fix)
- Explication pédagogique de la metric et des default gateways multiples
- Vocabulaire anglais networking ajouté aux fiches Anki

### Validation finale
```
$ ping -c 4 192.168.20.100
PING 192.168.20.100 (192.168.20.100) 56(84) bytes of data.
64 bytes from 192.168.20.100: icmp_seq=1 ttl=63 time=1.21 ms
64 bytes from 192.168.20.100: icmp_seq=2 ttl=63 time=1.98 ms
64 bytes from 192.168.20.100: icmp_seq=3 ttl=63 time=1.14 ms
64 bytes from 192.168.20.100: icmp_seq=4 ttl=63 time=1.37 ms

--- 192.168.20.100 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
```

**✅ Lab VMware 100% fonctionnel : Kali → pfSense → Metasploitable validé.**

### Prochaine session (#6)
- Rendre persistantes : route statique Kali + IP statique Meta (survie au reboot)
- Explorer les services Metasploitable exposés (nmap)
- **🐍 PHASE 2 : écrire le PREMIER SCRIPT PYTHON — un ping scanner**
- Objectif : scanner tout le VLAN Servers en Python et détecter les hôtes actifs

---