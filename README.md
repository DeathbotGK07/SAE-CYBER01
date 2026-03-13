# SAE4.Cyber.01 - Sécuriser un système d’information 

Ce dépôt contient les livrables de la SAE 4.Cyber.01 du BUT R&T (Semestre 4). L'objectif est de concevoir, maquetter et sécuriser l'architecture réseau d'une entreprise répartie sur deux sites distants.

## Contexte du projet

L'entreprise dispose de deux sites géographiques. Chaque site possède une architecture LAN segmentée. L'interconnexion entre les sites est assurée par un tunnel sécurisé traversant un réseau public (simulation Internet).

**Outil utilisé :** Cisco Packet Tracer

##  Architecture Réseau
Outil utilisé : Cisco Packet Tracer

<img width="1294" height="628" alt="image" src="https://github.com/user-attachments/assets/b4917727-1b90-40d3-a994-aba8801db9e6" />

### Topologie
Le réseau est composé de deux sites connectés via un **Tunnel IPSEC GRE**.

Chaque site est divisé en 3 zones (VLANs) :
1.  **Service**
2.  **Production**
3.  **Admin**

### Politique de Sécurité (ACL)
Les règles de filtrage suivantes ont été implémentées :
*  **Réseau Admin :** Accès total à tous les réseaux (locaux et distants).
*  **Réseaux Service & Production :** Isolés. Ils ne peuvent accéder à aucun autre réseau (ni localement, ni sur le site distant).

## Équipe et Spécialisations

Ce projet a été réalisé par un quadrinôme. Chaque membre s'est spécialisé sur un aspect critique de la sécurité :

| Membre de l'équipe | Spécialisation | Description succincte |
| :--- | :--- | :--- |
| **Geraud Karbowski** | **Sécurisation DNS** | Mise en place de DNSSEC pour garantir l'authenticité des réponses DNS. |
| **Esteban Cubizolle** | **Sécurisation WEB** | Durcissement des serveurs Web (HTTPS, configurations sécurisées). |
| **Etan Robain** | **Tests de sécurité** | Scénarios d'attaques et vérification de la robustesse (Pentesting). |
| **Thibaut Lhernout** | **Recommandations ANSSI** | Audit de la maquette via la checklist officielle de l'ANSSI. |

# 📋 Tableau d'Adressage Complet de l'Infrastructure

## 🌐 Cœur de Réseau (Routage OSPF)
| Équipement | Interface | Adresse IP | Masque | Rôle |
| :--- | :--- | :--- | :--- | :--- |
| **Router 2** | Gi0/1 | `10.1.0.1` | `255.255.255.0` | Lien vers Router 3 |
| **Router 2** | Gi0/0 | `10.255.254.1` | `255.255.255.252` | Passerelle PF1 (Outside) |
| **Router 3** | Gi0/0 | `10.1.0.2` | `255.255.255.0` | Lien vers Router 2 |
| **Router 3** | Gi0/1 | `10.2.0.2` | `255.255.255.0` | Lien vers Router 4 |
| **Router 4** | Gi0/0 | `10.2.0.1` | `255.255.255.0` | Lien vers Router 3 |
| **Router 4** | Gi0/1 | `10.255.253.1` | `255.255.255.252` | Passerelle PF2 (Outside) |

---

## 🛡️ Site 1 (Côté Gauche)
**Passerelle vers Internet/WAN :** ASA PF1 (`10.255.254.2`)

| Équipement | Interface / VLAN | Adresse IP | Masque | Usage |
| :--- | :--- | :--- | :--- | :--- |
| **PF1 (ASA)** | Vlan 2 (Outside) | `10.255.254.2` | `255.255.255.252` | Lien vers Router 2 |
| **PF1 (ASA)** | Vlan 1 (Inside) | `10.255.255.1` | `255.255.255.252` | Lien vers Switch L3-1 |
| **Switch L3-1** | Fa0/4 (Routed) | `10.255.255.2` | `255.255.255.252` | Lien vers PF1 |
| **Switch L3-1** | **VLAN 10** | `192.168.1.254` | `255.255.255.0` | Passerelle **ADMIN-1** |
| **Switch L3-1** | **VLAN 20** | `192.168.2.254` | `255.255.255.0` | Passerelle **PROD-1** |
| **Switch L3-1** | **VLAN 30** | `192.168.3.254` | `255.255.255.0` | Passerelle **SERVICE-1** |
| **PC0** | FastEthernet0 | `192.168.1.10` | `255.255.255.0` | PC Admin (VLAN 10) |
| **PC1** | FastEthernet0 | `192.168.2.20` | `255.255.255.0` | PC Production (VLAN 20) |
| **PC2** | FastEthernet0 | `192.168.3.30` | `255.255.255.0` | PC Service (VLAN 30) |

---

## 🛡️ Site 2 (Côté Droit)
**Passerelle vers Internet/WAN :** ASA PF2 (`10.255.253.2`)

| Équipement | Interface / VLAN | Adresse IP | Masque | Usage |
| :--- | :--- | :--- | :--- | :--- |
| **PF2 (ASA)** | Vlan 2 (Outside) | `10.255.253.2` | `255.255.255.252` | Lien vers Router 4 |
| **PF2 (ASA)** | Vlan 1 (Inside) | `10.255.255.5` | `255.255.255.252` | Lien vers Switch L3-2 |
| **Switch L3-2** | Fa0/4 (Routed) | `10.255.255.6` | `255.255.255.252` | Lien vers PF2 |
| **Switch L3-2** | **VLAN 100** | `192.168.10.1` | `255.255.255.0` | Passerelle **ADMIN-2** |
| **Switch L3-2** | **VLAN 200** | `192.168.20.1` | `255.255.255.0` | Passerelle **PROD-2** |
| **Switch L3-2** | **VLAN 300** | `192.168.30.1` | `255.255.255.0` | Passerelle **SERVICE-2** |
| **PC3** | FastEthernet0 | `192.168.10.10` | `255.255.255.0` | PC Admin (VLAN 100) |
| **PC4** | FastEthernet0 | `192.168.20.20` | `255.255.255.0` | PC Production (VLAN 200) |
| **PC5** | FastEthernet0 | `192.168.30.30` | `255.255.255.0` | PC Service (VLAN 300) |

---

## 🛣️ Résumé des Routes par Défaut (Static Routes)
| Équipement | Commande de routage | Destination |
| :--- | :--- | :--- |
| **Switch L3-1** | `ip route 0.0.0.0 0.0.0.0 10.255.255.1` | Sortie via PF1 (Inside) |
| **ASA PF1** | `route outside 0.0.0.0 0.0.0.0 10.255.254.1 1` | Sortie via Router 2 |
| **Switch L3-2** | `ip route 0.0.0.0 0.0.0.0 10.255.255.5` | Sortie via PF2 (Inside) |
| **ASA PF2** | `route outside 0.0.0.0 0.0.0.0 10.255.253.1 1` | Sortie via Router 4 |
