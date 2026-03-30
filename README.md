# SAE4.Cyber.01 - Sécuriser un système d’information 

Ce dépôt contient les livrables de la SAE 4.Cyber.01 du BUT R&T (Semestre 4). L'objectif est de concevoir, maquetter et sécuriser l'architecture réseau d'une entreprise répartie sur deux sites distants.

## Contexte du projet

L'entreprise dispose de deux sites géographiques. Chaque site possède une architecture LAN segmentée. L'interconnexion entre les sites est assurée par un tunnel sécurisé traversant un réseau public (simulation Internet).

**Outil utilisé :** Cisco Packet Tracer

##  Architecture Réseau
Outil utilisé : Cisco Packet Tracer

<img width="1294" height="628" alt="image" src="Capture d’écran 2026-03-30 124615.png" />

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

# 📖 Dossier d'Architecture : Interconnexion WAN Sécurisée & Zero Trust

## 1. Tableau d'Adressage IP

| Équipement | Rôle / Localisation | Interface | Adresse IP | Masque de sous-réseau | Passerelle par défaut |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **PC0** | Admin (Site 1) | FastEthernet0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.254 |
| **PC1** | Production (Site 1) | FastEthernet0 | 192.168.2.20 | 255.255.255.0 | 192.168.2.254 |
| **PC2** | Service (Site 1) | FastEthernet0 | 192.168.3.30 | 255.255.255.0 | 192.168.3.254 |
| **Switch0** | Multicouche L3 (Site 1) | Vlan 10 (Admin) | 192.168.1.254 | 255.255.255.0 | - |
| | | Vlan 20 (Prod) | 192.168.2.254 | 255.255.255.0 | - |
| | | Vlan 30 (Service) | 192.168.3.254 | 255.255.255.0 | - |
| | | Fa0/4 (Vers PF1) | 10.255.255.2 | 255.255.255.252 | - |
| **PF1 (ASA)**| Pare-feu (Site 1) | Vlan 1 (Inside) | 10.255.255.1 | 255.255.255.252 | - |
| | | Vlan 2 (Outside) | 10.255.254.2 | 255.255.255.252 | - |
| **Router2** | Routeur VPN (Site 1) | Gi0/0 (LAN) | 10.255.254.1 | 255.255.255.252 | - |
| | | Gi0/1 (WAN) | 10.1.0.1 | 255.255.255.0 | - |
| | | Tunnel0 | 172.16.0.1 | 255.255.255.252 | - |
| **Router3** | FAI / Internet | Gi0/0 (Vers R2) | 10.1.0.2 | 255.255.255.0 | - |
| | | Gi0/1 (Vers R4) | 10.2.0.2 | 255.255.255.0 | - |
| **Router4** | Routeur VPN (Site 2) | Gi0/1 (LAN) | 10.255.253.1 | 255.255.255.252 | - |
| | | Gi0/0 (WAN) | 10.2.0.1 | 255.255.255.0 | - |
| | | Tunnel0 | 172.16.0.2 | 255.255.255.252 | - |
| **PF2 (ASA)**| Pare-feu (Site 2) | Vlan 1 (Inside) | 10.255.255.5 | 255.255.255.252 | - |
| | | Vlan 2 (Outside) | 10.255.253.2 | 255.255.255.252 | - |
| **Switch1** | Multicouche L3 (Site 2) | Vlan 100 (Admin) | 192.168.10.1 | 255.255.255.0 | - |
| | | Vlan 200 (Prod) | 192.168.20.1 | 255.255.255.0 | - |
| | | Vlan 300 (Service)| 192.168.30.1 | 255.255.255.0 | - |
| | | Fa0/4 (Vers PF2) | 10.255.255.6 | 255.255.255.252 | - |
| **PC3** | Admin (Site 2) | FastEthernet0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| **PC4** | Production (Site 2) | FastEthernet0 | 192.168.20.20 | 255.255.255.0 | 192.168.20.1 |
| **PC5** | Service (Site 2) | FastEthernet0 | 192.168.30.30 | 255.255.255.0 | 192.168.30.1 |

---

## 2. Configurations des Équipements

### 🏢 SITE 1 : Cœur de Réseau & Sécurité

