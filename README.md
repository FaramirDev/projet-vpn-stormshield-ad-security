# **🛡️ Projet OpenBank : Architecture Réseau Multi-Sites Haute Sécurité**

Alexis Rousseau - *Administrateur Système Réseau & Sécurité* 

## **📖 1\. Présentation du Projet**

Le projet **OpenBank** consiste à concevoir et déployer une infrastructure réseau résiliente et sécurisée pour une banque disposant d'un siège social (Paris), d'une agence distante (Nantes) et d'une force de travail nomade (Télétravail).

L'enjeu technique repose sur l'implémentation d'une stratégie de **défense en profondeur**, garantissant la confidentialité des flux bancaires et le contrôle strict des accès via une approche **Zero-Trust**.

---

## Objectifs principaux :
- Interconnexion sécurisée des sites via des tunnels VPN robustes entre Paris et Nantes.

- Contrôle strict des accès internet via un proxy avec inspection de flux chiffrés.

- Mise en place du Zero Trust (ZTNA) pour les accès distants.

- Authentification forte (MFA) pour garantir l'identité des collaborateurs.

---
## Tâches Réalisées
### 1. Administration Système (ADDS & GPO)
- Déploiement et configuration de Windows Server 2022, Principale Paris et lecture Seule pour Nantes avec lien de réplication.

- Structuration de l'Active Directory (Unités d'Organisation, Groupes, Utilisateurs).

#### Mise en œuvre de GPO critiques :

- Déploiement automatique du certificat racine du Firewall.

- Restrictions des périphériques de stockage (USB).

- Configuration de l'authentification mise en cache pour le mode hors-ligne.

- Configuration spécifique pour une Utilisatrice soufrant d'Handicape Visuel.

- Deconnexion des Utilisateurs hors plage d'horaire Autorisé 

### 2. Sécurité Réseau & Filtrage (Stormshield SNS)
- Couplage LDAP : Intégration du Firewall à l'Active Directory pour un filtrage par identité.

- Proxy Transparent HTTP/HTTPS : Mise en place du déchiffrement SSL pour l'audit des flux chiffrés.

- ACL & Filtrage d'URL : Politiques de navigation par Identification et Filtrage des Flux sur internets ( blocage a certaine catégorie d'URL )

### 3. Connectivité VPN & Chiffrement
- VPN IPsec Site-à-Site : Interconnexion Paris-Nantes avec authentification par certificats X.509.

- Optimisation Post-Quantique : Utilisation d'algorithmes de chiffrement modernes (AES-256 GCM, DH Group 19).

### 4. Accès Distant Haute Sécurité (ZTNA / MFA)
- VPN SSL : Configuration des pools d'adresses dynamiques avec segmentation par tunnel.

- MFA (Multi-Factor Authentication) : Double validation via mot de passe AD + code TOTP.

