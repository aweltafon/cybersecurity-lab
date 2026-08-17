# Jour 3 — Administration Windows & Analyse des événements

## 🎯 Objectifs

L'objectif de ce laboratoire était de découvrir les bases de l'administration Windows et de commencer à analyser les journaux de sécurité Windows dans une logique d'analyse SOC.

Les objectifs étaient de :

* Comprendre le fonctionnement des services Windows.
* Identifier l'état et le type de démarrage d'un service.
* Utiliser l'Observateur d'événements Windows.
* Analyser des événements d'authentification.
* Comprendre les événements **4624** et **4625**.
* Identifier les différents types d'ouverture de session.
* Analyser une tentative d'authentification échouée.
* Comprendre les premiers indicateurs d'un éventuel **password spraying**.

---

## 1. Administration des services Windows

L'outil `services.msc` permet d'administrer et d'observer les services Windows.

Un service Windows est un programme qui fonctionne en arrière-plan afin d'assurer une fonction spécifique du système.

### Windows Update

Observation réalisée :

* **État :** Arrêté
* **Type de démarrage :** Manuel

Un service configuré en **manuel** peut être démarré lorsque le système ou une application en a besoin.

### Pare-feu Microsoft Defender

Observation réalisée :

* **État :** En cours d'exécution
* **Type de démarrage :** Automatique

Le service du pare-feu est donc configuré pour fonctionner automatiquement et assurer la protection réseau du système.

### Intérêt sécurité

L'analyse des services permet notamment de vérifier :

* les services actifs ;
* les services arrêtés ;
* leur type de démarrage ;
* les changements de configuration ;
* la présence éventuelle d'un service suspect.

Un attaquant peut notamment chercher à exploiter ou modifier certains services afin d'exécuter du code ou de maintenir une persistance sur une machine.

---

## 2. Observateur d'événements Windows

L'Observateur d'événements (`eventvwr.msc`) permet de consulter les journaux générés par Windows.

Pour cette séance, le journal étudié était :

**Journaux Windows → Sécurité**

Ce journal contient notamment des événements liés aux authentifications et aux accès au système.

---

## 3. Analyse d'un événement 4624

### Event ID 4624

L'événement **4624** correspond à une **ouverture de session réussie**.

Un événement 4624 doit cependant être analysé dans son contexte.

Il est notamment possible d'étudier :

* le compte utilisé ;
* l'heure de connexion ;
* le type d'ouverture de session ;
* la machine concernée ;
* l'adresse source ;
* le processus ou mécanisme d'authentification.

### Exemple observé

L'événement analysé contenait :

* **Compte :** `SRV-AD-01$`
* **Domaine :** `WORKGROUP`
* **Logon Type :** `5`
* **Jeton élevé :** Oui
* **ID de session :** `0x3E7`

Le caractère `$` du compte indique qu'il s'agit d'un **compte machine**.

Le **Logon Type 5** correspond à une ouverture de session effectuée par un **service**.

L'événement était donc cohérent avec une authentification effectuée dans le contexte d'un service Windows plutôt qu'avec une connexion interactive d'un utilisateur.

---

## 4. Analyse d'un événement 4625

### Event ID 4625

L'événement **4625** correspond à une **ouverture de session échouée**.

Un seul événement 4625 ne suffit pas à conclure à une attaque.

Il faut analyser le contexte, notamment :

* le nombre de tentatives ;
* leur fréquence ;
* les comptes ciblés ;
* l'adresse IP source ;
* le type d'ouverture de session ;
* les codes d'état et de sous-état ;
* le mécanisme d'authentification.

### Événement observé

L'événement étudié présentait :

* **Logon Type :** `3`
* **Adresse réseau source :** `127.0.0.1`
* **Nom de la station :** `SRV-AD-01`
* **Port source :** `54670`
* **État :** `0xC000006D`
* **Sous-état :** `0xC0000064`
* **Processus d'authentification :** `NtLmSsp`
* **Package d'authentification :** `NTLM`

### Interprétation

Le **Logon Type 3** correspond à une tentative d'ouverture de session via le réseau.

L'adresse `127.0.0.1` correspond à l'adresse **loopback**, ce qui indique que la communication provenait de la machine locale.

