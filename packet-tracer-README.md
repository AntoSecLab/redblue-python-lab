# 🎓 Lab Packet Tracer — Architecture VLANs + ACL

> Lab Cisco Packet Tracer parallèle au projet principal `redblue-python-lab`.
>
> **Objectif pédagogique** : comprendre les concepts réseau (VLANs, routing, firewall) côté Cisco pour approfondir la compréhension théorique de l'infrastructure VMware réelle.

---

## 🎯 Pourquoi un lab Packet Tracer en parallèle ?

La vraie infrastructure du projet tourne sur **VMware avec pfSense**. Mais pfSense **masque** la complexité réseau sous une interface web. En reconstruisant la même architecture dans Packet Tracer avec des équipements **Cisco** (routeur 1941 + switch 2960), j'apprends :

- Les **commandes Cisco IOS** (standard du marché, utilisé dans 80% des infrastructures réseau)
- Les concepts **VLANs 802.1Q** et **router-on-a-stick** en profondeur
- Le **Mode Simulation** de PT pour **voir les paquets voyager** en temps réel
- La **syntaxe ACL Cisco** (équivalent des règles firewall pfSense)
- Les fondamentaux nécessaires pour la **certification CCNA**

---

## 📐 Architecture du lab actuel (v1 - interne uniquement)

```
                 Router 1941 (PFSENSE)
                  │ Gig0/0 (trunk 802.1Q)
                  │ 4 sous-interfaces : .10 .20 .30 .99
                  │
               Switch 2960 (Switch0)
              /    |    |    \
           Fa0/2 Fa0/3 Fa0/4 Fa0/5
             │    │    │    │
            PC   PC   PC   PC
           V10  V20  V30  V99
          CLIENT SRV BLUE RED
         (DHCP) (DHCP)(DHCP)(DHCP)
```

### Plan d'adressage

| VLAN | Nom | Subnet | Rôle | DHCP |
|---|---|---|---|---|
| 10 | CLIENTS | 192.168.10.0/24 | Postes utilisateurs | ✅ |
| 20 | SERVERS | 192.168.20.0/24 | Serveurs internes | ✅ (ici) |
| 30 | BLUE | 192.168.30.0/24 | Blue Team / SOC | ✅ |
| 99 | RED | 192.168.99.0/24 | Red Team / attaquant | ✅ |

### IP des gateways (sous-interfaces routeur)

| Interface | IP | VLAN | Rôle |
|---|---|---|---|
| G0/0.10 | 192.168.10.1 | 10 | Gateway Clients |
| G0/0.20 | 192.168.20.1 | 20 | Gateway Servers |
| G0/0.30 | 192.168.30.1 | 30 | Gateway Blue |
| G0/0.99 | 192.168.99.1 | 99 | Gateway Red |

### Plages DHCP

Pour chaque VLAN :
- `.1 à .99` → **exclus** du DHCP (réservé statique pour serveurs, gateways)
- `.100 à .254` → **distribué** par DHCP (postes clients dynamiques)

---

## 🔧 Configurations clés documentées

### 1️⃣ Sécurisation des devices

Appliqué sur le routeur ET le switch :

```cisco
enable
configure terminal
hostname <NOM>

! Mot de passe mode privilégié
enable secret <mdp>

! Protection console physique
line console 0
 password <mdp>
 login
 exit

! Protection accès distant SSH/Telnet
line vty 0 4
 password <mdp>
 login
 exit

! Chiffrement de tous les mdp en clair
service password-encryption

end
write memory
```

### 2️⃣ Création des VLANs (sur le switch)

```cisco
vlan 10
 name CLIENTS
vlan 20
 name SERVERS
vlan 30
 name BLUE
vlan 99
 name RED
```

### 3️⃣ Assignation des ports en mode access (sur le switch)

Chaque PC est branché sur un port assigné à son VLAN :

```cisco
interface fa0/2
 switchport mode access
 switchport access vlan 10
 no shutdown
 exit

interface fa0/3
 switchport mode access
 switchport access vlan 20
 no shutdown
 exit

interface fa0/4
 switchport mode access
 switchport access vlan 30
 no shutdown
 exit

interface fa0/5
 switchport mode access
 switchport access vlan 99
 no shutdown
 exit
```

