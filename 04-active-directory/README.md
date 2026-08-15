# Lab 04 — Active Directory

## Objectif

Mettre en place et administrer une infrastructure Active Directory pour une PME dans un environnement virtualisé avec Windows Server et VirtualBox.

## Environnement

- Hyperviseur : Oracle VirtualBox
- Serveur : Windows Server
- Nom du serveur : `SRV-AD-01`
- Domaine Active Directory : `pme.local`
- Contrôleur de domaine : `SRV-AD-01.pme.local`

## Travaux réalisés

- Préparation du serveur Windows Server
- Configuration du nom du serveur
- Configuration réseau et adressage IP
- Installation et configuration d'Active Directory Domain Services (AD DS)
- Promotion du serveur en contrôleur de domaine
- Création du domaine `pme.local`
- Création et gestion d'un utilisateur Active Directory
- Création et application d'une stratégie de groupe (GPO)
- Vérification de l'application des GPO avec `gpresult`
- Diagnostic du contrôleur de domaine avec `dcdiag`
- Vérification des services DNS et de la résolution de noms

## Vérifications

Les commandes suivantes ont été utilisées pour vérifier l'infrastructure :

```powershell
whoami
gpresult /r
dcdiag
dcdiag /test:dns
ipconfig /all
ipconfig /flushdns
ipconfig /registerdns
nslookup
Get-NetAdapter
Résultats

Le domaine pme.local et le contrôleur de domaine SRV-AD-01 sont fonctionnels.

Des problèmes de résolution DNS ont été identifiés lors des tests dcdiag /test:dns. Ces problèmes ont été analysés et documentés dans le cadre du laboratoire.

Compétences développées
Active Directory
Windows Server
DNS
Group Policy / GPO
Gestion des utilisateurs et groupes
Administration système
Diagnostic réseau
Virtualisation avec VirtualBox
