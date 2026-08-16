# Lab 05 — Fondamentaux des réseaux et diagnostic

## 🎯 Objectif

Renforcer les fondamentaux réseaux indispensables à l'administration systèmes, réseaux et cybersécurité, puis apprendre à raisonner méthodiquement face à un problème de connectivité ou de résolution de noms.

L'objectif est également de pouvoir expliquer ces notions et les défendre lors d'un entretien technique.

---

## 🧠 1. Adressage IPv4

Une adresse IPv4 permet d'identifier une interface réseau sur un réseau IP.

Exemple :

```text
192.168.56.10
```

Une adresse IPv4 est composée de quatre octets, chacun pouvant prendre une valeur de 0 à 255.

### Exemple de sous-réseau

```text
Adresse IP : 192.168.1.10
Masque     : 255.255.255.0
```

Le réseau correspondant est :

```text
192.168.1.0/24
```

Le masque permet de déterminer la partie réseau et la partie hôte de l'adresse.

---

## 🌐 2. Même réseau et réseau différent

Deux machines avec :

```text
192.168.1.10/24
192.168.1.20/24
```

appartiennent au même sous-réseau :

```text
192.168.1.0/24
```

Elles peuvent donc communiquer directement au niveau IP, sous réserve des autres paramètres réseau.

À l'inverse :

```text
192.168.1.10/24
192.168.2.20/24
```

appartiennent à deux réseaux différents.

Le trafic destiné à l'autre réseau doit alors être envoyé vers la passerelle par défaut.

---

## 🚪 3. Passerelle par défaut

La passerelle permet à une machine d'atteindre des réseaux qui ne sont pas directement accessibles depuis son réseau local.

Exemple :

```text
PC-A
IP          : 192.168.1.10
Masque      : 255.255.255.0
Passerelle  : 192.168.1.1
```

Pour joindre :

```text
192.168.2.20
```

le poste utilise sa passerelle :

```text
PC-A
  │
  ▼
192.168.1.1
  │
  ▼
192.168.2.20
```

La décision de savoir si une destination est locale ou distante repose notamment sur la table de routage.

---

## 🌍 4. DNS

Le DNS (Domain Name System) est un service de résolution de noms.

Il permet notamment d'associer un nom à une adresse IP ou de localiser certains services réseau.

Exemple :

```text
srv-ad-01.pme.local → 192.168.56.10
```

### Vérification de la configuration DNS

```powershell
ipconfig /all
```

Cette commande permet notamment d'identifier les serveurs DNS configurés sur la machine.

### Test de résolution

```powershell
nslookup srv-ad-01.pme.local
```

---

## 📚 5. Principaux enregistrements DNS

| Type  | Fonction                  |
| ----- | ------------------------- |
| A     | Nom → adresse IPv4        |
| AAAA  | Nom → adresse IPv6        |
| CNAME | Alias → autre nom         |
| MX    | Serveur de messagerie     |
| SRV   | Localisation d'un service |

### Exemple A

```powershell
nslookup -type=A www.google.com
```

Recherche d'un enregistrement A correspondant à une adresse IPv4.

### Exemple SRV

```powershell
nslookup -type=SRV _ldap._tcp.pme.local
```

Cette requête permet de rechercher un enregistrement SRV permettant de localiser le service LDAP du domaine `pme.local`.

Un résultat SRV peut notamment indiquer :

```text
port = 389
svr hostname = SRV-AD-01.pme.local
```

Le port TCP 389 correspond au service LDAP.

---

## 🔐 6. DNS et Active Directory

DNS est particulièrement important dans un environnement Active Directory.

Il permet notamment aux machines de résoudre les noms et de localiser les services du domaine grâce aux enregistrements DNS, notamment les enregistrements SRV.

Exemple :

```text
_ldap._tcp.pme.local
```

Cette entrée permet de rechercher le service LDAP du domaine.

Une mauvaise résolution DNS peut donc perturber la localisation des services Active Directory.

Cette problématique a également été rencontrée et diagnostiquée lors du Lab 04.

---

## 🔄 7. TCP et UDP

### TCP

TCP est un protocole de transport orienté connexion qui fournit des mécanismes de fiabilité et d'ordre des données.

Il est adapté lorsqu'une transmission fiable est nécessaire.

Exemple :

```text
Transfert d'un fichier important
→ TCP
```

### UDP

UDP est un protocole de transport sans connexion et plus léger.

Il ne fournit pas les mêmes garanties de livraison et d'ordre que TCP.

Le choix entre TCP et UDP dépend des besoins de l'application.

---

## 🔌 8. Ports et services

L'adresse IP permet d'identifier une machine tandis que le port permet d'identifier le service réseau auquel on souhaite accéder.

Exemple :

```text
192.168.1.50:443
```

