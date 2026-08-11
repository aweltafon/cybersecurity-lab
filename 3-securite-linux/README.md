# Lab 03 — Sécurité Linux

## 1. Objectif

Ce laboratoire a pour objectif d'effectuer un premier audit de sécurité d'un système Linux afin d'identifier sa configuration, ses utilisateurs et privilèges, ses permissions, ses processus, ses services, ses ports réseau ainsi que ses mécanismes de journalisation.

L'analyse est réalisée sur une machine virtuelle Kali Linux dans un environnement de laboratoire VirtualBox.

Les vérifications effectuées permettent également d'identifier les éléments pouvant représenter une surface d'attaque et de formuler des recommandations de hardening adaptées.

## 2. Environnement

* **Système :** Kali Linux
* **Environnement :** Machine virtuelle
* **Hyperviseur :** Oracle VirtualBox
* **Type d'analyse :** Audit local de sécurité Linux
* **Objectif :** Identification des configurations, services et éléments de sécurité du système
## 3. Identification du système

La première étape de l'audit consiste à identifier le système d'exploitation et son environnement.

### Commandes utilisées

```bash
cat /etc/os-release
```

```bash
uname -a
```

Ces commandes permettent d'obtenir des informations sur la distribution Linux, sa version, le noyau utilisé et l'architecture du système.

### Analyse

L'environnement analysé est une machine virtuelle Kali Linux utilisée comme environnement de laboratoire de cybersécurité.

L'identification du système constitue une étape importante d'un audit car elle permet de connaître l'environnement technique avant d'effectuer les vérifications de sécurité.

---

## 4. Utilisateurs et groupes

L'objectif de cette étape est d'identifier les utilisateurs présents sur le système ainsi que les groupes auxquels ils appartiennent.

### Commandes utilisées

```bash
whoami
```

```bash
id
```

```bash
cat /etc/passwd
```

### Analyse

La commande `whoami` permet d'identifier l'utilisateur actuellement connecté.

La commande `id` permet d'obtenir l'identifiant de l'utilisateur ainsi que les groupes auxquels il appartient.

Le fichier `/etc/passwd` contient les informations relatives aux comptes utilisateurs du système. Son analyse permet notamment d'identifier les comptes présents et leurs paramètres associés.

---

## 5. Vérification des privilèges

L'analyse des privilèges permet de déterminer les commandes que l'utilisateur courant peut exécuter avec des privilèges élevés.

### Commande utilisée

```bash
sudo -l
```

### Analyse

La commande `sudo -l` permet de connaître les privilèges `sudo` accordés à l'utilisateur courant.

Cette vérification est importante dans un audit de sécurité car des permissions `sudo` trop larges peuvent permettre une élévation de privilèges en cas de compromission d'un compte.

Le principe du moindre privilège doit être appliqué : un utilisateur ne devrait disposer que des privilèges nécessaires à ses tâches.

---

## 6. Analyse des fichiers sensibles

Une attention particulière est portée aux fichiers contenant des informations relatives aux comptes et à l'authentification.

### Fichiers étudiés

```bash
/etc/passwd
```

```bash
/etc/shadow
```

### Analyse

Le fichier `/etc/passwd` contient les informations générales relatives aux comptes utilisateurs.

Le fichier `/etc/shadow` contient les informations sensibles liées aux mots de passe et possède des permissions plus restrictives.

L'accès à ces fichiers doit être correctement contrôlé afin d'empêcher la divulgation d'informations sensibles.

---

## 7. Vérification des permissions

L'analyse des permissions permet d'identifier les fichiers ou répertoires dont les droits d'accès pourraient représenter un risque.

### Vérification effectuée

Une recherche de fichiers accessibles en écriture par tous les utilisateurs a été réalisée afin d'identifier les ressources potentiellement exposées.

### Analyse

Les permissions Linux reposent notamment sur trois catégories :

* propriétaire ;
* groupe ;
* autres utilisateurs.

Une mauvaise configuration des permissions peut permettre à un utilisateur non privilégié de modifier des fichiers qui devraient être protégés.

Le principe du moindre privilège doit également être appliqué aux permissions des fichiers.

---

## 8. Analyse des processus et des sockets réseau

L'analyse des processus permet d'identifier les programmes actuellement exécutés sur le système.

L'analyse des sockets permet ensuite d'identifier les services réseau susceptibles d'être exposés.

### Commandes utilisées

```bash
ps aux
```

```bash
ss -tuln
```

### Analyse

La combinaison de l'analyse des processus et des sockets permet d'établir une relation entre les applications exécutées et les services réseau exposés.

Un port ouvert n'est pas automatiquement une vulnérabilité. Il est nécessaire d'identifier le service associé, de déterminer s'il est nécessaire et d'évaluer son niveau d'exposition.

Cette démarche permet de réduire la surface d'attaque du système.
## 9. Analyse des services systemd

L'analyse des services permet d'identifier les composants actuellement actifs ainsi que ceux configurés pour démarrer automatiquement.

### 9.1 Services actuellement actifs

```bash
systemctl --type=service --state=running
```

La commande a permis d'identifier **20 services actifs**.

Parmi les services observés :

* `cron` : exécution de tâches planifiées ;
* `NetworkManager` : gestion des connexions réseau ;
* `lightdm` : gestion de l'environnement graphique ;
* `polkit` : gestion des autorisations ;
* `systemd-journald` : gestion des journaux système ;
* `systemd-logind` : gestion des sessions utilisateurs ;
* `udisks2` : gestion des périphériques de stockage ;
* `virtualbox-guest-utils` : intégration avec VirtualBox.

### Analyse

Un service actif n'est pas automatiquement une vulnérabilité. Il est nécessaire d'identifier son rôle et de vérifier s'il est réellement nécessaire.