### 4️⃣ Trunk 802.1Q vers le routeur (sur le switch)

Le port Fa0/1 transporte **les 4 VLANs** vers le routeur via un seul câble :

```cisco
interface fa0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,99
 no shutdown
 exit
```

### 5️⃣ Router-on-a-stick (sur le routeur)

Création de **4 sous-interfaces virtuelles** sur Gig0/0, chacune dédiée à un VLAN :

```cisco
interface gigabitEthernet 0/0
 no shutdown
 exit

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit

interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 exit

interface gigabitEthernet 0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
 exit
```

### 6️⃣ DHCP serveur sur le routeur

```cisco
! Exclusion des plages statiques (1-99)
ip dhcp excluded-address 192.168.10.1 192.168.10.99
ip dhcp excluded-address 192.168.20.1 192.168.20.99
ip dhcp excluded-address 192.168.30.1 192.168.30.99
ip dhcp excluded-address 192.168.99.1 192.168.99.99

! Pool pour VLAN Clients
ip dhcp pool POOL_CLIENTS
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 exit

! Pool pour VLAN Servers
ip dhcp pool POOL_SERVERS
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
 exit

! Pool pour VLAN Blue
ip dhcp pool POOL_BLUE
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8
 exit

! Pool pour VLAN Red
ip dhcp pool POOL_RED
 network 192.168.99.0 255.255.255.0
 default-router 192.168.99.1
 dns-server 8.8.8.8
 exit
```

### 7️⃣ ACL : bloquer RED → SERVERS

Simulation réaliste : un attaquant depuis Red ne doit **pas** pouvoir atteindre les serveurs sensibles.

```cisco
access-list 150 deny ip 192.168.99.0 0.0.0.255 192.168.20.0 0.0.0.255
access-list 150 permit ip any any

interface gigabitEthernet 0/0.99
 ip access-group 150 in
 exit
```

