# 🔐 FortiGate VPN Site-to-Site Lab

[![EVE-NG](https://img.shields.io/badge/Platform-EVE--NG-blue.svg)](https://www.eve-ng.net/)
[![FortiGate](https://img.shields.io/badge/FortiGate-v7-red.svg)](https://www.fortinet.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> Laboratoire réseau professionnel sous EVE-NG simulant une interconnexion sécurisée entre deux sites distants via VPN IPsec site-to-site avec FortiGate.

---

## 📋 Vue d'ensemble

Ce projet implémente une architecture réseau d'entreprise complète avec :
- **2 sites géographiquement distants** (LAN A & LAN B)
- **6 VLANs segmentés** pour différents services métier
- **Tunnel VPN IPsec** chiffré entre les sites
- **Routage inter-VLAN** et **inter-site** transparent
- **Services DHCP** automatisés par VLAN

### 🎯 Objectifs

- Déployer une infrastructure réseau multi-sites sécurisée
- Maîtriser les concepts FortiGate (firewall policies, zones, VPN, DHCP)
- Simuler un environnement d'entreprise réaliste pour la formation et les tests

---

## 🏗️ Architecture

![Topologie réseau](1%20-%20Docs/Topologie.png)

### Schéma simplifié

```
                          ┌─────────────┐
                          │  Internet   │
                          │ 192.168.200 │
                          └──────┬──────┘
                    ┌────────────┴────────────┐
                    │                         │
              .20   │                         │   .30
         ┌──────────▼────────┐       ┌───────▼──────────┐
         │   FortiGate-A     │       │   FortiGate-B    │
         │   (Site A)        │◄─────►│   (Site B)       │
         └─────────┬─────────┘  VPN  └────────┬─────────┘
                   │ port2                     │ port2
              TRUNK│ (VLAN)               TRUNK│ (VLAN)
         ┌─────────▼─────────┐       ┌────────▼─────────┐
         │    Switch-A       │       │    Switch-B      │
         └─┬────────┬────┬───┘       └───┬───────┬───┬──┘
           │        │    │               │       │   │
       ┌───▼──┐ ┌──▼──┐ ┌▼────┐      ┌──▼──┐ ┌──▼─┐ ┌▼────┐
       │VLAN10│ │VLAN │ │VLAN │      │VLAN │ │VLAN│ │VLAN │
       │Admin │ │  20 │ │ 30  │      │ 40  │ │ 50 │ │ 60  │
       └──────┘ └─────┘ └─────┘      └─────┘ └────┘ └─────┘
```

---

## 🌐 Segmentation réseau

### **Site A** (LAN A)

| VLAN | Nom          | Réseau           | Passerelle    | Rôle              |
|------|--------------|------------------|---------------|-------------------|
| 10   | NetAdmin     | 10.10.10.0/24    | 10.10.10.254  | Administrateurs réseau |
| 20   | SysAdmin     | 10.10.20.0/24    | 10.10.20.254  | Administrateurs système |
| 30   | NetDev       | 10.10.30.0/24    | 10.10.30.254  | Développeurs réseau |

### **Site B** (LAN B)

| VLAN | Nom          | Réseau           | Passerelle    | Rôle              |
|------|--------------|------------------|---------------|-------------------|
| 40   | Testeurs     | 10.10.40.0/24    | 10.10.40.254  | Équipe QA/Test    |
| 50   | Managers     | 10.10.50.0/24    | 10.10.50.254  | Management        |
| 60   | Contributors | 10.10.60.0/24    | 10.10.60.254  | Contributeurs     |

### **Interconnexion WAN**

| Équipement   | Interface | IP WAN          | Rôle          |
|--------------|-----------|-----------------|---------------|
| FortiGate-A  | port1     | 192.168.200.20  | Endpoint VPN A |
| FortiGate-B  | port1     | 192.168.200.30  | Endpoint VPN B |

---

## 🛠️ Stack technologique

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

## 🚀 Déploiement

### Prérequis

```bash
✓ EVE-NG installé et fonctionnel
✓ Images FortiGate v7 disponibles
✓ Images vIOS-L2 pour les switches
✓ Connaissance de base en réseaux IP et VLANs
```

### Installation

1. **Cloner le dépôt**
```bash
git clone https://github.com/Nelson2410/FortiGate_VPN_S2S.git
cd FortiGate_VPN_S2S
```

2. **Importer dans EVE-NG**
   - Créer un nouveau lab
   - Ajouter 2 FortiGate et 2 switches vIOS-L2
   - Connecter selon la topologie (voir diagramme)

3. **Appliquer les configurations**
```bash
# Sur FortiGate-A
config restore config flash 2-Configs/Fortigate-A.conf

# Sur FortiGate-B
config restore config flash 2-Configs/Fortigate-B.conf

# Sur les switches (via CLI)
copy tftp running-config
# ou copier-coller depuis 2-Configs/Switch-A.conf et Switch-B.conf
```

4. **Ajouter les hôtes de test**
   - Connecter des VPCS sur chaque VLAN
   - Configurer en DHCP : `ip dhcp`

---

## ✅ Validation

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

## 📊 Fonctionnalités clés

### 🔒 Sécurité
- Tunnel VPN IPsec avec chiffrement AES-256
- Zones de sécurité (ZONE_LAN_A, ZONE_LAN_B, ZONE_VPN, ZONE_WAN)
- Firewall policies granulaires par direction de trafic
- DHCP Snooping + Dynamic ARP Inspection sur les switches
- Port Security avec MAC sticky

### 🌍 Routage
- Routage inter-VLAN assuré par les FortiGate
- Routage inter-site via tunnel VPN (sans NAT)
- Route par défaut vers Internet avec NAT

### ⚙️ Services
- DHCP automatique par VLAN (plages .100-.200)
- DNS configuré via DHCP
- Spanning Tree avec PortFast et BPDU Guard

---

## 📁 Structure du projet

```
FortiGate_VPN_S2S/
├── 1 - Docs/
│   ├── Doc_Technique.pdf     # Documentation détaillée
│   └── Topologie.png         # Schéma d'architecture réseau
├── 2 - Configs/
│   ├── Fortigate-A.conf      # Configuration FortiGate Site A
│   ├── Fortigate-B.conf      # Configuration FortiGate Site B
│   ├── Switch-A.conf         # Configuration Switch Site A
│   └── Switch-B.conf         # Configuration Switch Site B
├── .gitignore                # Exclusions Git
├── LICENSE                   # Licence MIT
└── README.md                 # Ce fichier
```

---

## 🎓 Cas d'usage

- **Formation** : Apprentissage FortiGate et VPN IPsec
- **Lab de test** : Validation de configurations avant production
- **POC** : Démonstration d'architecture multi-sites
- **Préparation certification** : Entraînement NSE (Network Security Expert)

---

## 🐛 Dépannage

### Le tunnel VPN ne monte pas

```bash
# Vérifier la connectivité WAN
execute ping 192.168.200.30

# Vérifier les paramètres Phase 1/2
show vpn ipsec phase1-interface
show vpn ipsec phase2-interface

# Vérifier les policies
show firewall policy
```

### Pas de communication inter-VLAN

```bash
# Vérifier le routage
get router info routing-table all

# Vérifier les policies de zone
show firewall policy | grep ZONE_LAN
```

---

## 📚 Documentation

- [Documentation technique complète](1%20-%20Docs/Doc_Technique.pdf)
- [Topologie réseau](1%20-%20Docs/Topologie.png)
- [FortiGate Documentation](https://docs.fortinet.com/)
- [EVE-NG Cookbook](https://www.eve-ng.net/index.php/documentation/)

---

## 👤 Auteur

**Nelson Bandos**  
📧 nelson.bandos@proton.me  
🔗 [GitHub](https://github.com/Nelson2410)

---

## 📄 Licence

Ce projet est sous licence MIT. Voir le fichier [LICENSE](LICENSE) pour plus de détails.

---

## 🙏 Remerciements

- Fortinet pour FortiGate
- Communauté EVE-NG
- Tous les contributeurs réseau open source

---

**⭐ Si ce projet vous a été utile, n'hésitez pas à lui donner une étoile !**