**1. Switch Multicouche (Switch0)**
```cisco
conf t
ip routing
ip route 0.0.0.0 0.0.0.0 10.255.255.1

! Configuration des ACL d'isolation (Zero Trust)
ip access-list extended ISOLATE_PROD
 permit icmp 192.168.2.0 0.0.0.255 any echo-reply
 deny ip 192.168.2.0 0.0.0.255 192.168.0.0 0.0.255.255
 permit ip any any
ip access-list extended ISOLATE_SERVICE
 permit icmp 192.168.3.0 0.0.0.255 any echo-reply
 deny ip 192.168.3.0 0.0.0.255 192.168.0.0 0.0.255.255
 permit ip any any

! Interfaces Virtuelles (SVI) et application des ACL
interface Vlan 10
 ip address 192.168.1.254 255.255.255.0
 no shutdown
interface Vlan 20
 ip address 192.168.2.254 255.255.255.0
 ip access-group ISOLATE_PROD in
 no shutdown
interface Vlan 30
 ip address 192.168.3.254 255.255.255.0
 ip access-group ISOLATE_SERVICE in
 no shutdown

! Lien de routage vers ASA
interface FastEthernet0/4
 no switchport
 ip address 10.255.255.2 255.255.255.252
 no shutdown
exit
```

**2. Pare-Feu ASA (PF1)**
```cisco
conf t
! Routage
route outside 0.0.0.0 0.0.0.0 10.255.254.1 1
route inside 192.168.1.0 255.255.255.0 10.255.255.2 1
route inside 192.168.2.0 255.255.255.0 10.255.255.2 1
route inside 192.168.3.0 255.255.255.0 10.255.255.2 1

! ACL Inside (Contrôle du trafic sortant vers le WAN)
access-list INSIDE_V3 extended permit ip 192.168.1.0 255.255.255.0 any
access-list INSIDE_V3 extended permit icmp 192.168.2.0 255.255.255.0 192.168.10.0 255.255.255.0
access-list INSIDE_V3 extended permit icmp 192.168.3.0 255.255.255.0 192.168.10.0 255.255.255.0
access-list INSIDE_V3 extended deny ip 192.168.2.0 255.255.255.0 192.168.0.0 255.255.0.0
access-list INSIDE_V3 extended deny ip 192.168.3.0 255.255.255.0 192.168.0.0 255.255.0.0
access-list INSIDE_V3 extended permit ip any any

! ACL Outside (Contrôle du trafic entrant depuis le tunnel VPN)
access-list OUTSIDE_V3 extended permit ip 192.168.10.0 255.255.255.0 192.168.1.0 255.255.255.0
access-list OUTSIDE_V3 extended permit ip 192.168.10.0 255.255.255.0 192.168.2.0 255.255.255.0
access-list OUTSIDE_V3 extended permit ip 192.168.10.0 255.255.255.0 192.168.3.0 255.255.255.0
access-list OUTSIDE_V3 extended permit icmp any any

! Application sur les interfaces
access-group INSIDE_V3 in interface inside
access-group OUTSIDE_V3 in interface outside
exit
```

**3. Routeur VPN (Router2)**
```cisco
conf t
! Tunnel GRE & VPN IPsec
crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key progtr00 address 10.2.0.1
crypto ipsec transform-set MYSET esp-aes esp-sha-hmac
crypto map MYMAP 10 ipsec-isakmp 
 set peer 10.2.0.1
 set transform-set MYSET 
 match address 110
 
! ACL pour déclencher le VPN
access-list 110 permit gre host 10.1.0.1 host 10.2.0.1

! Configuration Interfaces
interface GigabitEthernet0/0
 ip address 10.255.254.1 255.255.255.252
 no shutdown
interface GigabitEthernet0/1
 ip address 10.1.0.1 255.255.255.0
 crypto map MYMAP
 no shutdown
interface Tunnel0
 ip address 172.16.0.1 255.255.255.252
 tunnel source GigabitEthernet0/1
 tunnel destination 10.2.0.1

! Routage OSPF et Statique
router ospf 1
 network 10.1.0.0 0.0.0.255 area 0
 network 10.255.254.0 0.0.0.3 area 0
 network 172.16.0.0 0.0.0.3 area 0
ip route 192.168.1.0 255.255.255.0 10.255.254.2 
ip route 192.168.2.0 255.255.255.0 10.255.254.2 
ip route 192.168.3.0 255.255.255.0 10.255.254.2 
ip route 192.168.10.0 255.255.255.0 172.16.0.2 
ip route 192.168.20.0 255.255.255.0 172.16.0.2 
ip route 192.168.30.0 255.255.255.0 172.16.0.2 
exit
```

---

### ☁️ WAN : Fournisseur d'Accès Internet

