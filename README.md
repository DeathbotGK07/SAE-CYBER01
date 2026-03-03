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
