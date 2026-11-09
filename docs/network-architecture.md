# 🌐 Architecture réseau du lab

## Vue d'ensemble

Le lab simule une petite infrastructure d'entreprise avec 4 zones réseau isolées, connectées via un routeur/pare-feu pfSense central.

## Mapping VMnets → Subnets

| Nom VMware | Subnet | Rôle | VMs prévues |
|---|---|---|---|
| `client` | 192.168.10.0/24 | Postes utilisateurs (VLAN Clients) | Windows 10 |
| `servers` | 192.168.20.0/24 | Serveurs internes (VLAN Servers) | Windows Server 2022, Ubuntu Server, Metasploitable 2 |
| `blue` | 192.168.30.0/24 | Outils défensifs (VLAN Blue team / SOC) | Wazuh |
| `red` | 192.168.99.0/24 | Attaquant (VLAN Red team) | Kali Linux |
| `VMnet8` (NAT) | 192.168.56.0/24 | Sortie internet (via NAT) | Kali (eth0), pfSense (WAN), Wazuh (eth0 pour updates) |

## Plan d'attribution des IPs statiques

À faire en Phase 1 lors de la configuration de pfSense.

| VM | Interface | Réseau | IP prévue |
|---|---|---|---|
| pfSense | WAN | NAT (VMnet8) | DHCP auto |
| pfSense | LAN clients | client | 192.168.10.1 |
| pfSense | LAN servers | servers | 192.168.20.1 |
| pfSense | LAN blue | blue | 192.168.30.1 |
| pfSense | LAN red | red | 192.168.99.1 |
| Kali | eth0 | NAT | DHCP |
| Kali | eth1 | red | 192.168.99.10 |
| Metasploitable | eth0 | servers | 192.168.20.100 |
| Windows Server 2022 | eth0 | servers | 192.168.20.10 |
| Windows 10 | eth0 | client | 192.168.10.10 (via DHCP pfSense) |
| Wazuh | eth0 | blue | 192.168.30.10 |
| Wazuh | eth1 | NAT | DHCP (pour updates) |

## Règles de sécurité importantes

- ⚠️ **Metasploitable 2 n'a JAMAIS d'accès internet** — elle vit uniquement sur `servers` pour être attaquée depuis Kali via pfSense
- ⚠️ **VMware doit être lancé en administrateur** pour que les modifs réseau soient sauvegardées
- ⚠️ **DHCP VMware désactivé** sur tous les VMnets du lab (c'est pfSense qui fera le DHCP)

## Justification pédagogique

Cette segmentation reproduit le pattern "défense en profondeur" utilisé en entreprise :
- Un attaquant sur le VLAN `red` ne peut PAS accéder directement aux VMs des autres VLANs → tout passe par pfSense
- Les règles firewall de pfSense simulent la politique de sécurité d'une DSI (Direction des Systèmes d'Information)
- Le SIEM Wazuh sur `blue` observe le trafic et détecte les tentatives d'intrusion