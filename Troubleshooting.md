# 🔧 Troubleshooting — Les pannes rencontrées et leurs solutions

> Ce document recense **toutes les galères techniques** rencontrées lors de la construction du lab, avec **le diagnostic détaillé** et **la solution appliquée**.
>
> Objectif : servir de référence pour éviter de refaire les mêmes erreurs, et comprendre les mécanismes sous-jacents de chaque problème.

---

## 📋 Index des problèmes rencontrés

| # | Problème | Composant | Solution courte |
|---|---|---|---|
| 1 | VMnet Red bloqué en APIPA (169.254.x.x) | VMware | Décocher/recocher host virtual adapter |
| 2 | Boutons grisés dans Virtual Network Editor | VMware | Cliquer "Change Settings" pour UAC |
| 3 | Mauvais fichier téléchargé pour pfSense | pfSense | Passer par le miroir direct Netgate |
| 4 | Pipe `\|` impossible à taper dans console pfSense | pfSense | Utiliser SSH depuis Kali |
| 5 | Confusion DHCP client vs DHCP server | pfSense | Toujours lire "via DHCP?" vs "Enable DHCP server?" |
| 6 | `dhclient: commande introuvable` sur Kali | Kali | Kali moderne utilise NetworkManager (`nmcli`) |
| 7 | eth0 et eth1 se déconnectent mutuellement | NetworkManager | Créer des connexions dédiées par interface |
| 8 | Fichier `~/.ssh/config` nommé `config.txt` par Notepad | Windows | Créer le fichier via PowerShell avec Out-File |
| 9 | Clé SSH mappée au mauvais compte GitHub | SSH | Fichier `~/.ssh/config` avec IdentityFile explicite |
| 10 | Timeout sur `curl https://192.168.99.1` depuis Red | pfSense Firewall | Règle Pass sur interface RED |
| 11 | Rate-limiting pfSense après scans nmap répétés | pfSense | Attendre 5 min, scanner avec `nmap -T2` |
| 12 | Login pfSense refusé après changement de mdp | pfSense | Reset via console (option 3), puis copier-coller |
| 13 | Conflit IP Windows ↔ pfSense sur VMnet3/5 | VMware | Décocher "Connect a host virtual adapter" sur toutes les VMnets |
| 14 | Firefox vers pfSense OK mais ping bloqué | pfSense Firewall | Ajouter règle "Allow RED ICMP (ping)" spécifique |
| 15 | Kali ping pfSense OK mais pas Metasploitable | Routage Linux | Route statique forcée via eth1 (metric priority) |

---

## 🔴 Problème 1 — VMnet Red bloqué en APIPA (169.254.x.x)

### Symptôme
Après création des VMnets dans VMware, la carte réseau virtuelle VMnet5 (Red) côté Windows affichait une IP en `169.254.97.203` au lieu de `192.168.99.1`.

### Diagnostic
L'IP `169.254.x.x` est une **APIPA** (Automatic Private IP Addressing). C'est l'adresse que Windows s'attribue automatiquement **quand il n'arrive pas à obtenir l'IP configurée**. Signal d'alarme réseau classique.

### Cause
VMware n'avait pas correctement appliqué la configuration de l'interface virtuelle Windows → la carte était "sèche" et Windows s'est auto-configuré en APIPA.

### Solution appliquée
Dans **Virtual Network Editor** :
1. Sélectionner la ligne **Red**
2. **Décocher** "Connect a host virtual adapter to this network" → Apply
3. Attendre 3 secondes
4. **Recocher** cette case → Apply
5. OK pour fermer

Puis vérifier dans PowerShell : `ipconfig | Select-String VMnet5` → doit afficher `192.168.99.1`.

### Apprentissage
**Toute IP en `169.254.x.x` est un indicateur de problème réseau.** C'est un réflexe à acquérir : en cybersécurité, voir cette plage = enquêter immédiatement.

---

## 🔴 Problème 2 — Boutons grisés dans Virtual Network Editor