**Routeur ISP (Router3)**
```cisco
conf t
interface GigabitEthernet0/0
 ip address 10.1.0.2 255.255.255.0
 no shutdown
interface GigabitEthernet0/1
 ip address 10.2.0.2 255.255.255.0
 no shutdown

router ospf 1
 network 10.1.0.0 0.0.0.255 area 0
 network 10.2.0.0 0.0.0.255 area 0
exit
```

---

### 🏢 SITE 2 : Cœur de Réseau & Sécurité

**1. Routeur VPN (Router4)**
```cisco
conf t
! Tunnel GRE & VPN IPsec
crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key progtr00 address 10.1.0.1
crypto ipsec transform-set MYSET esp-aes esp-sha-hmac
crypto map MYMAP 10 ipsec-isakmp 
 set peer 10.1.0.1
 set transform-set MYSET 
 match address 110

! ACL pour déclencher le VPN
access-list 110 permit gre host 10.2.0.1 host 10.1.0.1

! Interfaces
interface GigabitEthernet0/0
 ip address 10.2.0.1 255.255.255.0
 crypto map MYMAP
 no shutdown
interface GigabitEthernet0/1
 ip address 10.255.253.1 255.255.255.252
 no shutdown
interface Tunnel0
 ip address 172.16.0.2 255.255.255.252
 tunnel source GigabitEthernet0/0
 tunnel destination 10.1.0.1

! Routage OSPF et Statique
router ospf 1
 network 10.2.0.0 0.0.0.255 area 0
 network 10.255.253.0 0.0.0.3 area 0
 network 172.16.0.0 0.0.0.3 area 0
ip route 192.168.10.0 255.255.255.0 10.255.253.2 
ip route 192.168.20.0 255.255.255.0 10.255.253.2 
ip route 192.168.30.0 255.255.255.0 10.255.253.2 
ip route 192.168.1.0 255.255.255.0 172.16.0.1 
ip route 192.168.2.0 255.255.255.0 172.16.0.1 
ip route 192.168.3.0 255.255.255.0 172.16.0.1 
exit
```

**2. Pare-Feu ASA (PF2)**
```cisco
conf t
! Routage
route outside 0.0.0.0 0.0.0.0 10.255.253.1 1
route inside 192.168.10.0 255.255.255.0 10.255.255.6 1
route inside 192.168.20.0 255.255.255.0 10.255.255.6 1
route inside 192.168.30.0 255.255.255.0 10.255.255.6 1

! ACL Inside (Contrôle du trafic sortant vers le WAN)
access-list INSIDE_V2 extended permit ip 192.168.10.0 255.255.255.0 any
access-list INSIDE_V2 extended permit icmp 192.168.20.0 255.255.255.0 192.168.1.0 255.255.255.0
access-list INSIDE_V2 extended permit icmp 192.168.30.0 255.255.255.0 192.168.1.0 255.255.255.0
access-list INSIDE_V2 extended deny ip 192.168.20.0 255.255.255.0 192.168.0.0 255.255.0.0
access-list INSIDE_V2 extended deny ip 192.168.30.0 255.255.255.0 192.168.0.0 255.255.0.0
access-list INSIDE_V2 extended permit ip any any

! ACL Outside (Contrôle du trafic entrant depuis le tunnel VPN)
access-list OUTSIDE_V2 extended permit ip 192.168.1.0 255.255.255.0 192.168.10.0 255.255.255.0
access-list OUTSIDE_V2 extended permit ip 192.168.1.0 255.255.255.0 192.168.20.0 255.255.255.0
access-list OUTSIDE_V2 extended permit ip 192.168.1.0 255.255.255.0 192.168.30.0 255.255.255.0
access-list OUTSIDE_V2 extended permit icmp any any

! Application sur les interfaces
access-group INSIDE_V2 in interface inside
access-group OUTSIDE_V2 in interface outside
exit
```

