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


## DNS DNSSEC

1. Problématique et Objectif

Le protocole DNS standard est vulnérable aux attaques de type DNS Cache Poisoning (empoisonnement de cache) et Man-in-the-Middle. Un attaquant pourrait détourner le trafic du réseau "Admin" vers un serveur malveillant en falsifiant les réponses UDP.
L'objectif est d'implémenter DNSSEC (Domain Name System Security Extensions) pour garantir l'intégrité et l'authenticité des résolutions de noms au sein de l'infrastructure entreprise.lan.
2. Détails de l'Architecture Technique

L'implémentation repose sur le serveur de noms BIND 9.18. Conformément aux recommandations de l'ANSSI, nous avons opté pour une politique de signature moderne :

    Algorithme : ECDSAP256SHA256 (Algorithme 13). Ce choix offre une sécurité équivalente au RSA-3072 tout en réduisant la taille des paquets DNS, évitant ainsi la fragmentation UDP.

    Gestion des clés (CSK) : Une clé combinée (Combined Signing Key) gère à la fois la signature des enregistrements (ZSK) et la validation de la zone (KSK).

    Durée de vie (TTL) : Ajustée à 86400s (24h) pour assurer une rotation et une fraîcheur optimale des signatures cryptographiques.

3. Configuration du Service (Extraits)
A. Options de sécurité (named.conf.options)

Nous avons activé la validation DNSSEC et masqué la version du service pour limiter la reconnaissance (Footprinting) :
Plaintext

options {
    dnssec-validation auto;
    version "none"; // Recommandation ANSSI
    listen-on-v6 { any; };
};

B. Politique de signature (named.conf.local)

La signature est automatisée via l'inline-signing pour garantir que chaque modification de la zone db.entreprise.lan déclenche une nouvelle signature RRSIG :
Plaintext

zone "entreprise.lan" {
    type master;
    file "/etc/bind/zones/db.entreprise.lan";
    key-directory "/etc/bind/keys";
    dnssec-policy default;
    inline-signing yes;
};

4. Résolution des contraintes de sécurité (Hardening)

L'implémentation a nécessité une intervention sur les couches de protection du système d'exploitation :

    AppArmor : Le profil de sécurité usr.sbin.named a été modifié pour autoriser explicitement l'écriture (rw) dans les répertoires /etc/bind/zones/ et /etc/bind/keys/. Sans cela, le moteur cryptographique de BIND était bloqué par le noyau.

    Droits UNIX : Application d'un chown bind:bind sur les dossiers de clés pour respecter le principe du moindre privilège.

5. Tests et Preuves de fonctionnement

La validité de la configuration a été confirmée par une requête dig avec le flag DO (DNSSEC OK).

Résultat du test :
Plaintext

;; ANSWER SECTION:
www.entreprise.lan.  86400  IN  A      10.0.2.15
www.entreprise.lan.  86400  IN  RRSIG  A 13 3 86400 20260410033836 ...

La présence de l'enregistrement RRSIG (Resource Record Signature) confirme que la réponse est signée cryptographiquement. Toute altération de l'adresse IP par un tiers rendrait la signature invalide, provoquant une erreur SERVFAIL chez le client, protégeant ainsi l'intégrité du réseau.