### Symptôme
Impossible de modifier les VMnets : tous les boutons (Add Network, Apply, options…) sont grisés dans Virtual Network Editor.

### Cause
VMware Workstation n'a pas été lancé avec les **privilèges administrateur**. La modification de la pile réseau virtuelle nécessite ces droits sous Windows.

### Solution appliquée
**Option A** : Fermer VMware, clic droit sur le raccourci → **Exécuter en tant qu'administrateur**.

**Option B** (plus rapide) : Dans la fenêtre Virtual Network Editor, en bas à droite, cliquer sur le bouton **"Change Settings"** → UAC apparaît → Oui. La fenêtre se réouvre avec les droits admin, tous les boutons sont actifs.

**Option pérenne** : Clic droit sur le raccourci VMware → Propriétés → Compatibilité → cocher **"Exécuter ce programme en tant qu'administrateur"**. Comme ça, VMware se lance toujours en admin.

### Apprentissage
Les outils réseau sur Windows nécessitent **presque toujours** des droits admin (VMware, Wireshark, scapy, etc.). Prendre le réflexe de les lancer en admin.

---

## 🔴 Problème 3 — Mauvais fichier téléchargé pour pfSense

### Symptôme
Au lieu de télécharger `pfSense-CE-2.7.2-RELEASE-amd64.iso.gz`, on a récupéré `netgate-installer-v1.2-RELEASE-amd64.iso.gz`, qui est un outil de maintenance pour les boîtiers physiques Netgate, pas pfSense lui-même.

### Cause
Le site Netgate présente plusieurs boutons de téléchargement qui mettent des produits différents dans un "panier à 0$". Sans vérification du nom du fichier dans le panier, on peut facilement télécharger le mauvais.

### Solution appliquée
Passage par le **miroir direct Netgate** (interface FTP brute, aucune confusion possible) :

**URL** : https://atxfiles.netgate.com/mirror/downloads/

Cliquer directement sur `pfSense-CE-2.7.2-RELEASE-amd64.iso.gz`.

### Apprentissage
**Toujours vérifier le nom du fichier téléchargé avant de l'utiliser.** En cyber, un attaquant peut te faire télécharger `facture.pdf.exe` à la place du vrai PDF. Même logique, même vigilance.

---

## 🔴 Problème 4 — Pipe `|` impossible à taper dans la console pfSense

### Symptôme
Dans la console VMware de pfSense (qui tourne en QWERTY par défaut), impossible de taper le caractère `|` pour faire un `pfctl -ss | grep ...`.

### Cause
La console pfSense est juste une capture d'écran du BIOS/console, pas un terminal interactif. Elle utilise le layout clavier **US (QWERTY)** par défaut. Sur un clavier **AZERTY français**, les touches sont mappées différemment.

### Solution appliquée
**Activation du SSH sur pfSense** et connexion depuis Kali :

1. Dans la console pfSense, menu principal → **option 14** → `y` pour activer SSH
2. Depuis Kali : `ssh admin@192.168.10.1`
3. Entrer le mot de passe
4. Maintenant on peut utiliser le `|` normalement (clavier Linux en AZERTY)

### Apprentissage
**SSH est le standard pour administrer un équipement réseau.** La console physique (ou VM) sert uniquement au bootstrap initial. Dès que SSH est accessible, on l'utilise.

---

## 🔴 Problème 5 — Confusion DHCP client vs DHCP server

### Symptôme
Lors de la configuration des interfaces pfSense (option 2 du menu), répondre `y` à la question `Configure IPv4 via DHCP?` → l'interface LAN n'a plus d'IP statique.

### Cause
Deux questions se ressemblent mais ont des sens opposés :

| Question | Signification | Réponse pour LAN |
|---|---|---|
| `Configure IPv4 via DHCP?` | pfSense doit-il **recevoir** son IP d'un DHCP ? | **n** (statique) |
| `Enable DHCP server on LAN?` | pfSense doit-il **distribuer** des IPs ? | **y** |

Confondre les deux = catastrophe.