L'objectif d'un audit est notamment de réduire les services inutiles afin de limiter la surface d'attaque.

### 9.2 Services activés au démarrage

```bash
systemctl list-unit-files --type=service --state=enabled
```

La commande a identifié **18 services configurés comme `enabled`**.

Parmi eux figurent notamment :

* `cron.service`
* `NetworkManager.service`
* `networking.service`
* `ModemManager.service`
* `smartmontools.service`
* `systemd-timesyncd.service`
* `virtualbox-guest-utils.service`

L'état `enabled` indique qu'un service est configuré pour être activé automatiquement selon le fonctionnement de systemd.

### 9.3 Vérification des services en échec

```bash
systemctl --failed
```

Résultat :

```text
0 loaded units listed.
```

Aucun service systemd n'a été signalé en état `failed` au moment de l'analyse.

Cela indique qu'aucun service n'était actuellement identifié comme étant en échec par systemd.

---

## 10. Analyse du service SSH

SSH constitue un élément important à vérifier lors d'un audit car il peut fournir un accès distant au système.

### Vérification de l'état du service

```bash
systemctl status ssh
```

Résultat observé :

```text
Loaded: loaded (...; disabled; ...)
Active: inactive (dead)
```

Le service OpenSSH est installé mais n'est actuellement pas actif et n'est pas configuré pour démarrer automatiquement.

### Vérification du port SSH

```bash
ss -tuln | grep :22
```

Aucun résultat n'a été retourné.

Cela indique qu'aucun service n'était en écoute sur le port TCP 22 au moment de la vérification.

### Analyse sécurité

Dans l'environnement actuel, SSH n'est donc pas exposé sur le réseau.

Cette configuration limite la surface d'attaque liée à l'accès distant.

Si SSH devait être activé ultérieurement, il serait recommandé d'utiliser une authentification forte, de limiter les utilisateurs autorisés et d'éviter l'accès direct du compte root.

---

## 11. Analyse des journaux système

La journalisation est essentielle pour détecter et analyser les événements système et de sécurité.

### Commande utilisée

```bash
journalctl -p warning -b
```

Cette commande permet d'afficher les événements de niveau `warning` et supérieur depuis le dernier démarrage du système.

### Observations

Plusieurs avertissements ont été observés, notamment concernant :

* la couche graphique et la virtualisation (`vmwgfx`) ;
* certains composants ACPI et matériels virtuels ;
* des composants systemd ;
* `haveged` ;
* `wireplumber` ;
* `systemd-journald`.

Ces messages ne sont pas considérés automatiquement comme des vulnérabilités. Une analyse complémentaire serait nécessaire pour déterminer leur cause et leur impact.

### Analyse sécurité

La présence de `systemd-journald` permet la collecte des événements système.

La surveillance des journaux constitue un élément essentiel de la détection d'incidents, notamment pour identifier des erreurs répétées, des tentatives d'authentification anormales ou des comportements inhabituels.

---

## 12. Vérification des mises à jour

La maintenance du système constitue une mesure importante de sécurité.

### Commande utilisée

```bash
sudo apt update
```

La commande a permis de synchroniser les informations disponibles depuis les dépôts Kali.

Résultat observé :

```text
Fetched 76.0 MB
1853 packages can be upgraded.
```

### Analyse

Le système indique que **1 853 paquets peuvent être mis à niveau**.

Ce nombre ne correspond pas directement au nombre de vulnérabilités présentes. `apt update` actualise uniquement les informations concernant les paquets disponibles ; il n'effectue pas leur mise à jour.

Une politique de mise à jour régulière doit néanmoins être appliquée afin de réduire l'exposition aux vulnérabilités connues.

---

## 13. Recommandations de sécurité

À partir des observations réalisées durant l'audit, plusieurs recommandations peuvent être formulées.

### Maintenir le système à jour

Effectuer régulièrement les mises à jour des paquets après vérification des changements disponibles.

### Réduire les services inutiles

Chaque service non nécessaire peut augmenter la surface d'attaque. Les services doivent être régulièrement identifiés et désactivés lorsqu'ils ne sont pas nécessaires.

### Limiter l'exposition réseau

Les ports ouverts doivent correspondre à des services nécessaires. Chaque service réseau doit être identifié et son exposition évaluée.

### Sécuriser SSH

SSH doit rester désactivé lorsqu'aucun accès distant n'est nécessaire. Lorsqu'il est utilisé, une configuration renforcée et une authentification forte doivent être privilégiées.

### Surveiller les journaux

Les journaux système doivent être régulièrement analysés afin d'identifier les erreurs, comportements inhabituels et événements de sécurité.

### Appliquer le principe du moindre privilège

Les utilisateurs, groupes, services et fichiers doivent disposer uniquement des permissions nécessaires à leur fonctionnement.

---

## 14. Conclusion

Cet audit a permis d'effectuer une première analyse de sécurité d'un système Kali Linux.

Les principales vérifications ont porté sur :

* l'identification du système ;
* les utilisateurs et groupes ;
* les privilèges `sudo` ;
* les fichiers sensibles ;
* les permissions ;
* les processus ;
* les sockets réseau ;
* les services systemd ;
* le service SSH ;
* les journaux système ;
* l'état des mises à jour.

L'analyse a notamment montré qu'aucun service systemd n'était en échec et que le service SSH était installé mais inactif et non exposé sur le port 22.

Le laboratoire met en évidence une démarche d'audit basée sur l'observation, la vérification et l'interprétation des résultats plutôt que sur la simple exécution de commandes.

Les recommandations formulées visent principalement à réduire la surface d'attaque, maintenir le système à jour, limiter les privilèges et améliorer la surveillance des événements système.
