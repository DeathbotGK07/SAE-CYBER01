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