### Solution appliquée
Relancer option 2 sur l'interface, répondre **n** à "Configure via DHCP", puis saisir manuellement l'IP, le CIDR, etc.

### Apprentissage
**DHCP client ≠ DHCP server.** Un routeur peut être les deux à la fois (client sur WAN pour obtenir une IP du FAI, serveur sur LAN pour distribuer des IPs aux postes). Toujours lire la question AVANT de répondre y/n.

---

## 🔴 Problème 6 — `dhclient: commande introuvable` sur Kali

### Symptôme
Tentative de `sudo dhclient -v eth0` pour forcer une IP DHCP → erreur `sudo: dhclient : commande introuvable`.

### Cause
Les versions récentes de Kali (2023+) **n'utilisent plus `dhclient`**. Elles utilisent **NetworkManager** via la commande `nmcli`.

### Solution appliquée
```bash
# Lister les interfaces et leur état
nmcli device status

# Activer une interface déconnectée
sudo nmcli device connect eth0

# Vérifier
ip addr show eth0
```

### Apprentissage
**Les commandes Linux évoluent** avec les distributions modernes. `dhclient` → `nmcli`, `ifconfig` → `ip`, `netstat` → `ss`. Les tutos de 2020 peuvent être obsolètes en 2026. Toujours vérifier avec `<commande> --help` si doute.

---

## 🔴 Problème 7 — eth0 et eth1 se déconnectent mutuellement

### Symptôme
Sur Kali, quand on fait `sudo nmcli device connect eth0`, l'interface eth1 se déconnecte automatiquement (et vice-versa).

### Cause
NetworkManager avait **une seule connexion générique** nommée `Wired connection 1`, qu'il essayait d'assigner à l'interface active. Une connexion = une interface à la fois.

### Solution appliquée
Création de **2 connexions dédiées**, une par interface :

```bash
# Supprimer la connexion générique
sudo nmcli connection delete "Wired connection 1"

# Créer une connexion pour eth0 (NAT/Internet)
sudo nmcli connection add type ethernet ifname eth0 con-name "NAT-Internet" autoconnect yes

# Créer une connexion pour eth1 (Red/Lab)
sudo nmcli connection add type ethernet ifname eth1 con-name "Red-Lab" autoconnect yes

# Activer les deux
sudo nmcli connection up "NAT-Internet"
sudo nmcli connection up "Red-Lab"

# Vérifier
nmcli device status
```

### Apprentissage
Sur une VM avec plusieurs cartes réseau, **toujours créer des connexions nommées et dédiées**. Noms parlants (NAT-Internet, Red-Lab) = debug plus facile dans 6 mois.

---

## 🔴 Problème 8 — Fichier `~/.ssh/config` nommé `config.txt` par Notepad

### Symptôme
Après création du fichier `config` via Notepad, SSH ne reconnaissait pas la configuration (continuait à utiliser la mauvaise clé).

### Cause
**Notepad ajoute automatiquement `.txt`** à tout fichier qu'il crée, sauf si on force l'extension avec des guillemets. Résultat : `config.txt` au lieu de `config`.

SSH cherche strictement `~/.ssh/config` sans extension.

### Solution appliquée
Création du fichier directement via PowerShell :

```powershell
@"
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_antoseclab
    IdentitiesOnly yes
"@ | Out-File -FilePath $env:USERPROFILE\.ssh\config -Encoding ascii -NoNewline
```

### Apprentissage
**Ne jamais utiliser Notepad** pour créer des fichiers de config sans extension. Utiliser VSCode, Notepad++ ou PowerShell. Activer "Afficher les extensions de fichier" dans l'Explorateur Windows.

---

## 🔴 Problème 9 — Clé SSH mappée au mauvais compte GitHub

### Symptôme
`ssh -T git@github.com` retournait `Hi AnthonySupinfo!` au lieu de `Hi AntoSecLab!`, même après avoir ajouté la bonne clé sur AntoSecLab.

### Cause
SSH essaie les clés dans un ordre précis (défini par OpenSSH). La clé `id_ed25519` (ancienne, liée à AnthonySupinfo) était testée **avant** `id_ed25519_antoseclab`. GitHub acceptait la première clé reconnue.