**Explication** :
- Ligne 1 : bloque tout trafic IP venant du VLAN Red vers le VLAN Servers
- Ligne 2 : autorise tout le reste (sinon `deny` implicite en fin d'ACL = tout bloqué)
- Application : à l'**entrée** (`in`) de la sous-interface Red = filtrage le plus tôt possible

---

## 🧪 Tests validés

| Test | Résultat attendu | Résultat obtenu |
|---|---|---|
| Ping Client → Server | OK | ✅ `Received = 4` |
| Ping Client → Blue | OK | ✅ `Received = 4` |
| Ping Client → Red | OK | ✅ `Received = 4` |
| Ping Red → Client | OK (permit) | ✅ `Received = 4` |
| Ping Red → Blue | OK (permit) | ✅ `Received = 4` |
| **Ping Red → Server** | **BLOQUÉ (deny)** | ✅ **`Destination host unreachable` + `Received = 0`** |
| Distribution DHCP 4 PCs | Chaque PC reçoit son IP auto | ✅ Validé par `show ip dhcp binding` |

### Preuve du fonctionnement de l'ACL

```
Router# show access-lists
Extended IP access list 150
    10 deny ip 192.168.99.0 0.0.0.255 192.168.20.0 0.0.0.255 (12 match(es))
    20 permit ip any any (8 match(es))
```

Le compteur `(12 match(es))` sur le `deny` prouve que **12 paquets Red→Server ont été bloqués** par cette règle.

### Preuve DHCP

```
Router# show ip dhcp binding
IP address       Client-ID (MAC)      Lease expiration    Type
192.168.10.100   0050.0F0B.5840       --                  Automatic
192.168.20.100   0001.4216.0E14       --                  Automatic
192.168.30.100   0001.C7A2.72A6       --                  Automatic
192.168.99.100   0060.4783.CB53       --                  Automatic
```

---

## 🎓 Concepts CCNA couverts dans ce lab

- ✅ **Topologie réseau** (routeur, switch, PCs, câblage)
- ✅ **Sécurisation des devices** (enable secret, console, VTY, service password-encryption)
- ✅ **VLANs** (création, nommage, assignation ports)
- ✅ **Trunk 802.1Q** (switchport mode trunk, allowed vlan)
- ✅ **Router-on-a-stick** (sous-interfaces, encapsulation dot1Q)
- ✅ **DHCP server** (pool, excluded-address, default-router, dns-server)
- ✅ **ACL Extended** (deny/permit avec wildcard masks)
- ✅ **Application d'ACL sur interface** (ip access-group in/out)
- ✅ **Vérification** (show vlan brief, show interface trunk, show ip interface brief, show ip dhcp binding, show access-lists)

---

## 🔗 Lien avec l'infra VMware réelle

Ce lab PT **reproduit la logique** de l'infra VMware pfSense, mais les 2 environnements sont **totalement indépendants** :

| Côté | VMware (réel) | Packet Tracer (simulé) |
|---|---|---|
| **Routeur** | pfSense (FreeBSD) | Cisco 1941 |
| **Switch** | VMnets (invisibles) | Cisco 2960 |
| **Attaques Python** | ✅ possibles | ❌ non |
| **Visualisation paquets** | Wireshark | Mode Simulation PT |
| **Syntaxe config** | GUI Web pfSense | CLI Cisco IOS |

**Le combo** = **le meilleur des 2 mondes** :
- VMware pour **faire** (vraies attaques, scripts Python)
- Packet Tracer pour **comprendre** (voir les paquets, apprendre Cisco IOS)

---

## 🚀 Évolutions prévues (v2 et au-delà)

### 🔴 v2 — Ajout d'un "Red externe" (phase future)

Idée : simuler un attaquant **vraiment externe** (comme un hacker depuis Internet qui essaie de pénétrer l'entreprise), en complément du Red interne actuel qui simule un **insider threat** (employé compromis).

**Topologie prévue** :
```
[Internet Cloud] → [Kali Externe] → [FW pfSense/Router WAN] → [réseau interne]
```

Cela permettra de simuler les 2 scénarios d'attaque les plus courants :
- **Type A - Attaquant externe** : perce les défenses périphériques (firewall, NAT, exposed services)
- **Type B - Attaquant interne** : déjà dans le réseau (phishing, insider, supply chain)

### 🛡️ v3 — Durcissement progressif

- ACL plus fines (bloquer SMB, autoriser SSH uniquement)
- VLAN hopping protection (storm control, port security)
- Dynamic ARP Inspection (anti-spoofing)
- DHCP snooping

### 🎯 v4 — Monitoring Blue Team

- Configuration SNMP sur routeur/switch
- NetFlow pour analyse de trafic
- Syslog vers un serveur centralisé (simulant Wazuh)

---

## 📁 Fichiers du dossier

- `lab-v1-vlans.pkt` — fichier Packet Tracer du lab
- `README.md` — ce fichier
- `screenshots/` — captures d'écran des tests validés (à ajouter)

---

## 💡 Comment ouvrir et explorer ce lab

1. Installer **Cisco Packet Tracer** (v8.0+ recommandé, v9.0 utilisé pour ce lab) depuis [netacad.com](https://www.netacad.com)
2. Ouvrir le fichier `lab-v1-vlans.pkt`
3. **Explorer** : double-clic sur chaque device → onglets Physical, Config, CLI
4. **Tester** : double-clic sur un PC → Desktop → Command Prompt → `ping X`
5. **Voir les paquets** : bouton **Simulation** (chronomètre) en bas à droite, Edit Filters pour cocher ICMP+ARP, puis Play

---

## 🎓 Pour continuer sur ce chemin (CCNA)

Les concepts de ce lab sont au programme du **CCNA 200-301** (certification Cisco). Pour aller plus loin :
- Netacad gratuit : [Introduction to Networks](https://www.netacad.com/courses/networking)
- Jeremy's IT Lab (YouTube) : excellente chaîne gratuite pour CCNA
- Lab suivants à construire : OSPF, VLAN hopping, EtherChannel, HSRP

---

*Lab réalisé dans le cadre du projet `redblue-python-lab` — Anthony Guarin*