**3. Switch Multicouche (Switch1)**
```cisco
conf t
ip routing
ip route 0.0.0.0 0.0.0.0 10.255.255.5

! Configuration des ACL d'isolation (Zero Trust)
ip access-list extended ISOLATE_PROD2
 permit icmp 192.168.20.0 0.0.0.255 any echo-reply
 deny ip 192.168.20.0 0.0.0.255 192.168.0.0 0.0.255.255
 permit ip any any
ip access-list extended ISOLATE_SERVICE2
 permit icmp 192.168.30.0 0.0.0.255 any echo-reply
 deny ip 192.168.30.0 0.0.0.255 192.168.0.0 0.0.255.255
 permit ip any any

! Interfaces Virtuelles (SVI) et application des ACL
interface Vlan 100
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface Vlan 200
 ip address 192.168.20.1 255.255.255.0
 ip access-group ISOLATE_PROD2 in
 no shutdown
interface Vlan 300
 ip address 192.168.30.1 255.255.255.0
 ip access-group ISOLATE_SERVICE2 in
 no shutdown

! Lien de routage vers ASA
interface FastEthernet0/4
 no switchport
 ip address 10.255.255.6 255.255.255.252
 no shutdown
exit
```

## DNS DNSSEC

# Configuration DNSSEC — domaine entreprise.lan

## 1. Configuration de la zone dans BIND

```conf
// Gestion de la zone entreprise.lan avec DNSSEC
zone "entreprise.lan" {
    type master;
    file "/etc/bind/zones/db.entreprise.lan";
    key-directory "/etc/bind/keys";

    // Sécurité : interdiction des transferts de zone
    allow-transfer { none; };

    // Activation de DNSSEC automatique
    dnssec-policy default;
    inline-signing yes;
};
```

## 2. Fichier de zone signé : db.entreprise.lan

```dns
$TTL    86400   ; TTL réduit à 24h (ANSSI : signatures plus fraîches)

@       IN      SOA     ns1.entreprise.lan. admin.entreprise.lan. (
                        2026032701 ; Serial
                        86400      ; Refresh
                        7200       ; Retry
                        1209600    ; Expire
                        86400 )    ; Negative Cache (avant 604800)
;

@       IN      NS      ns1.entreprise.lan.
ns1     IN      A       10.0.2.15
www     IN      A       10.0.2.15
```

## 3. Options globales — named.conf.options

```conf
options {
    directory "/var/cache/bind";

    // Sécurité + DNSSEC
    dnssec-validation auto;
    listen-on-v6 { any; };

    // Forwarders éventuels (désactivés dans notre cas)
    // forwarders {
    //     8.8.8.8;
    //     1.1.1.1;
    // };

    // Recommandation ANSSI : masquer la version de BIND
    version "none";
};
```

## 4. Vérification DNSSEC

### Commande exécutée

```bash
dig @10.0.2.15 www.entreprise.lan +dnssec
```

### Résultat obtenu

```plaintext
;; ANSWER SECTION:
www.entreprise.lan. 86400 IN A 10.0.2.15
www.entreprise.lan. 86400 IN RRSIG A 13 3 86400 20260410033836 20260327130206 59609 entreprise.lan.
7PuNzotl93ttZsSNgezAfA8Yhl8nxOlgTYlePTabkyVXWORYynKqGuGl4F25KRDyOQhJQkfCrZSVO22VrwHVMw==
```
# Sécurisation complète d'un serveur Apache2 en HTTPS avec une CA Locale (Debian)

## SECURISAION WEB
Lors du déploiement d'un serveur web en environnement de développement ou sur un réseau local (Machine Virtuelle), l'utilisation d'un simple certificat auto-signé génère systématiquement une alerte de sécurité ("Connexion non sécurisée") sur les navigateurs modernes. 

L'objectif de cette intervention est de passer un serveur Apache 2 de HTTP à HTTPS de manière propre, en résolvant les conflits de configuration, en créant une véritable Autorité de Certification (CA) locale pour valider la chaîne de confiance, et en forçant la redirection des utilisateurs vers la version sécurisée.

**Environnement technique :**
- **OS :** Debian (Machine Virtuelle)
- **Serveur Web :** Apache 2.4
- **Réseau :** IP de la VM `10.0.2.15` 
- **PKI (Public Key Infrastructure) :** OpenSSL (Localité configurée : Béthune)

---

##  Étape 1 : Diagnostic et résolution du conflit de démarrage (Port 443)

Lors de l'activation initiale du SSL, le redémarrage d'Apache échoue avec une erreur fatale. L'analyse des logs (`systemctl status apache2`) révèle le message suivant : 
`(98)Address already in use: AH00072: make_sock: could not bind to address [::]:443`.

Le port 443 est bloqué car Apache tente de s'y attacher (bind) deux fois suite à une erreur de configuration.