Le sous-état **`0xC0000064`** indique que le compte fourni n'a pas été reconnu.

Le code **`0xC000006D`** correspond à un échec d'authentification lié aux informations d'identification fournies.

Le mécanisme d'authentification observé était **NTLM**.

L'événement ne permettait pas d'identifier directement le processus à l'origine de la tentative, car le champ du processus appelant était vide.

---

## 5. Différence entre 4624 et 4625

| Event ID | Signification                |
| -------- | ---------------------------- |
| **4624** | Ouverture de session réussie |
| **4625** | Ouverture de session échouée |

L'analyse ne doit cependant pas se limiter à l'ID de l'événement.

Un analyste SOC doit rechercher le contexte autour de l'événement.

---

## 6. Logon Types étudiés

| Logon Type | Signification                       |
| ---------: | ----------------------------------- |
|      **3** | Ouverture de session via le réseau  |
|      **5** | Ouverture de session par un service |

Ces informations permettent de comprendre la manière dont l'authentification a été effectuée.

---

## 7. Mise en situation SOC — Password Spraying

Un scénario a ensuite été étudié afin de comprendre comment identifier un comportement potentiellement malveillant.

Exemple :

```text
50 événements 4625
environ 2 minutes
source : 192.168.1.50

Comptes ciblés :
- admin
- compta
- rh
- direction
- support
```

Ce comportement est beaucoup plus suspect qu'un événement 4625 isolé.

La combinaison :

* d'un nombre important d'échecs ;
* d'une fréquence élevée ;
* d'une même source ;
* de plusieurs comptes ciblés ;

peut indiquer une tentative de **password spraying**.

### Password spraying

Le password spraying consiste à essayer un même mot de passe ou un petit nombre de mots de passe sur plusieurs comptes afin d'éviter de déclencher les protections liées aux nombreuses tentatives sur un seul compte.

À l'inverse, une attaque **brute force** vise généralement un compte avec un grand nombre de mots de passe différents.

---

## 8. Raisonnement SOC

Lorsqu'un événement 4625 est détecté, il ne faut pas immédiatement conclure à une attaque.

Une analyse doit notamment rechercher :

```text
Événement
    ↓
Source
    ↓
Compte ciblé
    ↓
Fréquence
    ↓
Nombre de tentatives
    ↓
Type de connexion
    ↓
Code d'échec
    ↓
Contexte
    ↓
Qualification de l'incident
```

Un événement isolé peut être lié à une erreur utilisateur ou à une mauvaise configuration.

En revanche, plusieurs échecs rapprochés provenant d'une même source et ciblant plusieurs comptes peuvent constituer un indicateur de compromission ou de tentative d'attaque.

---

## 9. Compétences acquises

À la fin de ce laboratoire, je suis capable de :

* Utiliser `services.msc`.
* Identifier l'état d'un service Windows.
* Identifier son type de démarrage.
* Utiliser l'Observateur d'événements.
* Accéder au journal **Sécurité**.
* Identifier un événement **4624**.
* Identifier un événement **4625**.
* Interpréter les **Logon Types 3 et 5**.
* Identifier une adresse loopback `127.0.0.1`.
* Interpréter les codes `0xC000006D` et `0xC0000064`.
* Identifier l'utilisation de **NTLM** dans un événement d'authentification.
* Différencier une anomalie isolée d'un comportement nécessitant une investigation.
* Reconnaître les principaux indicateurs d'un éventuel **password spraying**.

---

## 🎤 Défense orale

> « J'ai utilisé l'Observateur d'événements Windows pour analyser des événements d'authentification 4624 et 4625. J'ai appris à interpréter le type d'ouverture de session, l'adresse source et les codes d'échec. J'ai également étudié comment plusieurs échecs d'authentification rapides ciblant plusieurs comptes peuvent constituer un indicateur de password spraying. »

---

## 📌 Conclusion

Ce laboratoire m'a permis de relier l'administration Windows à la cybersécurité.

L'analyse des journaux Windows constitue une compétence importante pour un analyste SOC, car elle permet de détecter et d'investiguer des comportements anormaux liés aux authentifications et aux accès au système.

Le prochain objectif sera d'approfondir progressivement l'analyse des logs et leur exploitation dans une démarche de supervision de sécurité.
