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

### Objectifs
- Déployer une infrastructure réseau multi-sites sécurisée.
- Maîtriser les concepts FortiGate (firewall policies, zones, VPN IPsec, DHCP).
- Simuler un environnement d'entreprise réaliste (Best Practices Cisco L2) pour la formation et les tests.

---

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

## Guide de Déploiement et Bonnes Pratiques

Pour garantir le succès du laboratoire et éviter les erreurs courantes de dépendance (rejet de commandes par les équipements ou blocage du trafic légitime), les fichiers du dossier `2 - Configs/` ont été structurés selon une **chronologie stricte**. 

Un copier-coller intégral des scripts fonctionnera du premier coup, car il respecte la logique d'initialisation suivante :

### 1. Séquence de configuration des Commutateurs Cisco
* **VLANs en premier :** Création de l'infrastructure logique de base.
* **Liaisons de confiance (Trunk) :** Le port Trunk vers le FortiGate est configuré avec l'encapsulation `dot1q` et déclaré comme fiable (`trust`) pour le DHCP Snooping et l'ARP Inspection.
* **Ports utilisateurs :** Configuration des accès avec `spanning-tree portfast` (vital pour ne pas court-circuiter les requêtes DHCP initiales) et le Port-Security.
* **Sécurité globale en dernier :** Activation du *DHCP Snooping* et du *Dynamic ARP Inspection (DAI)* uniquement à la fin. *Activer ces mécanismes avant de déclarer les ports de confiance bloquerait instantanément tout le trafic.*

### 2. Séquence de configuration des FortiGate
* **Fondations réseau :** Interfaces physiques, sous-interfaces VLAN (802.1Q) et activation immédiate des serveurs DHCP. Le pare-feu doit être prêt à répondre avant que les switchs n'activent leur inspection ARP.
* **Phase 1 IPsec :** La configuration de la Phase 1 est prioritaire car elle génère l'**interface virtuelle dynamique** du tunnel VPN (ex: `VPN_A_B`).
* **Zones & Routage :** Sans la Phase 1 préalablement créée, l'ajout de routes statiques vers le site distant ou la création de Zones de sécurité ciblant le VPN échouerait.
* **Firewall Policies :** Rédigées en toute fin de script, elles peuvent s'appuyer sereinement sur les Zones de sécurité (LAN, WAN, VPN) et les interfaces désormais existantes.

---

## 📂 Structure du Répertoire

```text
.
├── 1 - Docs/
│   ├── Doc_Technique.pdf     # Documentation détaillée du projet
│   └── Topologie.png         # Schéma de l'architecture
├── 2 - Configs/
│   ├── Fortigate-A.conf      # Configuration FortiGate Site A (Prêt à coller)
│   ├── Fortigate-B.conf      # Configuration FortiGate Site B (Prêt à coller)
│   ├── Switch-A.conf         # Configuration Switch Site A (Prêt à coller)
│   └── Switch-B.conf         # Configuration Switch Site B (Prêt à coller)
├── .gitignore                # Exclusions Git
├── LICENSE                   # Licence MIT
└── README.md                 # Ce fichier

```
---

## Dépannage

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

## Documentation

- [Documentation technique complète](1%20-%20Docs/Doc_Technique.pdf)
- [Topologie réseau](1%20-%20Docs/Topologie.png)
- [FortiGate Documentation](https://docs.fortinet.com/)
- [EVE-NG Cookbook](https://www.eve-ng.net/index.php/documentation/)

---

## Auteur

**Nelson Bandos**  
📧 nelson.bandos@proton.me  
🔗 [GitHub](https://github.com/Nelson2410)

---

