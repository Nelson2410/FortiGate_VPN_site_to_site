# 🔐 FortiGate VPN Site-to-Site Lab

[![EVE-NG](https://img.shields.io/badge/Platform-EVE--NG-blue.svg)](https://www.eve-ng.net/)
[![FortiGate](https://img.shields.io/badge/FortiGate-v7-red.svg)](https://www.fortinet.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> Laboratoire réseau professionnel sous EVE-NG simulant une interconnexion sécurisée entre deux sites distants via VPN IPsec site-to-site avec FortiGate.

---

## Vue d'ensemble

Ce projet implémente une architecture réseau d'entreprise complète avec :
- **2 sites géographiquement distants** (LAN A & LAN B)
- **6 VLANs segmentés** pour différents services métier
- **Tunnel VPN IPsec** chiffré entre les sites (Mode Route-based)
- **Routage inter-VLAN** et **inter-site** transparent
- **Services DHCP** automatisés par VLAN et sécurisés (Snooping, DAI)

## Architecture et Segmentation

![Topologie réseau](1%20-%20Docs/Topologie.png)

### **Site A** (FortiGate-A & Switch-A)

| VLAN | Nom          | Réseau           | Passerelle (FortiGate) | Rôle              |
|------|--------------|------------------|------------------------|-------------------|
| 10   | NetAdmin     | 10.10.10.0/24    | 10.10.10.254           | Administration    |
| 20   | SysAdmin     | 10.10.20.0/24    | 10.10.20.254           | Systèmes          |
| 30   | NetDev       | 10.10.30.0/24    | 10.10.30.254           | Développement     |

### **Site B** (FortiGate-B & Switch-B)

| VLAN | Nom          | Réseau           | Passerelle (FortiGate) | Rôle              |
|------|--------------|------------------|------------------------|-------------------|
| 40   | Testeurs     | 10.10.40.0/24    | 10.10.40.254           | QA & Tests        |
| 50   | Managers     | 10.10.50.0/24    | 10.10.50.254           | Direction         |
| 60   | Contrib      | 10.10.60.0/24    | 10.10.60.254           | Contributeurs     |

---
## Fonctionnalités clés

### Sécurité
- Tunnel VPN IPsec avec chiffrement AES-256
- Zones de sécurité (ZONE_LAN_A, ZONE_LAN_B, ZONE_VPN, ZONE_WAN)
- Firewall policies granulaires par direction de trafic
- DHCP Snooping + Dynamic ARP Inspection sur les switches
- Port Security avec MAC sticky

### Routage
- Routage inter-VLAN assuré par les FortiGate
- Routage inter-site via tunnel VPN (sans NAT)
- Route par défaut vers Internet avec NAT

### Services
- DHCP automatique par VLAN (plages .100-.200)
- DNS configuré via DHCP
- Spanning Tree avec PortFast et BPDU Guard
---

## Stack technologique

### Plateforme
- **EVE-NG** : Environnement de virtualisation réseau
- **Hyperviseur** : VMware / Proxmox (selon infrastructure)

### Équipements
- **Pare-feu** : 2× FortiGate v7
- **Switches** : 2× vIOS-L2 (Cisco IOS)
- **Hôtes** : VPCS / VM Linux

### Protocoles & technologies
- **VPN** : IPsec (IKEv2)
  - Phase 1 : AES-256 + SHA-256/SHA-1, DH Group 14
  - Phase 2 : AES-256, Perfect Forward Secrecy
  - PSK : `FortiVPN@2026`
- **Routage** : Routes statiques + routage inter-VLAN
- **Sécurité** : Zones de sécurité, firewall policies, NAT
- **Services** : DHCP par VLAN, DHCP Snooping, DAI, Port Security
---

## Tests de validation

### Tests de connectivité

```bash
# Depuis un hôte VLAN 10 (Site A)
ping 10.10.20.100    # Test inter-VLAN local
ping 10.10.40.100    # Test inter-site via VPN
ping 8.8.8.8         # Test sortie Internet

# Vérifier le tunnel VPN
# Sur FortiGate-A/B
diagnose vpn ike gateway list
diagnose vpn tunnel list
get router info routing-table all
```

### Monitoring

```bash
# Logs VPN
diagnose debug application ike -1
diagnose debug enable

# Traffic firewall
diagnose sniffer packet any 'host 10.10.10.100' 4
```
---
## Preuves de réussite (Validation du Lab)

### 1. État du Tunnel VPN IPsec
![Statut VPN](1%20-%20Docs/Capture-1.png)
*Le tunnel IPsec est établi (UP) avec une négociation réussie des phases 1 et 2, confirmée par l'échange de paquets chiffrés (TX/RX).*

### 2. Connectivité Inter-Site (Ping)
![Ping Inter-Site](1%20-%20Docs/Capture-2.png)
*Test de ping réussi entre le PC du Site A (VLAN 10) et le PC du Site B (VLAN 40) à travers le tunnel sécurisé.*

### 3. Accès Internet et Sortie WAN
![Sortie Internet](1%20-%20Docs/Capture-3.png)
*Validation de la sortie Internet et de la résolution DNS depuis les différents VLANs du LAN.*

### 4. Attribution DHCP et Services locaux
![DHCP Leases](1%20-%20Docs/Capture-4.png)
*Confirmation de l'attribution dynamique des adresses IP par le FortiGate et du bon fonctionnement du routage inter-VLAN.*

---

## Guide de Déploiement

Pour garantir le succès du laboratoire, les fichiers du dossier `2 - Configs/` respectent une chronologie stricte évitant tout rejet de commande :

1. **Switchs Cisco** : Configuration des VLANs, puis des Trunks de confiance (`trust`), et enfin activation globale du DHCP Snooping/DAI.
2. **FortiGate** : Création des interfaces et DHCP en premier, suivi de la Phase 1 IPsec (pour générer l'interface virtuelle), puis du routage et des politiques de sécurité.

---

## 📂 Structure du Répertoire

```text
.
├── 1 - Docs/
│   ├── Doc_Technique.pdf     # Documentation détaillée
│   ├── Topologie.png         # Schéma réseau
│   ├── Capture-1.png         # Preuve : Statut VPN
│   ├── Capture-2.png         # Preuve : Connectivité Inter-site
│   ├── Capture-3.png         # Preuve : Sortie Internet
│   └── Capture-4.png         # Preuve : Services DHCP/Routage
├── 2 - Configs/
│   ├── Fortigate-A.conf      # Config optimisée (Passerelle VMware & Crypto)
│   ├── Fortigate-B.conf      # Config optimisée (Passerelle VMware & Crypto)
│   ├── Switch-A.conf         # Config L2 sécurisée
│   └── Switch-B.conf         # Config L2 sécurisée
├── LICENSE                   # Licence MIT
└── README.md                 # Ce fichier
```
---

## Auteur

**Nelson Bandos**  
📧 nelson.bandos@proton.me  
🔗 [GitHub](https://github.com/Nelson2410)

---