- ZTNA (Zero Trust Network Access) : Vérification de la posture de sécurité du poste (ex: blocage de l'accès si l'utilisateur possède des droits d'administrateur local).
---


## **💻 2\. Architecture Technique**

L'infrastructure repose sur une maquette virtualisée sous VirtualBox simulant un environnement de production réel :
* **Réseau WAN (Simulé) :** `192.56.253.0/24` (Interconnexion des Firewalls et accès Nomade).  
* **LAN Paris (HQ) :** `10.0.1.0/24` (Zone de confiance principale).  
* **LAN Nantes (Agence) :** `10.0.2.0/24` (Zone de confiance secondaire). 
--- 
* **Pools VPN SSL :** `10.0.5.0/24` (UDP) et `10.0.6.0/24` (TCP).
### **Schéma Architecture**

![Architecture](./CAPTURES/Architecture_Infr_Openbank_v2.png)


---

### Configuration Réseau

| Site **Paris**   | Interface Réseau 1  | Interface Réseau 2 | Gateway |
|------------------|---------------------|--------------------| ------- |
| **SNS Firewall Paris**  |  `10.0.1.1`      | `192.56.253.1` | |
| **DC Paris**            |  `10.0.1.2`      |                | `10.0.1.1` |
| **Client Paris**        |  `10.0.1.0/24`   |                | `10.0.1.1` |

---

| Site **Nantes**   | Interface Réseau 1  | Interface Réseau 2 | Gateway |
|-------------------|---------------------|--------------------| ------- |
| **SNS Firewall Nantes**  |  `10.0.2.1`      | `192.56.253.2` |  |
| **RODC Nantes**          |  `10.0.2.2`      |                | `10.0.2.1` |
| **Client Nantes**        |  `10.0.2.0/24`   |                | `10.0.2.1` |

---
| Nomade **Teletravail**   | Interface Réseau 1  | Interface Réseau 2 |
|-------------------|---------------------|-------------------|
|   Machine Nomade  | `192.56.253.0/24` | `Pool VPN SSL` |


### **Composants Infrastructure**

* **Services d'Identité :** Windows Server 2022 (ADDS Maître à Paris \+ RODC à Nantes).  
* **Sécurité Périmétrique :** Firewalls Stormshield Network Security (SNS) v4.8.6.  
* **Clients :** Windows 11 Pro intégrés au domaine.


---

## 🛡️ **3\. Implémentations Critiques**

### **Sécurité n°1 - Nomade VPN SSL (ZTNA & MFA)**

L'accès distant repose sur le couple identifiant/mot de passe présent dans l'Annuaire Active Directory du Domaine, ainsi que la mise en place de MFA et ZTNA :

- **MFA (Multi-Factor Authentication) :** Validation via TOTP (Google Authenticator) après authentification LDAP et Synchronisation du Compte. 

- **ZTNA (Zero Trust Network Access) :** Vérification de la posture du poste avant l'établissement du tunnel.  
   * *Critère de Vérification :* 
        - L'utilisateur ne possède pas des droits **Administrateur Local**.
        - La Machine est rattaché au Domaine `OpenBank.loc` 
        - L'Utilisateur est membre du Domaine `OpenBank.loc` 

### **Sécurité n°2 - Interconnexion Inter-Site (VPN IPsec)**

* **VPN IPSec :** Mise en place d'un Tunnel VPNIpsec pour relier les deux sites.
* **Authentification :** Certificats numériques X.509 (PKI interne).  
* **Chiffrement Post-Quantique :** Algorithmes AES-256 GCM et groupes Diffie-Hellman 21 (Elliptic Curve) pour une résilience long terme.

### **Sécurité n°3 - Inspection de Flux (Proxy & SSL Inspection)**

* **Authentification :** Redirection vers Portail d'Authentification ( Synchronisation AD - Firewall ) pour un accès internet 
* **Proxy Transparent :** Interception automatique des flux 80/443.  
* **Politique de Filtrage Web:** Catégorie de Site Refusé par la DSI.  
* **Déchiffrement HTTPS :** Inspection du trafic chiffré pour bloquer les menaces (Malwares, Phishing) via un certificat d'autorité (CA) déployé par GPO.

---

## **🏛️ 4\. Gouvernance Active Directory (GPO)**

Le pilotage des postes de travail est centralisé via des stratégies de groupe optimisées :

* **Inclusion (Ana Garcia) \- Handicap Visuel :** Paramétrage automatique de l'accessibilité (Loupe, Narrateur, Contraste élevé).  
* **Sécurité Physique :** Désactivation stricte des disques amovibles (USB), sauf pour le service IT.  
* **Conformité Horaire :** Interdiction de connexion et déconnexion forcée entre **20h00 et 06h00**.  
* **Confiance SSL :** Déploiement du certificat racine du Firewall sur l'ensemble du parc.  
* **Mise en Cache** : du Domaine de la machine pour les postes nomades

---

## **📁 5\. Livrables Techniques**

Les fichiers suivants constituent le dossier d'ingénierie du projet :

1. **Procédure d'Exploitation Technique:** rédaction d’une Procédure technique  détaillant la mise en place de la configuration de l’infrastructure.   
   Comprenant :   
   1. **PARTIE A : Configuration des Autorités de Certifications ( CA )**  
   2. **PARTIE B : OPTIMISATION du VPN Ipsec Inter-Site**  
   3. **PARTIE C : Déploiement du Proxy HTTP/HTTPS avec Filtrage**  
   4. **PARTIE D : Déploiement du VPN SSL avec ZTNA et MFA**

---

## **✅ 6\. Validation du Bon Fonctionnement** 

Afin de valider la mission, les preuves suivantes sont intégrées :

* Installation Serveur Win 22 PARIS \- ADDS & Serveur Console Win Nantes \- RODC   
* Installation Firewall SNS Stormshield Paris et SNS Stormshield Nantes  
* VPN Ipsec entre Paris et Nantes, certificat \+ Clé post-quantique  
* Démonstration Connexion Réseau Inter-Site  
* Domaine \+ Réplication Nantes ( RODC )   
* Création UO & Utilisateurs & Groupe de Sécurité  
* Création & Démonstration des GPOs   
* Politique d’Authentification ( Synchronisation Firewall Stormshield avec AD ) pour accès à internet ( redirection vers Portail Captif )  
* Politique de Filtrage de site Web ( Autorisé et Refusé )   
* VPN SSL \+ ZTNA \+ MFA pour les Travailleur Nomade   
* Supervision et Log

---

## **⚙️ Stack Technique**

* **Firewall :** Stormshield SNS v4.8.6  
* **OS Serveur :** Windows Server 2022 (ADDS, DNS, DHCP, RODC)  
* **Client VPN :** Stormshield SSL VPN Client v4.0.10  
* **Hyperviseur :** VirtualBox (Réseau interne segmenté)

---
# 📋 Demonstration du Bon Fonctionnement

## 1. Installation Serveur Win 22 AD PARIS 

![Installation Serveur Win 22 Paris](./CAPTURES/Installation_Serveur_DC_Paris.png)

## 2. Installation Serveur Win 22 RODC Nantes

![Installation Serveur Win 22 RODC Nantes](./CAPTURES/Installation_Serveur_RODC_Nantes.png)

## 3. Installation Firewall SNS Stormshield 

![Installation Firewall SNS](./CAPTURES/Installation_Firewall_SNS.png)

## 4. VPN Ipsec entre Paris et Nantes, certificat \+ Clé post-quantique  

- Ici, l'extremité du Firewall SNS Paris

![Configuration Passerelle VPN IPsec](./CAPTURES/04_Configuration_VPN_Ipsec.png)

## 5. Réplication Nantes ( RODC )  

![Réplication RODC Nantes](./CAPTURES/Repplication_RODC_Nantes.png)

## 6. Création UO & Utilisateurs & Groupe de Sécurité  

### Partie 1. GROUPE DE SECURITE
- `G_DG` : Pour notre Direction, qui poura avoir besoin de droit spécifique
---
- `G_U_IT_Admin` : Pour des Autorisations spécifique.
- `G_U_Banque_Paris` : Pour des droits spécifique au service de paris
- `G_U_Banque_Nantes` : Pour anticiper des Droits spécifique au besoin
---
- `G_U_Employes` : Pour l'ensemble de employé, et un restriction des Heures de travail
- `G_U_Internet` : Pour un Acces Internet et Mises a jours. 
- `G_U_Managers` : Pour couvrir l'ensemble des Mangers de l'entreprise
---
- `G_U_Resp_IT` : Pour les responsables du département IT
- `G_U_Resp_Banque_Nantes` : Pour les responsables du service Banque sur Nantes
- `G_U_Resp_Banque_Paris` : Pour les responsables du service Banque sur Paris
---
- `G_U_VPN_Acces_Nantes` : Pour les Utilisateurs de Nantes ayant le droit au VPN
- `G_U_VPN_Acces_Paris` : Pour les Utilisateurs de Paris ayant le droit au VPN
---
- `G_PC_IT` : Pour les PC du Service IT, et la GPO spécifique des Disques Amovibles

![Unite Organisation + Groupe Sécurité](./CAPTURES/UniteOrganisation_Utilisateurs_PC_01.png)

### Partie 2. Utilisateurs  & Affectation

**Tableau** représentation les Membres d'OpenBank avec Affectations de groupes de sécurités. 

---

| Membre  | OU | Compte | Groupe Sécurité Affecté |
|---------|----|--------|----------------|
| **Louise Chapat (DG)** | Direction |lchapat |`G_U_DG`,`G_U_Internet` |
||||
| *Site - PARIS* |||
| **Samir Assaf (DSI)** | Paris/IT | sassaf |`G_U_Managers`, `G_U_Resp_IT`, `G_U_IT_Admin`,`G_U_Internet`,`G_U_Employes`, |
| **Paul Bokadi** | Paris/IT | pbokali |`G_U_IT_Admin`,`G_U_Internet`,`G_U_Employes`, |
| **Alexis Rousseau** | Paris/IT | arousseau |`G_U_IT_Admin`,`G_U_Internet`,`G_U_Employes`, `G_U_VPN_Acces_Paris`|
||||
| **Sabrina Ouazani** | Paris/Banque | souazani |`G_U_Managers`, `G_U_Resp_Banque_Paris`, `G_U_Banque_Paris`,`G_U_Internet`,`G_U_Employes`, |
| **Lucie Garrido** | Paris/Banque | lgarrido |`G_U_Banque_Paris`,`G_U_Internet`,`G_U_Employes`, |
| **David Azoulay** | Paris/Banque | dazoulay |`G_U_Banque_Paris`,`G_U_Internet`,`G_U_Employes`, |
||||
| *Site - Nantes* |||
| **Théo Perier** | Nantes/Banque | tperier |`G_U_Managers`, `G_U_Resp_Banque_Nantes`, `G_U_anque_Nantes`,`G_U_Internet`,`G_U_Employes`, |
| **Ana Garcia** | Nantes/Banque | agarcia |`G_U_Banque_Nantes`,`G_U_Internet`,`G_U_Employes`, `G_U_VPN_Acces_Nantes`|
| **Salif Diallo** | Nantes/Banque | sdiallo |`G_U_Banque_Nantes`,`G_U_Internet`,`G_UEmployes`, |

![Utilisateurs](./CAPTURES/UniteOrganisation_Utilisateurs_PC_02.png)

## 7. Création & Démonstration des GPOs

### Partie 1. Ana Garcia - Handicap Visuel ( Site Nantes / Banque )

- **Demande** : Paramétrage automatique de l'accessibilité (Loupe, Narrateur, Contraste élevé).  

![Configuration GPO Ana Garcia](./CAPTURES/GPO_AnaGarcia_01.png)

- **Demonstration** GPO Ana Garcia 

### Partie 2. Controle Acces des Disques Amovibles

- **Demande** : Seul le Service IT peux avoir acces aux Disques Amovibles

![Configuration GPO Disques Amovibles](./CAPTURES/GPO_DisquesAmovibles_01.png)

- **Demonstrations** ( exemple : avec l'utilisatrice Ana Garcia )

### Partie 3. Deconnexion Forcé Hors Plage Horaire

- **Demande** : Utilisateurs ne doivent avoir accès au Domaine hors plage Horaire de Travai Autorisé ( 6h-20h )

- Pourquoi une GPO ?
    - Nous avons limité la connexion du Compte de 6h à 20h mais si l'utilisateurs a établie une connexion par exemple a 19h59, la connexion sera toujours établie après 20h. 
    - Necessite une GPO pour forcer la Deconnexion.

![Configuration GPO Deconnexion](./CAPTURES/GPO_Deconnexion_01.png)

- **Demonstration** :

![Demonstration GPO Deconnexion](./CAPTURES/GPO_Deconnexion_02.png)

### Partie 4. Déploiement sur Certificat de Sécurité SNS

- Demande : Déploiement du Certificat de Sécurité pour l'inspection SSL par le Firewall Stormshield

![Configuration GPO Certificat](./CAPTURES/GPO_CertificatAutorite_01.png)

### Partie 5. Mise en Cache pour VPN

- Demande : Forcer la Mise en Cache de la session Utilisateurs nottament utile pour les machines a destinations du Travail Nomade. 

![Configuration GPO MiseEnCache](./CAPTURES/GPO_MiseEnCache_01.png)

## Partie 6. Demonstration GPO - Pref Ana Garcia

- Config Réseau Machine | Ana Garcia | Siège Nantes

![Configuration Machine Ana Garcia](./CAPTURES/GPO_AnaGarcia_DEMO_ConfigReseau.png)

- Acces Aux Ressources de L'entreprise ( Réplication RODC)

![GPO Acces ressources](./CAPTURES/GPO_AnaGarcia_DEMO_AccesRessources.png)

- Les Rapports Etabli sur la Machine ( Mode Admin & Utilisateurs )

![GPO Rapport Commande](./CAPTURES/GPO_AnaGarcia_DEMO_rapports.png)

- Le Rapport mode Admin 

![GPO Rapport Administrateur PC](./CAPTURES/GPO_AnaGarcia_DEMO_rapportsModeAdmin.png)

- Le Rapport mode Utilisateur

![GPO Rapport Mode utilsiateurs U](./CAPTURES/GPO_AnaGarcia_DEMO_rapportsModeUtilisateurs.png)

- GPO Acces aux Disque Amovibles Refusé pour Ana Garcia

![GPO Disque Amovible Refusé](./CAPTURES/GPO_AnaGarcia_DEMO_DisqueAmovibles.png)

## 7. Politique d'Authentification Acces Internet

- Demande : Les Utilisateurs doivent etre Authentifié pour Accéder a Internet. 

### Partie 1. Synchronisation Annuaire LDAP ( AD ) avec les Firewalls SNS 

![Configuration Synchro LDAP ](./CAPTURES/PolitiqueFiltrage_SynchronisationLDAP.png)

### Partie 2. Activation du Portail Captif pour s'authentifié

![Configuration Activation Portail ](./CAPTURES/PolitiqueFiltrage_ActivationPortail.png)

### Partie 3. Redirection vers le Portail via les ACL

![Configuration ACL Redirection](./CAPTURES/PolitiqueFiltrage_ACL.png)

### Partie 4. Demonstration

![Demonstration Redirection](./CAPTURES/PolitiqueFiltrage_RedirectionWEB.png)

![Demonstration Acces WIkipedia](./CAPTURES/PolitiqueFiltrage_AccesWikipedia.png)

## 8. Politique de Filtrage de site Web ( Autorisé et Refusé )

- **Demande** : La DSI souhaite bloqué l'acces a certain site et certaine catégorie d'url.

### Partie 1. Création des Groupes d'URL

![Creation Groupe URL](./CAPTURES/PolitiqueFiltrageURL_GroupeCategorie.png)

### Partie 2. Création de la Politique de Filtrage

![Creation Politique Filtrage URL](./CAPTURES/PolitiqueFiltrageURL_CreationPolitique.png)

### Partie 3. Ajout de la Catégorie dans les ACL

![Aujout Politique aux ACL](./CAPTURES/PolitiqueFiltrageURL_AjoutACL.png)

### Partie 4. Demonstration Filtrage

![Demonstration 01 ](./CAPTURES/PolitiqueFiltrageURL_Demonstration01.png)

![Demonstration 02](./CAPTURES/PolitiqueFiltrageURL_Demonstration02.png)

## 9. VPN SSL \+ ZTNA \+ MFA pour les Travailleur Nomade 

Pour toutes les Configurations détaillé, je vous invite a lire la documentation Technique Etabli dans les livrables, voici les grandes étapes. 

### Partie 1. Configuration VPN SSL

![VPN SSL Activation](./CAPTURES/VPN_SSL_Activation_ZTNA.png)

### Partie 2. Politique d'Authentification

![VPN SSL Politique Authentification](./CAPTURES/VPN_SSL_PolitiqueAuthentification.png)

### Partie 3. Préparation Machine Nomade - Domaine

![VPN SSL Preparation ](./CAPTURES/VPN_SSL_Preparation.png)

### Partie 4. Demonstration 

![VPN SSL Demonstration 01](./CAPTURES/VPN_SSL_Demonstration_01.png)

![VPN SSL Demonstration 02](./CAPTURES/VPN_SSL_Demonstration_02.png)

## 10. Supervision & LOG

### Partie 1. Utilisateurs Connecté au Domaine 

![Supervision LOG Utilisateurs](./CAPTURES/Supervision_utilisateurs.png)

### Partie 2. Suivi des Connexions Internet ( Autorisé et Bloqué )

![Supervision LOG Utilisateurs](./CAPTURES/Supervision_Web.png)

### Partie 3. Suivi des Logs VPN SSL et Connexion établie

![Supervision LOG Utilisateurs](./CAPTURES/Supervision_VPNSSL.png)


---
## Conclusion
L'architecture OpenBank répond aux plus hauts standards de sécurité actuels. Le passage d'une sécurité périmétrique classique à une approche centrée sur l'identité (MFA) et la posture du poste (ZTNA) permet de protéger l'entreprise contre l'exfiltration de données et les mouvements latéraux de malwares.

---
*Projet réalisé dans le cadre de la formation Administrateur Systèmes, Réseaux et Sécurité.*

---
---
## Licence
Ce projet est sous licence **Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)**.
Toute utilisation commerciale est strictement interdite sans autorisation préalable. Consultez le fichier [LICENSE](./LICENSE) pour plus de détails.

---
Alexis Rousseau - *Administrateur Système Réseau & Sécurité* 