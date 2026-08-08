

# 🔎 Reconnaissance réseau

## 🎯 Objectif

Réaliser une reconnaissance réseau sur une machine de laboratoire afin d'identifier les ports ouverts, les services exposés et leurs versions, puis analyser certaines pistes de vulnérabilités.

## 🧪 Environnement

- Kali Linux — `192.168.56.104`
- Metasploitable — `192.168.56.101`
- VirtualBox
- Réseau de laboratoire : `192.168.56.0/24`

La cible est une machine Metasploitable volontairement vulnérable utilisée uniquement dans un environnement de laboratoire.

---

## 1. Vérification de la connectivité

Avant toute reconnaissance, la communication entre Kali et la cible a été vérifiée.

```bash
ping -c 4 192.168.56.101
Résultat
4 paquets envoyés
4 paquets reçus
0 % de perte

La connectivité entre les deux machines est donc fonctionnelle.

2. Reconnaissance des ports

Un premier scan Nmap a été réalisé afin d'identifier les ports TCP ouverts.

nmap 192.168.56.101
Principaux services identifiés
Port	Service
21/tcp	FTP
22/tcp	SSH
23/tcp	Telnet
25/tcp	SMTP
53/tcp	DNS
80/tcp	HTTP
139/tcp	NetBIOS/Samba
445/tcp	SMB/Samba
2049/tcp	NFS
3306/tcp	MySQL
5432/tcp	PostgreSQL
5900/tcp	VNC
8009/tcp	AJP
8180/tcp	HTTP/Tomcat

La cible expose donc un nombre important de services réseau.

3. Détection des versions

Une détection de versions a ensuite été effectuée afin d'obtenir davantage d'informations sur les services.

nmap -sV 192.168.56.101
Exemples de versions identifiées
Port	Service	Version détectée
21	FTP	vsftpd 2.3.4
22	SSH	OpenSSH 4.7p1
80	HTTP	Apache 2.2.8
139/445	Samba	Samba 3.X - 4.X
3306	MySQL	MySQL 5.0.51a
5432	PostgreSQL	PostgreSQL 8.3.x
8180	HTTP	Apache Tomcat/Coyote

La détection de versions permet d'identifier les logiciels potentiellement concernés par des vulnérabilités connues.

4. Analyse du service FTP

Le port 21 expose :

21/tcp open ftp vsftpd 2.3.4

La version vsftpd 2.3.4 est associée à la vulnérabilité CVE-2011-2523.

Cette vulnérabilité concerne une version compromise de vsftpd 2.3.4 distribuée pendant une période spécifique en 2011.

Une vérification du port TCP 6200 a été réalisée afin de rechercher le comportement associé à cette vulnérabilité :

nmap -p 6200 192.168.56.101
Résultat
6200/tcp closed
Conclusion

La présence de vsftpd 2.3.4 constitue une piste de vulnérabilité, mais la vérification effectuée ne permet pas de confirmer la présence de la backdoor associée à CVE-2011-2523.

Cette analyse illustre l'importance de ne pas considérer automatiquement une version détectée comme une vulnérabilité confirmée.

5. Analyse du serveur Web

Le port 80 expose :

80/tcp open http Apache httpd 2.2.8

Une vérification de la ressource /server-status a été effectuée :

curl -I http://192.168.56.101/server-status
Résultat
HTTP/1.1 403 Forbidden
Server: Apache/2.2.8 (Ubuntu) DAV/2

L'accès à cette ressource est refusé. La présence d'une page server-status accessible publiquement n'a donc pas été démontrée.

6. Analyse des en-têtes HTTP

Les en-têtes de la page principale ont ensuite été examinés :

curl -I http://192.168.56.101/
Résultat
HTTP/1.1 200 OK
Server: Apache/2.2.8 (Ubuntu) DAV/2
X-Powered-By: PHP/5.2.4-2ubuntu5.10
Content-Type: text/html

Plusieurs informations techniques sont exposées :

Apache 2.2.8
PHP 5.2.4
WebDAV (DAV/2)
Type de contenu retourné

L'en-tête X-Powered-By révèle notamment la version de PHP utilisée par le serveur.

Cette information peut faciliter l'identification de technologies potentiellement vulnérables et constitue une information utile lors d'une phase de reconnaissance.

7. Méthodologie appliquée

La démarche suivie durant ce laboratoire est :

Reconnaissance
      ↓
Identification des ports
      ↓
Identification des services
      ↓
Détection des versions
      ↓
Recherche de vulnérabilités potentielles
      ↓
Validation adaptée au contexte
      ↓
Analyse des résultats
      ↓
Conclusion

Une version logicielle ancienne ou un service exposé ne suffit pas à lui seul à confirmer une vulnérabilité.

📚 Compétences développées
Reconnaissance réseau
Utilisation de Nmap
Identification des ports et services
Détection de versions
Analyse des services exposés
Recherche de vulnérabilités
Compréhension des CVE
Validation d'une hypothèse de vulnérabilité
Analyse des en-têtes HTTP
Documentation technique
⚠️ Cadre du laboratoire

Toutes les manipulations ont été réalisées sur une machine Metasploitable dédiée à l'apprentissage de la cybersécurité, dans un environnement de laboratoire contrôlé et autorisé.