1. **Recherche de la source du conflit :**
   On cherche où la directive d'écoute des ports est déclarée dans les fichiers d'Apache.
   ```bash
   grep -r "Listen" /etc/apache2/

   #  Sécurisation d'un serveur Apache2 en HTTPS avec une CA Locale (Debian)

##  Contexte et Objectifs

Lors du déploiement d'un serveur web en environnement de développement ou sur un réseau local (machine virtuelle), l'utilisation d'un certificat auto-signé provoque une alerte de sécurité ("Connexion non sécurisée") sur les navigateurs modernes.

L'objectif est de :

- Passer Apache2 de **HTTP à HTTPS**
- Résoudre les conflits de configuration
- Mettre en place une **Autorité de Certification (CA) locale**
- Établir une **chaîne de confiance valide**
- Forcer la redirection vers HTTPS

---

##  Environnement technique

- **OS :** Debian (VM)
- **Serveur Web :** Apache 2.4
- **IP VM :** `10.0.2.15`
- **Loopback :** `127.0.0.1`
- **PKI :** OpenSSL

---

##  Étape 1 : Correction du conflit port 443

###  Erreur

```
(98)Address already in use: AH00072: could not bind to address [::]:443
```

###  Diagnostic

```bash
grep -r "Listen" /etc/apache2/
```

 Vérifier la présence de doublons dans :

```
/etc/apache2/ports.conf
```

###  Correction

```bash
sudo nano /etc/apache2/ports.conf
```

Garder uniquement :

```apache
<IfModule ssl_module>
    Listen 443
</IfModule>
```

###  Redémarrage

```bash
sudo systemctl restart apache2
```

---

##  Étape 2 : Création de la CA locale

###  Préparation

```bash
mkdir ~/mon_ca && cd ~/mon_ca
```

###  Clé privée CA

```bash
openssl genrsa -out MaCA.key 2048
```

### Certificat racine

```bash
openssl req -x509 -new -nodes \
  -key MaCA.key \
  -sha256 -days 3650 \
  -out MaCA.pem \
  -subj "/C=FR/ST=Hauts-de-France/L=Bethune/O=Mon Infrastructure/CN=Ma Propre Autorite"
```

---

##  Étape 3 : Certificat serveur avec SAN

###  Génération clé + CSR

```bash
openssl genrsa -out serveur.key 2048

openssl req -new \
  -key serveur.key \
  -out serveur.csr \
  -subj "/C=FR/ST=Hauts-de-France/L=Bethune/O=Mon Serveur/CN=10.0.2.15"
```

###  Fichier SAN

```bash
cat > extfile.cnf <<EOF
subjectAltName = IP:10.0.2.15,IP:127.0.0.1
EOF
```

###  Signature

```bash
openssl x509 -req \
  -in serveur.csr \
  -CA MaCA.pem \
  -CAkey MaCA.key \
  -CAcreateserial \
  -out serveur.crt \
  -days 825 \
  -sha256 \
  -extfile extfile.cnf
```

---

##  Étape 4 : Configuration Apache SSL

###  Installation des certificats

```bash
sudo cp serveur.crt /etc/ssl/certs/apache-selfsigned.crt
sudo cp serveur.key /etc/ssl/private/apache-selfsigned.key
```

###  Configuration SSL

```bash
sudo nano /etc/apache2/sites-available/default-ssl.conf
```

Configurer :

```apache
SSLCertificateFile    /etc/ssl/certs/apache-selfsigned.crt
SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
```

###  Activation

```bash
sudo a2enmod ssl
sudo a2ensite default-ssl.conf
sudo apache2ctl configtest
```

Résultat attendu :

```
Syntax OK
```

###  Redémarrage

```bash
sudo systemctl restart apache2
```

---

##  Étape 5 : Redirection HTTP → HTTPS

###  Configuration

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Ajouter :

```apache
Redirect permanent / https://10.0.2.15/
```

###  Rechargement

```bash
sudo systemctl reload apache2
```

---

##  Étape 6 : Validation côté client

###  Rendre la CA accessible

```bash
sudo cp ~/mon_ca/MaCA.pem /var/www/html/MaCA.crt
```

###  Télécharger

```
http://10.0.2.15/MaCA.crt
```

###  Importer dans Firefox

1. Paramètres  
2. Vie privée et sécurité  
3. Certificats → Afficher les certificats  
4. Onglet **Autorités**  
5. Importer `MaCA.crt`

###  Important

Cocher :

```
Confirmer cette autorité de certification pour identifier les sites web
```

---

##  Résultat

-  HTTPS fonctionnel
-  Aucun avertissement navigateur
-  Redirection automatique HTTP → HTTPS
-  Chaîne de confiance valide