### Solution appliquée
Fichier `~/.ssh/config` avec `IdentitiesOnly yes` pour forcer SSH à n'utiliser que la clé spécifiée :

```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_antoseclab
    IdentitiesOnly yes
```

Le `IdentitiesOnly yes` est **crucial** : sans lui, SSH propose la clé spécifiée PUIS continue avec les autres clés disponibles.

### Apprentissage
`ssh -v git@github.com` est **la commande de debug SSH ultime**. Elle montre quelles clés sont testées dans quel ordre. Compétence indispensable pour tout pentester.

---

## 🔴 Problème 10 — Timeout sur `curl https://192.168.99.1` depuis Red

### Symptôme
Depuis Kali (eth1 sur VLAN Red), impossible d'accéder à l'interface web pfSense sur son IP Red (192.168.99.1). Le ping passait, mais HTTPS timeout.

### Cause
Par défaut, pfSense **bloque TOUT trafic entrant** sur les interfaces OPT (OPT1/2/3 renommées SERVERS/BLUE/RED). Même l'accès à son propre webConfigurator admin est bloqué depuis ces interfaces.

La règle anti-lockout n'autorise que **l'interface LAN "officielle"** à accéder à l'admin.

### Solution appliquée (initiale — session #3)
Création d'une règle firewall "allow all" sur l'interface RED (pour débloquer rapidement).