* `192.168.1.50` → adresse IP de la machine
* `443` → port généralement utilisé par HTTPS

Quelques exemples étudiés :

```text
22   → SSH
53   → DNS
80   → HTTP
443  → HTTPS
389  → LDAP
```

Un port ouvert ne signifie pas automatiquement qu'une vulnérabilité existe. Une analyse complémentaire est nécessaire pour identifier le service, sa version et les éventuelles vulnérabilités.

---

## 🔎 9. ARP

ARP (Address Resolution Protocol) permet de retrouver l'adresse MAC associée à une adresse IPv4 sur le réseau local.

Exemple :

```text
PC-A
192.168.1.10
     │
     │ ARP Request
     ▼
« Qui possède 192.168.1.20 ? »
     │
     ▼
PC-B
192.168.1.20
MAC : XX:XX:XX:XX:XX:XX
```

La correspondance IPv4/MAC permet ensuite de construire la trame Ethernet destinée à l'équipement concerné.

### Commandes utilisées pour observer les informations ARP

Linux :

```bash
ip neigh
```

Windows :

```powershell
arp -a
```

---

## 📡 10. ICMP et ping

La commande `ping` permet notamment de tester la joignabilité IP d'une destination et d'obtenir une indication sur le temps de réponse.

Exemple :

```bash
ping 192.168.1.20
```

Elle utilise le protocole ICMP.

Schéma simplifié :

```text
PC-A
  │
  │ ICMP Echo Request
  ▼
PC-B
  │
  │ ICMP Echo Reply
  ▼
PC-A
```

Un échec du ping ne signifie cependant pas automatiquement que la machine est indisponible : ICMP peut être bloqué par un pare-feu ou le problème peut venir d'un autre élément du réseau.

---

## 🛠️ 11. Méthodologie de diagnostic DNS

Scénario :

> Un utilisateur peut accéder à un serveur avec son adresse IP mais pas avec son nom.

Exemple :

```text
192.168.56.10       → fonctionne
srv-ad-01.pme.local → ne fonctionne pas
```

La première hypothèse est un problème de résolution DNS.

### Étape 1 — vérifier la configuration DNS

```powershell
ipconfig /all
```

### Étape 2 — tester la résolution

```powershell
nslookup srv-ad-01.pme.local
```

### Étape 3 — analyser le résultat

`Non-existent domain` indique que le serveur DNS interrogé répond mais ne trouve pas le nom demandé.

`No response from server` indique que le serveur DNS interrogé ne fournit pas de réponse à la requête.

Le diagnostic doit ensuite être poursuivi selon le résultat obtenu.

---

## 🎤 12. Préparation aux entretiens techniques

Les notions étudiées ont également été travaillées sous forme de questions d'entretien.

### Exemple

**Question :** Un serveur est accessible par son adresse IP mais pas par son nom. Que faites-vous ?

**Réponse travaillée :**

> Comme le serveur est accessible par son adresse IP mais pas par son nom, je suspecte d'abord un problème de résolution DNS. Je vérifie la configuration DNS du poste avec `ipconfig /all`, puis je teste la résolution du nom avec `nslookup`. Selon le résultat, je poursuis le diagnostic au niveau du serveur DNS ou de l'enregistrement concerné.

### Autre exemple

**Question :** Un port TCP 22 est ouvert. Est-ce une vulnérabilité ?

**Réponse travaillée :**

> Le port TCP 22 ouvert indique généralement qu'un service SSH est accessible. Cependant, un port ouvert ne signifie pas automatiquement qu'il existe une vulnérabilité. Il faut identifier précisément le service et sa version puis effectuer des vérifications complémentaires.

---

## ✅ Compétences travaillées

* Adressage IPv4
* Masques et sous-réseaux
* Passerelle par défaut
* Principes du routage
* DNS
* Enregistrements A, AAAA, CNAME, MX et SRV
* DNS et Active Directory
* TCP / UDP
* Ports et services
* ARP
* ICMP
* Diagnostic réseau
* Diagnostic DNS
* Utilisation de `ipconfig`
* Utilisation de `nslookup`
* Raisonnement technique en entretien

---

## 📌 Bilan du Lab

Ce laboratoire a permis de renforcer les fondamentaux nécessaires à l'administration systèmes, réseaux et cybersécurité.

L'accent a été mis sur la compréhension et le raisonnement plutôt que sur la mémorisation des commandes.

Les notions ont également été mises en situation sous forme de scénarios d'entretien afin de développer la capacité à expliquer une démarche technique de manière claire et professionnelle.

## 🚀 Suite

Le prochain jour de formation approfondira les compétences nécessaires au profil Systèmes, Réseaux et Cybersécurité, avec une progression vers des manipulations plus pratiques et des scénarios de diagnostic.
