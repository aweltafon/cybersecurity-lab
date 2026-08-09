# 🔎 Analyse des vulnérabilités

## 🎯 Objectif

Analyser les services et versions identifiés lors de la phase de reconnaissance afin d'identifier des vulnérabilités connues, d'évaluer leur niveau de risque et de vérifier leur exposition dans l'environnement de laboratoire.

## 🧪 Environnement

- Kali Linux — `192.168.56.104`
- Metasploitable — `192.168.56.101`
- VirtualBox
- Réseau de laboratoire — `192.168.56.0/24`

La machine Metasploitable est volontairement vulnérable et utilisée uniquement dans un environnement de laboratoire isolé.
---

## 1. Analyse du service FTP

### 🔍 Identification

Lors de la reconnaissance effectuée avec Nmap, le port `21/tcp` a été identifié comme ouvert.

Le service détecté est :

- Port : `21/tcp`
- Service : FTP
- Logiciel : `vsftpd`
- Version : `2.3.4`

Commande utilisée :

```bash
nmap -sV 192.168.56.101

### 🧠 Pourquoi on fait ça ?

On reproduit exactement la démarche d'un pentester :

**Nmap nous donne :**

```text
21/tcp → FTP → vsftpd 2.3.4
vsftpd 2.3.4
       ↓
CVE-2011-2523
       ↓
CVSS 9.8 — Critical
### 🐛 Vulnérabilité identifiée

La version `vsftpd 2.3.4` est associée à la vulnérabilité **CVE-2011-2523**.

Cette vulnérabilité concerne certaines copies de `vsftpd 2.3.4` distribuées entre le 30 juin et le 3 juillet 2011. Ces copies contenaient une backdoor capable d'ouvrir un shell distant sur le port `6200/tcp`.

- **CVE :** CVE-2011-2523
- **Type :** Backdoor / exécution de commandes à distance
- **Port associé :** `6200/tcp`
- **CVSS v3.1 :** `9.8 — Critical`

### ⚠️ Interprétation

La détection de la version `vsftpd 2.3.4` ne suffit pas à confirmer que la backdoor est présente.

Une vérification supplémentaire du port `6200/tcp` permet donc de rechercher un indice d'exposition associé à cette vulnérabilité.
### 🧪 Vérification technique

La documentation de la vulnérabilité indique que la backdoor associée à CVE-2011-2523 ouvre un shell sur le port `6200/tcp`.

Une vérification ciblée a donc été réalisée :

```bash
nmap -p 6200 192.168.56.101
6200/tcp closed
---

### ⚠️ Évaluation du risque

La vulnérabilité CVE-2011-2523 possède un score **CVSS v3.1 de 9.8 (Critical)**.

Cependant, le niveau de risque observé sur cette cible doit être interprété avec prudence :

- `vsftpd 2.3.4` a été détecté ;
- CVE-2011-2523 correspond à cette version dans un contexte précis ;
- le port `6200/tcp` est fermé ;
- la présence de la backdoor n'a donc pas été confirmée.

Le résultat est donc considéré comme une **vulnérabilité potentielle nécessitant une investigation complémentaire**, et non comme une vulnérabilité confirmée.

### 🛡️ Recommandations

Pour réduire le risque associé à ce service :

1. Mettre à jour ou remplacer la version vulnérable de `vsftpd`.
2. Utiliser une version maintenue et provenant d'une source fiable.
3. Désactiver FTP lorsqu'il n'est pas nécessaire.
4. Préférer un protocole sécurisé comme SFTP lorsque cela est possible.
5. Restreindre l'accès au service FTP à des réseaux ou utilisateurs autorisés.
6. Surveiller les connexions et événements associés au service.

### 📌 Conclusion

L'analyse a permis d'identifier `vsftpd 2.3.4` comme une technologie présentant un intérêt particulier du point de vue de la sécurité.

La recherche de CVE a identifié **CVE-2011-2523**, classée **Critical (CVSS 9.8)**. La vérification du port `6200/tcp` n'a toutefois pas permis de confirmer la présence de la backdoor.

Cette analyse montre qu'une détection de version ou une correspondance CVE ne suffit pas à elle seule pour confirmer une vulnérabilité : une validation adaptée au contexte est nécessaire.
---

## 2. Analyse du service HTTP — Apache

### 🔍 Identification

Lors de la reconnaissance, le port `80/tcp` a été identifié comme ouvert.

Le service détecté est :

- Port : `80/tcp`
- Service : HTTP
- Serveur : Apache
- Version : `2.2.8`
- Système détecté : Ubuntu

Commande utilisée :

```bash
nmap -sV 192.168.56.101
curl -I http://192.168.56.101/server-status
HTTP/1.1 403 Forbidden
Server: Apache/2.2.8 (Ubuntu) DAV/2
curl -I http://192.168.56.101/
HTTP/1.1 200 OK
Server: Apache/2.2.8 (Ubuntu) DAV/2
X-Powered-By: PHP/5.2.4-2ubuntu5.10
Content-Type: text/html

### 🧠 Pourquoi ce deuxième cas est intéressant ?

Regarde la différence avec FTP :

**FTP :**

```text
Version
 ↓
CVE
 ↓
Comportement attendu
 ↓
Vérification
---

## 3. Synthèse de l'analyse

L'analyse des services exposés a permis d'identifier plusieurs technologies anciennes et potentiellement sensibles sur la machine Metasploitable.

### Résultats principaux

| Service | Version | Analyse | Statut |
|---|---|---|---|
| FTP | vsftpd 2.3.4 | CVE-2011-2523 | Non confirmée |
| HTTP | Apache 2.2.8 | Analyse de configuration | Exposition partielle observée |
| PHP | 5.2.4 | Version révélée dans les headers | À analyser |
| WebDAV | DAV/2 | Fonctionnalité exposée | À analyser |

### 🧠 Méthodologie retenue

L'analyse d'une vulnérabilité ne doit pas se limiter à une correspondance entre une version logicielle et une CVE.

La démarche suivie est :

```text
Service identifié
      ↓
Version détectée
      ↓
Recherche de vulnérabilités
      ↓
Analyse des conditions nécessaires
      ↓
Vérification technique adaptée
      ↓
Évaluation du risque
      ↓
Recommandations