### Solution finale (session #5, best practice)
Remplacement par **3 règles granulaires** (voir Problème #14) :
- Allow RED to pfSense HTTPS (admin) — port 443 TCP uniquement
- Allow RED ICMP (ping) — pour les diagnostics
- Allow RED to Servers (pentest) — accès ciblé au VLAN Servers

### Apprentissage
**"Deny by default, allow by exception"** est le principe fondamental des firewalls. Une règle "allow all" est pratique en debug mais **doit être remplacée** par des règles granulaires en configuration finale. Chaque flux légitime = 1 règle explicite.

---

## 🔴 Problème 11 — Rate-limiting pfSense après scans nmap répétés

### Symptôme
Un premier `nmap -p 80,443 192.168.10.1` affichait les ports comme `open`. Un second scan 30 secondes plus tard affichait les mêmes ports comme `filtered`.

### Cause
pfSense a un **mécanisme automatique anti-scan** : après plusieurs connexions rapides depuis une même IP, il **met cette IP en quarantaine temporaire** (5 minutes par défaut). Les ports apparaissent alors comme "filtered" côté scanner.

### Solution appliquée
- **Attendre 5 minutes** pour que la quarantaine expire
- **Scanner plus lentement** avec `nmap -T2` (polite) ou `nmap -T1` (sneaky)
- En pentest réel : **toujours espacer les tests** pour éviter la détection

### Apprentissage
**Scanner trop vite = se faire bannir.** En vraie situation de pentest :
- `-T0` (paranoid) : 5 min entre probes
- `-T1` (sneaky) : 15 secondes entre probes
- `-T2` (polite) : 0.4 secondes entre probes
- `-T3` (normal) : défaut, détectable par firewall moderne
- `-T4` (aggressive) : rapide, très détectable
- `-T5` (insane) : ultra-rapide, systématiquement détecté

Les pros utilisent **T2-T3 max**, jamais T4-T5.

---

## 🔴 Problème 12 — Login pfSense refusé après changement de mdp

### Symptôme
Après avoir changé le mot de passe admin de `pfsense` à `adminkalired`, impossible de se reconnecter. Message "Username or Password incorrect".

### Cause probable
**Erreur de frappe invisible** dans un champ password (qui masque ce qu'on tape). Un caractère mal tapé = mdp différent de ce qu'on croit.

### Solution appliquée
1. **Console pfSense → option 3** → `y` → Enter pour reset le mdp à `pfsense` (default)
2. **Firefox → login** avec `admin` / `pfsense`
3. Dans System → User Manager → edit admin
4. **Copier-coller** du nouveau mot de passe depuis un éditeur de texte (pour éviter toute erreur)
5. Save → test déconnexion/reconnexion

### Apprentissage
- **Toujours tester un nouveau mdp immédiatement après changement** (logout + login)
- **Copier-coller depuis un éditeur** = zéro erreur de frappe
- Les équipements pros ont **toujours une procédure de reset mdp** via console → fonctionnalité volontaire, pas une faille

---

## 🔴 Problème 13 — Conflit IP Windows ↔ pfSense sur VMnet3/5 _(Session #4)_

### Symptôme
Kali ne pouvait pas pinger pfSense sur son IP OPT (192.168.20.1 pour SERVERS, 192.168.99.1 pour RED). Pourtant les interfaces étaient bien configurées côté pfSense.

### Diagnostic avec tcpdump
```bash
sudo tcpdump -i eth1 arp
```
→ Les réponses ARP venaient d'une **adresse MAC Windows** (`00:50:56:XX:XX:XX`) au lieu de l'adresse MAC de pfSense (`00:0c:29:XX:XX:XX`).

### Cause
Windows avait aussi configuré son adaptateur VMnet avec l'IP `.1` du subnet (par défaut VMware attribue `.1` à l'host). **Deux machines sur le même réseau avec la même IP** = conflit d'IP classique.

### Solution appliquée
Dans **Virtual Network Editor** pour CHAQUE VMnet (Clients, Servers, Blue, Red) :
- **Décocher** "Connect a host virtual adapter to this network"
- Apply

Vérification : `ipconfig` sur Windows ne doit plus afficher ces VMnets.

### Apprentissage
**Les conflits d'IP sont silencieux et destructeurs.** `tcpdump` sur le trafic ARP permet d'identifier **qui répond réellement** à une IP. Si deux machines répondent → conflit à résoudre.

Dans un lab où pfSense fait office de gateway sur chaque VLAN, **l'adaptateur host Windows n'a aucune utilité** sur ces VMnets → le désactiver pour éviter tout conflit.

---

## 🔴 Problème 14 — Firefox vers pfSense OK mais ping bloqué _(Session #5)_

### Symptôme
Depuis Kali, l'accès web à pfSense (`https://192.168.99.1`) fonctionnait parfaitement, mais `ping -c 4 192.168.99.1` → 100% packet loss.

### Diagnostic
Le trafic TCP sur port 443 passait, mais le trafic ICMP était bloqué. Ce n'est donc pas un problème de connectivité réseau, mais de **règle firewall spécifique**.

### Cause
Après avoir remplacé la règle "allow all" (Problème #10) par une règle plus restrictive **"Allow RED to pfSense HTTPS (admin)"** (uniquement port 443 TCP), tout autre protocole (ICMP, UDP, autres TCP) était bloqué par la règle implicite "deny all" de pfSense.

**C'est le principe "deny by default" en action** : seul ce qui est explicitement autorisé passe.

### Solution appliquée
Création de **2 règles supplémentaires** sur l'interface RED :

**Règle "Allow RED ICMP (ping)"** :
| Champ | Valeur |
|---|---|
| Action | Pass |
| Interface | RED |
| Protocol | ICMP |
| ICMP Subtypes | any |
| Source | RED subnets |
| Destination | any |

**Règle "Allow RED to Servers (pentest)"** :
| Champ | Valeur |
|---|---|
| Action | Pass |
| Interface | RED |
| Protocol | any |
| Source | RED subnets |
| Destination | Network 192.168.20.0/24 |

**⚠️ Ne jamais oublier le "Apply Changes"** après avoir sauvé une règle (bandeau vert en haut).

### Apprentissage
- **Un firewall stateful = chaque flux non autorisé explicitement est bloqué**
- **Un service qui marche (HTTPS) ne garantit pas que les autres passent** (ICMP, UDP, etc.)
- **Les logs firewall** (Status → System Logs → Firewall) montrent les paquets bloqués en temps réel = outil de debug essentiel
- **Diagnostics → Ping** depuis pfSense avec `Source address` custom = permet de tester depuis une interface pfSense spécifique, outil pro

---

## 🔴 Problème 15 — Kali ping pfSense OK mais pas Metasploitable _(Session #5)_

### Symptôme
Depuis Kali :
- `ping -c 4 192.168.99.1` (pfSense côté RED) → ✅ 4/4 received
- `ping -c 4 192.168.20.100` (Metasploitable) → ❌ 0/4 received, 100% packet loss

Depuis Metasploitable :
- `ping -c 4 192.168.20.1` (pfSense côté SERVERS) → ✅ 4/4 received

### Diagnostic en 4 étapes

**Étape 1** — Vérification firewall pfSense : règle "Allow RED to Servers" bien active.

**Étape 2** — Test crucial `Diagnostics → Ping` depuis pfSense avec `Source address: RED` et `Hostname: 192.168.20.100` :
```
PING 192.168.20.100 from 192.168.99.1: 4/4 received ✅
```
→ pfSense PEUT atteindre Meta depuis son interface RED. **Donc le routage et le firewall fonctionnent côté pfSense.**

**Étape 3** — `tcpdump -i eth0 icmp` sur Metasploitable pendant un ping depuis Kali :
```
listening on eth0, link-type EN10MB (Ethernet), capture size 96 bytes
<RIEN n'est capturé>
```
→ **AUCUN paquet n'arrive sur Meta.** Le paquet se perd AVANT d'atteindre pfSense.

**Étape 4** — Analyse de la table de routage Kali :
```bash
$ ip route show
default via 192.168.56.2  dev eth0 proto dhcp metric 100  ← NAT
default via 192.168.99.1  dev eth1 proto dhcp metric 101  ← Red
192.168.56.0/24 dev eth0 proto kernel scope link
192.168.99.0/24 dev eth1 proto kernel scope link
```

### Cause racine

Kali avait **DEUX default gateways** :
- Via eth0 (NAT) avec **metric 100**
- Via eth1 (Red) avec **metric 101**

**Linux priorise la route avec la metric la plus basse**. Donc tout trafic vers un réseau inconnu (ex: 192.168.20.0/24) partait via **eth0 vers le NAT** au lieu de pfSense Red. Les paquets se perdaient dans la nature.

### Solution appliquée

Ajout d'une **route statique** sur Kali pour forcer le trafic 192.168.20.0/24 à passer par eth1 :

```bash
sudo ip route add 192.168.20.0/24 via 192.168.99.1 dev eth1
```

### Vérification

```bash
$ ip route show
192.168.20.0/24 via 192.168.99.1 dev eth1     ← Nouvelle route ★
default via 192.168.56.2 dev eth0 metric 100
default via 192.168.99.1 dev eth1 metric 101
...

$ ping -c 4 192.168.20.100
64 bytes from 192.168.20.100: icmp_seq=1 ttl=63 time=1.21 ms
...
4 packets transmitted, 4 received, 0% packet loss ✅
```

**TTL=63** (au lieu de 64) → preuve mathématique que le paquet a bien traversé 1 routeur (pfSense).

### ⚠️ Attention : persistance

La route statique ajoutée est **en RAM uniquement**. Elle disparaît au reboot.

Pour la rendre persistante sur Kali (à faire en session #6) :

**Option A — NetworkManager** :
```bash
nmcli connection modify Red-Lab +ipv4.routes "192.168.20.0/24 192.168.99.1"
```

**Option B — /etc/network/interfaces** :
```
iface eth1 inet static
    address 192.168.99.101
    netmask 255.255.255.0
    up route add -net 192.168.20.0/24 gw 192.168.99.1 dev eth1
```

De même pour l'IP statique de Metasploitable, à ajouter dans `/etc/network/interfaces` :
```
auto eth0
iface eth0 inet static
    address 192.168.20.100
    netmask 255.255.255.0
    gateway 192.168.20.1
```

### Apprentissage

- **Toujours vérifier la table de routage** avant de supposer un problème firewall
- **`tcpdump` = premier réflexe** quand "rien ne marche" : voir si le paquet arrive vraiment
- **`Diagnostics → Ping` de pfSense avec source custom** = outil n°1 pour tester les règles depuis l'extérieur
- **TTL décrémenté** = preuve de traversée d'un routeur (compteur de hops)
- **Metric Linux** : plus petit = plus prioritaire (contre-intuitif pour un débutant)
- **Firewall stateful** : les retours sont autorisés automatiquement si la connexion a été initiée

### Schéma du problème et solution

#### AVANT (bug)
```
                    Table de routage Kali:
                    - default via NAT (metric 100) ← Linux prend celle-ci
                    - default via Red (metric 101)

KALI ────ping 192.168.20.100────▶ eth0 ────▶ NAT ──X── Perdu
                                  (mauvaise interface)

                                  eth1 (Red) ← Jamais utilisée
```

#### APRÈS (fix)
```
                    Table de routage Kali:
                    - 192.168.20.0/24 via 192.168.99.1 dev eth1 ★ (nouvelle)
                    - default via NAT (metric 100)
                    - default via Red (metric 101)

KALI ────ping 192.168.20.100────▶ eth1 ────▶ pfSense ────▶ META
                                  (bonne interface)   (routage inter-VLAN)   ✅
```

---

## 🎯 Récapitulatif des bonnes pratiques apprises

### Côté outils
- ✅ VMware **toujours en admin**
- ✅ Notepad **jamais** pour les fichiers sans extension (préférer VSCode / PowerShell)
- ✅ Activer "Afficher les extensions de fichier" dans Windows
- ✅ SSH + `nmcli` au lieu de console + `dhclient` sur Kali moderne

### Côté réseau
- ✅ Toujours **nommer les VMnets** (Clients, Servers, Blue, Red) au lieu de numéros
- ✅ **Créer des connexions NetworkManager dédiées** par interface
- ✅ **Désactiver les host adapters** sur les VMnets où pfSense est gateway (évite conflit IP)
- ✅ `ip addr`, `ip route`, `ip neigh` = trio de base pour diagnostiquer
- ✅ **Toujours vérifier la table de routage** avant de blâmer un firewall
- ✅ **Metric Linux** : plus petit = plus prioritaire

### Côté firewall
- ✅ **"Deny by default, allow by exception"** = principe
- ✅ **Toujours documenter chaque règle** (description claire)
- ✅ **Règles granulaires** (par protocole, par destination) > règle "allow all"
- ✅ **Vérifier les logs firewall** (Status → System Logs → Firewall) en live pour debug
- ✅ **Diagnostics → Ping** avec source custom = tester le routage depuis une interface spécifique
- ✅ **Scanner lentement** (`-T2`) pour éviter la détection
- ✅ **Firewall stateful** : les retours sont automatiquement autorisés

### Côté sécurité
- ✅ **Changer les mots de passe par défaut** immédiatement
- ✅ **Tester après chaque changement** (logout/login)
- ✅ **Copier-coller** les mdp depuis un éditeur pour éviter les typos
- ✅ **Documenter les credentials** dans un fichier ignoré par Git (`.gitignore`)

### Côté debug
- ✅ **tcpdump = outil n°1** pour voir si un paquet arrive vraiment
- ✅ **Dichotomie du debug** : tester chaque segment indépendamment (A→Router, Router→B, puis A→B)
- ✅ **TTL dans ping** : décrémente de 1 à chaque routeur = compteur de hops
- ✅ **Si tcpdump ne voit RIEN** = problème en amont (source), pas en destination

### Côté méthode
- ✅ **Lire les messages d'erreur** avant de paniquer (90% des solutions sont dans le message)
- ✅ **Utiliser `-v` ou `--verbose`** pour debug en profondeur
- ✅ **Un changement à la fois**, tester, documenter
- ✅ **Snapshot VMware** avant toute modif importante
- ✅ **Un service qui marche ≠ tous les services passent** (HTTPS OK ≠ ping OK)
