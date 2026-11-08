# Guide de sécurité et de confidentialité de macOS

[English](README.md)

Ce guide est un recueil de techniques pour améliorer la sécurité et la confidentialité de macOS sur les Mac [avec puce Apple](https://support.apple.com/116943). Il s'adresse aux utilisateurs expérimentés qui souhaitent appliquer des pratiques de sécurité couramment utilisées par les organisations, mais convient également aux utilisateurs novices intéressés par la confidentialité et la sécurité.

Pour les Mac gérés par une organisation, consultez le [macOS Security Compliance Project](https://github.com/usnistgov/macos_security), maintenu par le [U.S. National Institute of Standards and Technology](https://en.wikipedia.org/wiki/National_Institute_of_Standards_and_Technology).

Ce guide est fourni « tel quel », sans garantie d'aucune sorte. Vous êtes seul responsable des conséquences de son application.

- [Notions de base](#basics)
- [Modèle de menace](#threat-model)
  - [Actifs](#assets)
  - [Adversaires](#adversaries)
  - [Capacités](#capabilities)
  - [Atténuations](#mitigations)
  - [Exemple de modèle](#example-model)
- [Matériel](#hardware)
- [Installation de macOS](#installing-macos)
  - [Activation du système](#system-activation)
  - [Compte Apple](#apple-account)
  - [App Store](#app-store)
  - [Virtualisation](#virtualization)
    - [Conteneurs Apple](#apple-containers)
- [Premier démarrage](#first-boot)
- [Comptes administrateur et utilisateur](#admin-and-user-accounts)
  - [Mises en garde](#caveats)
  - [Configuration](#setup)
- [Micrologiciel](#firmware)
- [FileVault](#filevault)
- [Mode Isolement](#lockdown-mode)
- [Pare-feu](#firewall)
  - [Pare-feu de couche applicative](#application-layer-firewall)
    - [Mode furtif](#stealth-mode)
    - [Apps signées](#signed-apps)
    - [Recharger le pare-feu](#reload-firewall)
    - [Obtenir l'état](#get-state)
    - [AirDrop](#airdrop)
  - [Pare-feu tiers](#third-party-firewalls)
  - [Filtre de paquets](#packet-filter)
    - [Exemple de configuration pf](#example-pf-config)
    - [Commandes du pare-feu](#firewall-commands)
    - [Bloquer des réseaux](#block-networks)
- [Services](#services)
- [Suggestions Siri et Spotlight](#siri-suggestions-and-spotlight)
- [Homebrew](#homebrew)
- [DNS](#dns)
  - [Profils DNS](#dns-profiles)
  - [Fichier hosts](#hosts-file)
  - [DNSCrypt](#dnscrypt)
  - [Dnsmasq](#dnsmasq)
- [Autorités de certification](#certificate-authorities)
- [Privoxy](#privoxy)
- [Navigateur](#browser)
  - [Firefox](#firefox)
  - [Chrome](#chrome)
  - [Safari](#safari)
  - [Confidentialité du navigateur Web](#web-browser-privacy)
- [Tor](#tor)
- [VPN](#vpn)
  - [WireGuard](#wireguard)
- [PGP/GPG](#pgpgpg)
- [Courrier électronique](#email)
  - [Thunderbird](#thunderbird)
- [Messageries](#messengers)
  - [XMPP](#xmpp)
  - [Signal](#signal)
  - [iMessage](#imessage)
- [Logiciels malveillants](#malware)
  - [Téléchargement de logiciels](#downloading-software)
  - [Bac à sable d'app](#app-sandbox)
  - [Environnement d'exécution renforcé](#hardened-runtime)
  - [Antivirus](#antivirus)
  - [Gatekeeper](#gatekeeper)
- [Protection de l'intégrité du système](#system-integrity-protection)
- [Métadonnées et artefacts](#metadata-and-artifacts)
- [Authentification](#authentication)
- [Sauvegarde](#backup)
  - [Time Machine](#time-machine)
  - [GnuPG](#gnupg)
  - [Utilitaire de disque](#disk-utility)
  - [Autres](#other)
- [Wi-Fi](#wi-fi)
- [SSH](#ssh)
- [Accès physique](#physical-access)
- [Surveillance](#monitoring)
  - [Journaux](#logs)
  - [DTrace](#dtrace)
  - [Processus](#processes)
  - [Sécurité des points de terminaison](#endpoint-security)
  - [Historique des installations](#install-history)
  - [Réseau](#network)
    - [Wireshark](#wireshark)
- [Divers](#miscellaneous)
  - [Économiseur d'écran](#screensaver)
  - [Données de diagnostic](#diagnostic-data)
  - [Lecteur multimédia](#media-player)
  - [Gestionnaires de fichiers](#file-handlers)
  - [Options du Finder](#finder-options)
  - [umask personnalisé](#custom-umask)
  - [Saisie au clavier](#keyboard-entry)
  - [Renforcement du réseau](#network-hardening)
  - [Sudoers](#sudoers)
- [Logiciels associés](#related-software)
- [Ressources supplémentaires](#additional-resources)

---

<a name="basics"></a>

# Notions de base

Appliquez les bonnes pratiques générales de sécurité :

- Créez un [modèle de menace](#threat-model)
  - Votre adversaire est-il une personne capable d’intercepter vos communications à proximité, un criminel utilisant des logiciels malveillants courants, ou une organisation bien financée et très compétente ?
  - Définissez les menaces ou les groupes contre lesquels vous vous protégez et ce qu'ils peuvent réellement faire.

- Maintenez le système et les logiciels à jour
  - Installez régulièrement les mises à jour disponibles pour le système d'exploitation et toutes les applications.
  - Les mises à jour sont installées dans [Réglages Système](https://support.apple.com/guide/mac-help/keep-your-mac-up-to-date-mchlpx1065) ou avec l'utilitaire en ligne de commande [`softwareupdate`](https://ss64.com/mac/softwareupdate.html). Ni l'un ni l'autre ne nécessite de compte Apple.
  - Abonnez-vous à la liste de diffusion [Apple security-announce](https://lists.apple.com/archives/list/security-announce@lists.apple.com/) ou consultez les [mises à jour de sécurité d’Apple](https://support.apple.com/100100).

- Chiffrer les données sensibles
  - Activez [FileVault](https://support.apple.com/guide/mac-help/protect-data-on-your-mac-with-filevault-mh11785) pour chiffrer le stockage interne.
  - Utilisez un [gestionnaire de mots de passe](https://support.apple.com/105115) pour les identifiants de compte et envisagez de chiffrer séparément les fichiers particulièrement sensibles.

- Assurer la disponibilité des données
  - Créez des [sauvegardes régulières](https://support.apple.com/104984) des données critiques et soyez prêt à [restaurer à partir d'une sauvegarde](https://support.apple.com/102551) en cas de compromission.
  - [Chiffrez les sauvegardes](https://support.apple.com/guide/mac-help/keep-your-time-machine-backup-disk-secure-mh21241) avant de les copier sur des supports externes ou un stockage cloud tiers. Sinon, utilisez un service de sauvegarde qui fournit un [chiffrement de bout en bout](https://support.apple.com/guide/security/advanced-data-protection-for-icloud-sec973254c5f).
  - Vérifiez les sauvegardes en y accédant à intervalles réguliers.

- Cliquer avec prudence
  - En fin de compte, la sécurité d'un système dépend fortement des capacités et des habitudes de la personne qui l'utilise et l'administre.
  - Faites attention lors de l'installation de nouveaux logiciels : installez-les uniquement à partir de sources que le développeur identifie comme officielles, comme son site web ou son dépôt GitHub.

<a name="threat-model"></a>

# Modèle de menace

L'étape la plus importante pour améliorer significativement la sécurité et la confidentialité est de créer un [modèle de menace](https://owasp.org/www-community/Threat_Modeling) : une description générale de ce que vous souhaitez protéger, qui pourrait tenter d'y accéder, comment ils pourraient le faire, et quels contrôles valent les compromis d'utilisabilité. Cela permet de comprendre les adversaires potentiels et leurs motivations, ce qui conduit à des défenses plus solides.

<a name="assets"></a>

## Actifs

Les actifs peuvent inclure un téléphone, un ordinateur portable, des identifiants et des informations personnelles, comme l'historique de navigation.

Listez-les par ordre d'importance, en commençant par ceux qui méritent le plus d'être protégés.

<a name="adversaries"></a>

## Adversaires

Définissez contre qui vous vous défendez. Commencez par définir la motivation que chaque adversaire pourrait avoir pour attaquer des actifs importants. [Le gain financier](https://www.verizon.com/business/resources/reports/dbir/) est un grand motivateur pour de nombreux attaquants, par exemple.

<a name="capabilities"></a>

## Capacités

Pour chaque adversaire, listez ce qu'il peut et ne peut pas faire, en les classant du moins au plus capable. Par exemple, un voleur opportuniste agit de manière opportuniste : il sera probablement déjoué par des contrôles de base, tels que le verrouillage de l'écran et le stockage chiffré avec des mots de passe forts. Un adversaire plus sophistiqué et déterminé peut nécessiter d'éteindre complètement un appareil lorsqu'il n'est pas utilisé pour effacer les identifiants de la mémoire, ainsi que des mécanismes d'authentification plus robustes.

<a name="mitigations"></a>

## Atténuations

Choisissez la meilleure atténuation pour chaque menace. Par exemple, évitez d'écrire des mots de passe sur papier si un colocataire pourrait les trouver, ou chiffrez le stockage pour protéger ses données en cas de vol.

La sécurité doit être équilibrée avec la convivialité : chaque atténuation doit contrer une capacité adverse pour justifier toute gêne. Arrêtez d'ajouter des défenses lorsque les risques restants sont acceptables pour une situation donnée. Révisez le modèle lorsque les appareils, les données, les déplacements, le travail ou les adversaires changent.

<a name="example-model"></a>

## Exemple de modèle

Le tableau suivant est un exemple simple de modèle de menace pour les appareils personnels, y compris un Mac.

Adversaire | Motivation | Capacités | Atténuation
:-: | :-: | :-: | :-:
Colocataire | Voir les conversations privées ou l'historique de navigation | Proximité immédiate ; peut voir l'écran ou observer les identifiants | Utiliser l'authentification biométrique (empreinte digitale ou visage), utiliser un filtre de confidentialité, garder le téléphone verrouillé lorsqu'il n'est pas utilisé
Voleur | Déverrouiller le téléphone et voler des informations personnelles et vider les comptes bancaires, revendre le téléphone pour de l'argent | Observer la saisie du mot de passe, voler un appareil connecté laissé sans surveillance | Maintenir la garde physique de l'appareil à tout moment, verrouiller les appareils lorsqu'ils ne sont pas utilisés, éviter de taper un mot de passe en public, utiliser Localiser ou des services similaires pour suivre ou désactiver à distance les appareils volés
Criminel | Gain financier | Ingénierie sociale (tromper un utilisateur pour lui faire révéler des informations ou effectuer des actions dangereuses), logiciels malveillants facilement disponibles, réutilisation de mots de passe, exploitation de vulnérabilités | Isoler les logiciels (sandbox), activer les fonctionnalités de sécurité du système d'exploitation, maintenir les mises à jour logicielles automatiquement
Entreprise | Marketing basé sur les données utilisateur | Collecte de télémétrie et de données comportementales | Bloquer les connexions réseau, réinitialiser les identifiants spécifiques à l'appareil, éviter d'ajouter des données de paiement
État-nation/APT | Surveillance ciblée | Surveillance passive de l'infrastructure Internet, analyse avancée du chiffrement | Utiliser des logiciels open source, des identifiants diceware forts, du matériel avec élément sécurisé, éteindre les appareils lorsqu'ils ne sont pas utilisés, alertes tripwire/honeypot/[jetons canari](https://canarytokens.org/)

En savoir plus sur [la modélisation des menaces](https://www.netmeister.org/blog/threat-model-101.html).

<a name="hardware"></a>

# Matériel

[Le matériel Apple silicon](https://support.apple.com/guide/security/hardware-security-overview-secf020d1074/1/web/1) offre des fonctionnalités de sécurité ancrées dans le matériel, notamment la protection des clés basée sur la Secure Enclave et des options de sécurité de démarrage renforcées. Il s'agit généralement de la plateforme préférée pour les protections abordées dans ce guide.

Certains Mac basés sur Intel, en particulier certains modèles dotés de [matériel de l'ère T2](https://en.wikipedia.org/wiki/Apple_T2), sont affectés par des vulnérabilités matérielles qui ne peuvent pas être entièrement corrigées par une simple mise à jour de macOS.

Évitez le matériel non-Apple exécutant macOS et les systèmes qui ne prennent pas en charge la dernière version de macOS, car Apple ne [corrige pas toutes les vulnérabilités](https://support.apple.com/guide/deployment/about-software-updates-depc4c80847a) dans les versions héritées.

Les accessoires Apple reçoivent généralement des mises à jour de firmware via macOS et prennent en charge les [fonctionnalités de sécurité Bluetooth](https://support.apple.com/guide/security/bluetooth-security-sec82597d97e/web) actuelles. Par exemple, la confidentialité [Bluetooth Low Energy](https://en.wikipedia.org/wiki/Bluetooth_Low_Energy) (BLE) utilise des adresses d'appareil rotatives pour réduire le suivi ; les accessoires tiers peuvent ne pas prendre en charge cette fonctionnalité.

<a name="installing-macos"></a>

# Installation de macOS

Il existe plusieurs façons d'[installer macOS](https://support.apple.com/102662).

Installez la dernière version prise en charge de macOS ; les versions plus récentes de macOS incluent des correctifs de sécurité et d'autres améliorations non disponibles dans les versions précédentes.

<a name="system-activation"></a>

## Activation du système

Lors de l'installation, les Mac avec puce Apple contactent les services d'activation d'Apple pour confirmer que l'appareil n'est pas signalé comme [perdu ou volé](https://support.apple.com/102541). Découvrez [comment ce processus fonctionne](https://support.apple.com/guide/security/localpolicy-signing-key-creation-management-sec1f90fbad1).

<a name="apple-account"></a>

## Compte Apple

Un [compte Apple](https://www.apple.com/legal/privacy/data/en/apple-id/) est facultatif pour l'utilisation de base de macOS, mais requis pour l'App Store et de nombreux services Apple. Consultez les [réglages de confidentialité des données](https://support.apple.com/102651) avant d'activer des fonctionnalités.

Activez le [chiffrement de bout en bout](https://support.apple.com/guide/security/advanced-data-protection-for-icloud-sec973254c5f/web) pour iCloud et consultez les [données du compte Apple](https://support.apple.com/102283).

## App Store

L'[App Store](https://en.wikipedia.org/wiki/Mac_app_store) est une plateforme de distribution de logiciels où les applications sont [examinées](https://developer.apple.com/app-store/review/guidelines) par Apple.

Les applications de l'App Store doivent généralement utiliser l'[App Sandbox](https://developer.apple.com/documentation/security/app_sandbox/protecting_user_data_with_app_sandbox) et le [Hardened Runtime](https://developer.apple.com/documentation/security/hardened_runtime) (des fonctionnalités de sécurité limitant ce que les applications peuvent faire, rendant certaines attaques plus difficiles). Il propose également des mises à jour automatiques des logiciels.

L'utilisation de l'App Store nécessite un compte Apple, ce qui peut avoir des implications en matière de confidentialité, car les téléchargements de logiciels et l'activité associée du compte sont liés.

<a name="virtualization"></a>

## Virtualisation

La virtualisation permet à un autre système d'exploitation de s'exécuter dans une machine virtuelle isolée. Sur [Apple silicon](https://developer.apple.com/documentation/virtualization), macOS peut exécuter des invités macOS pris en charge et Windows 11 pour ARM à l'aide d'applications telles que les suivantes :

- [UTM](https://mac.getutm.app/) - Suivez la [documentation](https://docs.getutm.app/guest-support/macos) pour créer des machines virtuelles macOS et autres.
- [VirtualBuddy](https://github.com/insidegui/VirtualBuddy) - Application pour virtualiser macOS 12+ sur Apple silicon.
- [Bushel](https://getbushel.app/) - Une application légère et gratuite. Au premier lancement, sélectionnez « Demander à l'app de ne pas me suivre ».
- [VMware Fusion](https://knowledge.broadcom.com/external/article/315638/download-and-install-vmware-fusion.html) - Désormais gratuit sous Broadcom. Interface épurée, configuration facile de macOS et prise en charge de Windows 11 ARM.
- [tart (CLI)](https://tart.run/) - Application en ligne de commande installée avec Homebrew.
- [Parallels](https://www.parallels.com/) - Option payante avec une intégration poussée.

> [!WARNING]
> VMware nécessite un compte Broadcom et l'acceptation des accords avant le téléchargement.
> Parallels nécessite un compte, des coordonnées de paiement et d'autres données - voir la [politique de confidentialité](https://my.parallels.com/data_reminder).

<a name="apple-containers"></a>

### Conteneurs Apple

[Apple Container](https://github.com/apple/container) est un outil en ligne de commande pour exécuter des conteneurs Linux sur macOS. Contrairement aux environnements d'exécution de conteneurs qui partagent une seule machine virtuelle Linux, Apple Container exécute chaque conteneur dans une machine virtuelle isolée et légère en utilisant les capacités de virtualisation de macOS. Cela offre une frontière d'isolation plus forte entre les charges de travail et le système d'exploitation hôte.

<a name="first-boot"></a>

# Premier démarrage

Lorsque macOS démarre pour la première fois, **Assistant réglages** exige la création d'un compte principal.

Définissez un [mot de passe long et unique](https://www.eff.org/dice). Laissez le champ d'indice de mot de passe vide.

Évitez les noms permettant d'identifier personnellement : le nom de l'ordinateur (comme « MacBook de John Appleseed ») peut être visible par d'autres appareils sur les réseaux locaux via les services de partage et de découverte.

Le nom du système peut être configuré dans **Réglages Système > Général > Informations** ou avec les commandes [`scutil`](https://ss64.com/mac/scutil.html) :

```bash
sudo scutil --set ComputerName MacBook
sudo scutil --set HostName MacBook
sudo scutil --set LocalHostName MacBook
```

<a name="admin-and-user-accounts"></a>

# Comptes administrateur et utilisateur

Le premier compte créé est un compte administrateur. Les administrateurs peuvent modifier les réglages à l'échelle du système et exécuter des commandes avec [sudo](https://en.wikipedia.org/wiki/Sudo), qui accorde temporairement des privilèges élevés. Tout programme exécuté par l'administrateur pourrait obtenir le même accès ; sudo peut également présenter des [vulnérabilités](https://bogner.sh/2014/03/another-mac-os-x-sudo-password-bypass/).

Il est considéré comme une [bonne pratique](https://help.apple.com/machelp/mac/10.12/index.html#/mh11389) d'utiliser un compte standard pour le travail quotidien et de réserver un compte administrateur pour les modifications système nécessitant des privilèges élevés.

Il n'est pas nécessaire de se connecter un jour avec le compte administrateur via l'écran de connexion macOS. Lorsqu'une commande Terminal nécessite des privilèges d'administrateur, le système demandera une authentification et Terminal continuera d'utiliser ces privilèges. À cette fin, Apple fournit des [recommandations](https://support.apple.com/102099) pour masquer le compte administrateur et son dossier de départ.

<a name="caveats"></a>

## Mises en garde

- Seuls les administrateurs peuvent installer des applications dans le répertoire système `/Applications`. Finder et Installer demandent à un utilisateur standard de saisir les identifiants d'un administrateur lorsque l'approbation est requise. Certaines applications peuvent plutôt être installées dans `~/Applications` ; les applications qui ne nécessitent pas d'accès administrateur doivent être installées dans le répertoire utilisateur. Les applications de l'App Store sont toujours installées dans `/Applications` et ne nécessitent aucune authentification supplémentaire.
- Un utilisateur standard n'est généralement pas autorisé à utiliser `sudo`. Lorsque des privilèges d'administrateur sont requis, macOS demande les identifiants d'un administrateur, ou la tâche peut être exécutée à partir d'un compte administrateur.
- Réglages Système et plusieurs utilitaires système (par exemple, Diagnostics Wi-Fi) nécessitent l'autorisation d'administrateur pour une fonctionnalité complète. Certains Réglages Système doivent être déverrouillés en sélectionnant l'icône de cadenas. Certaines applications demanderont simplement une authentification à l'ouverture, d'autres doivent être ouvertes directement par un compte administrateur pour accéder à toutes les fonctions (par exemple, Console).
- Certaines applications tierces supposent que l'utilisateur actuel est administrateur et ne fonctionneront pas à partir d'un compte standard.
- Voir [le ticket 167](https://github.com/drduh/macOS-Security-and-Privacy-Guide/issues/167) pour des considérations supplémentaires.

<a name="setup"></a>

## Configuration

Les comptes peuvent être créés et gérés dans Réglages Système.

Sur les systèmes existants, il est généralement plus simple de créer d'abord un second compte administrateur, puis de convertir le compte d'origine en compte standard. Sur les systèmes nouvellement installés, il convient plutôt d'ajouter un compte standard après la configuration.

La rétrogradation d'un compte peut être effectuée depuis le nouveau compte administrateur dans Réglages Système.

Voir aussi [cet article](https://superuser.com/a/395738) pour plus d'informations sur la façon dont macOS détermine l'appartenance aux groupes.

<a name="firmware"></a>

# Micrologiciel

Vérifiez que la sécurité du micrologiciel est réglée sur [Sécurité complète](https://support.apple.com/guide/mac-help/mchl768f7291/mac) pour empêcher toute altération du système. Il s'agit du réglage par défaut.

# FileVault

Les Mac Apple silicon utilisent par défaut le chiffrement du stockage pris en charge par le matériel. [FileVault](https://support.apple.com/guide/mac-help/protect-data-on-your-mac-with-filevault-mh11785/mac) exige en outre le mot de passe d'un utilisateur autorisé pour déverrouiller le volume de démarrage après un redémarrage.

Sur Apple silicon, l'autorisation FileVault protège également le [micrologiciel](https://support.apple.com/102384), ce qui empêche de démarrer à partir de tout autre disque que le disque de démarrage désigné, d'accéder à [Récupération](https://support.apple.com/guide/mac-help/macos-recovery-a-mac-apple-silicon-mchl82829c17/15.0/mac/15.0#mchl5abfbb29) et de le [restaurer](https://support.apple.com/108900) avec le mode DFU (device firmware update).

FileVault proposera de définir une clé de récupération, qui doit être conservée en lieu sûr si elle est utilisée. FileVault offre également une option d'utilisation d'iCloud pour la récupération.

Confirmez l'état de FileVault :

```bash
fdesetup status
```

<a name="lockdown-mode"></a>

# Mode Isolement

[Le mode Isolement](https://support.apple.com/105120) réduit considérablement la surface d'attaque en désactivant des fonctionnalités système et d'applications couramment exploitées dans des attaques sophistiquées.

Lorsque le mode Isolement est activé, Safari propose une option pour [exclure des sites web de confiance](https://ssd.eff.org/module/how-to-enable-lockdown-mode-on-iphone) des restrictions.

Confirmez l'état du mode Isolement :

```bash
sysctl -n security.mac.lockdown_mode_state 2>/dev/null | awk '{print "Lockdown Mode: " ($1 == 1 ? "ON" : "OFF")}'
```

<a name="firewall"></a>

# Pare-feu

Plusieurs types de pare-feu sont disponibles pour macOS.

> [!NOTE]
> Si un logiciel malveillant obtient le contrôle administrateur, il peut désactiver ou contourner les stratégies de pare-feu ; ne comptez pas sur un pare-feu local comme protection robuste contre une compromission totale du système.

<a name="application-layer-firewall"></a>

## Pare-feu de couche applicative

Le pare-feu intégré offre une protection de base en contrôlant les connexions réseau entrantes vers les applications. Il ne surveille ni ne bloque les connexions sortantes.

Il peut être contrôlé par l'onglet **Pare-feu** de **Réseau** dans **Réglages Système**, ou avec la commande suivante :

```bash
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on
```

<a name="stealth-mode"></a>

### Mode furtif

Les attaquants analysent fréquemment les réseaux pour identifier les systèmes à cibler. Lorsque le [mode furtif](https://support.apple.com/guide/mac-help/use-stealth-mode-to-keep-your-mac-more-secure-mh17133/mac) est activé, aucune réponse n'est envoyée aux tentatives de connexion depuis des ports fermés, ce qui rend le système plus difficile à détecter.

Activer le mode furtif :

```bash
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on
```

<a name="signed-apps"></a>

### Apps signées

Par défaut, le pare-feu autorise les connexions entrantes pour les logiciels signés par Apple ou par un développeur identifié. La désactivation de ces exceptions amène macOS à demander une confirmation avant que ces applications puissent accepter des connexions entrantes :

```bash
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setallowsigned off
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setallowsignedapp off
```

<a name="reload-firewall"></a>

### Recharger le pare-feu

Après avoir modifié les réglages avec `socketfilterfw`, rechargez le service de pare-feu pour appliquer la nouvelle configuration :

```bash
sudo pkill -HUP socketfilterfw
```

<a name="get-state"></a>

### Obtenir l'état

Confirmer l'état du pare-feu :

```bash
for firewallFlag in \
  --getglobalstate \
  --getblockall \
  --getallowsigned \
  --getstealthmode \
  --listapps
do /usr/libexec/ApplicationFirewall/socketfilterfw "$firewallFlag"
done
```

Ou en utilisant le [profil système](https://ss64.com/mac/system_profiler.html) :

```bash
system_profiler SPFirewallDataType
```

### AirDrop

L'activation du pare-feu de couche applicative et la désactivation des connexions entrantes pour les logiciels intégrés empêchent [AirDrop](https://support.apple.com/119857) de fonctionner. Pour qu'AirDrop fonctionne, accordez les exceptions de pare-feu `sharingd` et `rapportd` :

```bash
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /usr/libexec/rapportd
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /usr/libexec/sharingd
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --unblockapp /usr/libexec/rapportd
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --unblockapp /usr/libexec/sharingd
```

<a name="third-party-firewalls"></a>

## Pare-feu tiers

Des applications telles que [Little Snitch](https://www.obdev.at/products/littlesnitch/index.html), [Radio Silence](https://radiosilenceapp.com/) et [LuLu](https://objective-see.com/products/lulu.html) offrent un équilibre entre facilité d'utilisation et sécurité.

Ces programmes sont capables de surveiller et de bloquer les connexions entrantes et sortantes. Toutefois, ils peuvent nécessiter une [extension système](https://support.apple.com/HT210999) à code source fermé.

Si les demandes fréquentes d'autorisation ou de blocage sont trop nombreuses, autorisez temporairement les connexions tout en les enregistrant (mode [Silent Allow](https://help.obdev.at/littlesnitch6/concepts-opmodes) dans Little Snitch). Examinez la configuration pour comprendre et contrôler l'activité réseau.

<a name="packet-filter"></a>

## Filtre de paquets

macOS inclut également [pf](https://en.wikipedia.org/wiki/PF_(firewall)), un pare-feu de filtrage de paquets configuré en ligne de commande. Il est puissant mais nettement plus complexe que le pare-feu d'application intégré.

pf peut également être contrôlé avec une application graphique telle que [Murus](https://www.murusfirewall.com/).

De nombreux [ouvrages](https://nostarch.com/book-of-pf-4e) et [guides](https://www.openbsd.org/faq/pf/) traitent du pare-feu pf. L'exemple suivant montre comment configurer une politique de base.

<a name="example-pf-config"></a>

### Exemple de configuration pf

Ajoutez les règles d'exemple suivantes à un fichier tel que `pf.rules` :

```console
# Define interface
wifi = "en0"

# Global options
set block-policy drop
set skip on lo0
set state-policy if-bound
set ruleset-optimization basic

# Normalize
scrub in on $wifi all fragment reassemble

# Define tables
table <blocklist> persist

# Deny and log blocked traffic
block log all
block quick from no-route to any
antispoof quick for $wifi

# Enforce blocklist
block log quick on $wifi from <blocklist> to any
block log quick on $wifi from any to <blocklist>

# Allow DHCP
pass out on $wifi proto udp from any port 68 to any port 67 keep state
pass in  on $wifi proto udp from any port 67 to any port 68 keep state

# Allow outbound TCP
pass out on $wifi proto tcp from ($wifi) to any flags S/SA keep state

# Allow outbound UDP
pass out on $wifi proto udp from ($wifi) to any keep state

# Allow outbound ICMP (ping)
pass out on $wifi proto icmp from ($wifi) to any keep state
```

Un exemple avancé de configuration de pf est disponible dans [config/pf.rules](https://github.com/drduh/macOS-Security-and-Privacy-Guide/blob/main/config/pf.rules).

<a name="firewall-commands"></a>

### Commandes du pare-feu

Pour contrôler le pare-feu :

Commande | Tâche
-: | :-
`sudo pfctl -e -f pf.rules` | activer le pare-feu avec le fichier de configuration
`sudo pfctl -t blocklist -T add 1.2.3.4` | ajouter une adresse IPv4 à la table de liste de blocage
`sudo pfctl -d` | désactiver le pare-feu

Pour surveiller le pare-feu :

Commande | Tâche
-: | :-
`sudo pfctl -t blocklist -T show` | afficher la liste de blocage
`sudo pfctl -s rules` | afficher les règles actives
`sudo pfctl -s states` | afficher la table d'état
`sudo ifconfig pflog0 create` | créer une interface de journal de paquets
`sudo tcpdump -ni pflog0` | surveiller les paquets bloqués

<a name="block-networks"></a>

### Bloquer des réseaux

pf peut bloquer des plages d'adresses réseau, par exemple pour une organisation entière. Interrogez [Merit RADb](https://www.radb.net/) pour obtenir la liste des réseaux utilisés par un [système autonome](https://en.wikipedia.org/wiki/Autonomous_system_(Internet)) (un grand réseau exploité par une seule organisation), comme [Facebook](https://ipinfo.io/AS32934) :

```bash
whois -h whois.radb.net '!gAS32934'
```

Ajoutez la liste de réseaux renvoyée à la liste de blocage :

```bash
sudo pfctl -t blocklist -T add $(whois -h whois.radb.net '!gAS32934' | sed -n '2p')
```

Confirmez que les adresses ont été ajoutées :

```console
$ sudo pfctl -t blocklist -T show
No ALTQ support in kernel
ALTQ related functions disabled
   31.13.24.0/21
   31.13.64.0/24
   157.240.0.0/16
   ...
```

Confirmez que le trafic réseau est bloqué vers ces adresses (les requêtes DNS fonctionneront toujours) :

```console
$ dig a +short facebook.com
157.240.2.35

$ curl --connect-timeout 5 -I https://facebook.com/
*   Trying 157.240.2.35...
* TCP_NODELAY set
* Connection timed out after 5002 milliseconds
* Closing connection 0
curl: (28) Connection timed out after 5002 milliseconds

$ sudo tcpdump -tqni pflog0 'host 157.240.2.35'
IP 192.168.1.1.62771 > 157.240.2.35.443: tcp 0
IP 192.168.1.1.62771 > 157.240.2.35.443: tcp 0
IP 192.168.1.1.62771 > 157.240.2.35.443: tcp 0
```

Le pare-feu abandonne les paquets initiaux nécessaires pour démarrer la [connexion TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Connection_establishment), la connexion expire donc.

Voir [drduh/config/scripts/pf-blocklist.sh](https://github.com/drduh/config/blob/main/scripts/pf-blocklist.sh) pour plus d'inspiration.

# Services

De nombreux services système et d'arrière-plan utilisateur sont gérés par [launchd](https://en.wikipedia.org/wiki/Launchd).

Les comptes administrateur peuvent modifier les services et extensions dans [Réglages Système](https://support.apple.com/guide/mac-help/change-login-items-settings-mtusr003).

Commande | Tâche
-: | :-
`launchctl list` | afficher les tâches utilisateur
`sudo launchctl list` | afficher les tâches système
`launchctl list com.apple.Finder` | examiner un service
`systemextensionsctl list` | lister les extensions système
`ls /System/Library/LaunchDaemons` | lister les démons système
`ls /System/Library/LaunchAgents` | lister les agents système

Pour lire la configuration d'un service :

```bash
defaults read /System/Library/LaunchAgents/com.apple.Finder
```

> [!IMPORTANT]
> Les services système sont protégés par [SIP](https://github.com/drduh/macOS-Security-and-Privacy-Guide#system-integrity-protection) ; désactiver SIP pour modifier les services système peut compromettre la sécurité et provoquer une instabilité du système.

Pour afficher l'état des services :

```bash
find /var/db/com.apple.xpc.launchd \
  -type f -print -exec defaults read {} \; 2>/dev/null
```

Voir [gestion des scripts avec launchd](https://support.apple.com/guide/terminal/script-management-with-launchd-apdc6c1077b-5d5d-4d35-9c19-60f2397b2369) et [launchd.info](https://launchd.info/) pour plus d'informations.

<a name="siri-suggestions-and-spotlight"></a>

# Suggestions Siri et Spotlight

Les suggestions Siri et Spotlight peuvent envoyer certaines requêtes ou informations d'utilisation à Apple, selon les fonctionnalités activées.
Dans Réglages Système, examinez les options Siri et Spotlight et désactivez les suggestions en ligne ou configurez les catégories à exclure de l'indexation ou des suggestions. Consultez la [politique de recherche et de confidentialité](https://www.apple.com/legal/privacy/data/en/siri-suggestions-search/) d'Apple pour plus d'informations.

# Homebrew

Si un programme n'est pas disponible sur l'App Store, envisagez d'utiliser [Homebrew](https://brew.sh/).

> [!IMPORTANT]
> Certains flux d'installation ou de gestion de Homebrew peuvent demander la Gestion des apps ou l'[Accès complet au disque](https://eclecticlight.co/2025/11/08/explainer-permissions-privacy-and-tcc/). N'accordez ces autorisations que lorsqu'elles sont pleinement comprises et nécessaires.

Exécutez périodiquement `brew upgrade` sur des réseaux fiables et sécurisés pour installer les mises à jour logicielles. Pour obtenir des informations sur un paquet avant l'installation, exécutez `brew info <package>` et consultez sa formule en ligne. Activez les [options de sécurité supplémentaires](https://github.com/drduh/macOS-Security-and-Privacy-Guide/issues/138), telles que `HOMEBREW_NO_INSECURE_REDIRECT=1`

Selon les [analytiques anonymes de Homebrew](https://docs.brew.sh/Analytics), Homebrew collecte des données d'utilisation anonymes et les rapporte à une instance [InfluxDB](https://en.wikipedia.org/wiki/InfluxDB) auto-hébergée.

Pour désactiver les analytiques de Homebrew, exécutez `brew analytics off` ou définissez `HOMEBREW_NO_ANALYTICS=1` dans le fichier de démarrage du shell.

# DNS

<a name="dns-profiles"></a>

## Profils DNS

macOS prend en charge les profils de configuration pour définir le DNS chiffré, le filtrage et les options [DNSSEC](https://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions).

Les profils DNS peuvent être [créés](https://dns.notjakob.com/) ou obtenus auprès de fournisseurs tels que [Quad9](https://docs.quad9.net/Setup_Guides/MacOS/Big_Sur_and_later_(Encrypted)/#download-profile), [AdGuard](https://adguard-dns.io/en/public-dns.html) et [NextDNS](https://nextdns.io/).

<a name="hosts-file"></a>

## Fichier hosts

Utilisez le [fichier hosts](https://en.wikipedia.org/wiki/Hosts_(file)) pour bloquer les domaines associés à des logiciels malveillants, à la publicité et à d'autres services indésirables.

Pour bloquer un domaine par [enregistrement A](https://en.wikipedia.org/wiki/List_of_DNS_record_types), ajoutez l'une des lignes suivantes à `/etc/hosts` :

```
0 example.com
0.0.0.0 example.com
127.0.0.1 example.com
```

> [!NOTE]
> IPv6 utilise des enregistrements AAAA plutôt que des enregistrements A : pour remplacer la résolution de noms IPv6, incluez une ligne telle que `::1 example.com`.

De nombreuses listes de domaines sont disponibles en ligne. Avant d'en ajouter une à `/etc/hosts`, assurez-vous que chaque entrée commence par `0`, `0.0.0.0` ou `127.0.0.1`, et conservez l'entrée `127.0.0.1 localhost`.

Les listes de hosts populaires incluent :
- [StevenBlack/hosts](https://github.com/StevenBlack/hosts)
- [Sinfonietta/hostfiles](https://github.com/Sinfonietta/hostfiles)
- [someonewhocares.org](https://someonewhocares.org/hosts/zero/hosts)

Pour télécharger et ajouter une liste au fichier hosts, utilisez la commande [tee](https://man7.org/linux/man-pages/man1/tee.1.html) :

```bash
curl https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts |
  sudo tee -a /etc/hosts
```

[Little Snitch](#third-party-firewalls) prend également en charge les [listes de blocage](https://help.obdev.at/littlesnitch6/lsc-blocklists).

## DNSCrypt

Pour chiffrer le trafic DNS, envisagez [DNSCrypt/dnscrypt-proxy](https://github.com/DNSCrypt/dnscrypt-proxy). Lorsqu'il est configuré avec un résolveur de confiance, DNSCrypt chiffre les requêtes DNS. DNSSEC peut en outre valider les réponses DNS signées lorsque cela est pris en charge.

Installez DNSCrypt depuis Homebrew et suivez les instructions pour configurer et démarrer `dnscrypt-proxy` :

```bash
brew install dnscrypt-proxy
```

Lors de l'utilisation de DNSCrypt avec Dnsmasq, localisez le fichier de configuration de DNSCrypt en exécutant :

```bash
brew info dnscrypt-proxy | grep dnscrypt-proxy.toml | head -1
```

Cette commande doit afficher un chemin tel que `/opt/homebrew/etc/dnscrypt-proxy.toml`.

Par défaut, dnscrypt-proxy écoute sur `127.0.0.1:53` et envoie les requêtes à un ou plusieurs fournisseurs DNS configurés. Modifiez la configuration et changez `listen_addresses` pour utiliser un port autre que 53, comme 5355 :

```toml
listen_addresses = ['127.0.0.1:5355', '[::1]:5355']
```

Démarrer DNSCrypt :

```bash
sudo brew services restart dnscrypt-proxy
```

Confirmer que DNSCrypt est en cours d'exécution :

```console
$ sudo lsof +c 15 -Pni UDP:5355
COMMAND          PID   USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
dnscrypt-proxy 15244 nobody    7u  IPv4 0x1337f85ff9f8beef      0t0  UDP 127.0.0.1:5355
dnscrypt-proxy 15244 nobody   10u  IPv6 0x1337f85ff9f8beef      0t0  UDP [::1]:5355
dnscrypt-proxy 15244 nobody   12u  IPv4 0x1337f85ff9f8beef      0t0  UDP 127.0.0.1:5355
dnscrypt-proxy 15244 nobody   14u  IPv6 0x1337f85ff9f8beef      0t0  UDP [::1]:5355
```

De plus, ces règles pf bloquent le trafic DNS conventionnel vers le port 53 en dehors de l'interface de bouclage, afin de réduire le risque de [fuites DNS](https://dnsleaktest.com/) :

```console
block drop quick on !lo0 proto udp from any to any port = 53
block drop quick on !lo0 proto tcp from any to any port = 53
```

## Dnsmasq

[dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html) peut mettre en cache les réponses, empêcher les requêtes en amont pour des noms non qualifiés, ou même bloquer des domaines de premier niveau entiers.

Utilisez-le en combinaison avec DNSCrypt pour chiffrer le trafic DNS.

Si vous n'utilisez pas DNSCrypt, choisissez au minimum un résolveur DNS autre que celui fourni par le FAI. Deux alternatives populaires sont [Google DNS](https://developers.google.com/speed/public-dns/) et [OpenDNS](https://www.opendns.com/home-internet-security/).

Installer Dnsmasq :

```bash
brew install dnsmasq
```

Téléchargez et modifiez [drduh/config/dnsmasq.conf](https://github.com/drduh/config/blob/main/dnsmasq.conf) ou le fichier de configuration par défaut.

Voir [drduh/config/domains](https://github.com/drduh/config/tree/main/domains) pour des exemples de listes de domaines pouvant être ajoutées pour bloquer des destinations spécifiques.

Installez et démarrez le programme (sudo est requis pour se lier au [port privilégié](https://unix.stackexchange.com/questions/16564/why-are-the-first-1024-ports-restricted-to-the-root-user-only) 53) :

```bash
sudo brew services start dnsmasq
```

Pour définir dnsmasq comme serveur DNS local, ouvrez **Réglages Système** > **Réseau** et sélectionnez la connexion réseau actuellement utilisée (comme Wi-Fi), puis ouvrez l'onglet **DNS**, sélectionnez **+**, et ajoutez `127.0.0.1`, ou utilisez :

```bash
sudo networksetup -setdnsservers "Wi-Fi" 127.0.0.1
```

Confirmer que Dnsmasq est configuré :

```console
$ scutil --dns | head
DNS configuration

resolver #1
  search domain[0] : whatever
  nameserver[0] : 127.0.0.1
  flags    : Request A records, Request AAAA records
  reach    : 0x00030002 (Reachable,Local Address,Directly Reachable Address)

$ networksetup -getdnsservers "Wi-Fi"
127.0.0.1
```

> [!NOTE]
> Certaines applications VPN remplacent les paramètres DNS lors de la connexion. Voir [issue 24](https://github.com/drduh/macOS-Security-and-Privacy-Guide/issues/24) et [drduh/config/scripts/macos-dns.sh](https://github.com/drduh/config/blob/main/scripts/macos-dns.sh).

<a name="certificate-authorities"></a>

# Autorités de certification

macOS inclut un ensemble d'autorités de certification racines de confiance (CA) exploitées par des entreprises, des gouvernements et d'autres organisations du monde entier. Une CA de confiance peut émettre des certificats que macOS et les navigateurs peuvent accepter pour les connexions HTTPS, ce qui fait du magasin de CA racines une partie critique de la frontière de confiance du système.

Inspectez **Trousseaux système** pour comprendre le magasin de racines de confiance fourni par Apple à l'aide de [Trousseau d'accès](https://support.apple.com/guide/keychain-access/welcome/mac) ou de l'outil en ligne de commande [`security`](https://ss64.com/mac/security.html) et du fichier `/System/Library/Keychains/SystemRootCertificates.keychain`. Trousseau d'accès peut également être lancé avec la commande :

```bash
open "/System/Library/CoreServices/Applications/Keychain Access.app"
```

Examinez également les certificats qui ont pu être ajoutés par un utilisateur, un administrateur, un client VPN, un produit de sécurité ou une application de gestion de périphériques mobiles dans les trousseaux **login**, **Local Items** et **System**.

Pour désactiver un certificat sélectionné, modifiez son réglage de confiance sur **Never Trust** et fermez la fenêtre pour confirmer. Cela peut réduire le risque d'[attaques MITM](https://wikipedia.org/wiki/Man-in-the-middle_attack), dans lesquelles un certificat frauduleux est utilisé pour [intercepter silencieusement](https://en.wikipedia.org/wiki/DigiNotar#Issuance_of_fraudulent_certificates) le trafic chiffré.

> [!WARNING]
> La suppression ou la modification des réglages de confiance des autorités de certification peut perturber les sites web, les mises à jour logicielles, les réseaux d'entreprise, les VPN, les portails captifs et d'autres services.

Consultez les listes Apple des [certificats de confiance disponibles](https://support.apple.com/103723) et des [certificats bloqués](https://support.apple.com/103247#blocked) pour obtenir des informations à jour sur le magasin de confiance.

# Privoxy

Envisagez d'utiliser [Privoxy](https://www.privoxy.org/) comme proxy local pour filtrer le trafic web.

Installez et démarrez Privoxy avec Homebrew :

```bash
brew install privoxy
brew services start privoxy
```

Alternativement, un paquet d'installation signé pour Privoxy est disponible sur [leur site web](https://www.privoxy.org/sf-download-mirror/Macintosh%20%28OS%20X%29/) ou [Sourceforge](https://sourceforge.net/projects/ijbswa/files/Macintosh%20%28OS%20X%29/). Le paquet signé est [plus sécurisé](https://github.com/drduh/macOS-Security-and-Privacy-Guide/issues/65) que la version Homebrew et bénéficie du support du projet Privoxy.

Par défaut, Privoxy écoute sur le port TCP local 8118.

Définissez le proxy **HTTP** du système pour l'interface réseau active sur `127.0.0.1` et le port `8118` :

```bash
sudo networksetup -setwebproxy "Wi-Fi" 127.0.0.1 8118
```

Définissez le proxy **HTTPS** du système :

```bash
sudo networksetup -setsecurewebproxy "Wi-Fi" 127.0.0.1 8118
```

Le proxy peut également être configuré dans **Réglages Système** > **Réseau** > **Détails** > **Proxys**.

Confirmez que le proxy est défini :

```console
$ scutil --proxy
<dictionary> {
  ExceptionsList : <array> {
    0 : *.local
    1 : 169.254/16
  }
  FTPPassive : 1
  HTTPEnable : 1
  HTTPPort : 8118
  HTTPProxy : 127.0.0.1
  HTTPSEnable : 1
  HTTPSPort : 8118
  HTTPSProxy : 127.0.0.1
}
```

Bien que la plupart du trafic web soit chiffré, Privoxy peut toujours filtrer de nombreuses requêtes par nom d'hôte lorsque le trafic est configuré pour utiliser le proxy. Par exemple, les règles suivantes bloquent tout le trafic sauf celui vers les domaines .net, github.com et Apple :

```console
{ +block{all} }
.

{ -block }
.apple.
.github.com
.net
```

Pour bloquer les domaines Facebook :

```console
{ +block{facebook} }
.cdninstagram.
.facebook*.
.fb.
.fbcdn*.
.fbinfra.
.fbsbx.
.fbstatic*.
.fbsv.
.fburl.
.instagr.am
.tfbnw.
.thefacebook.
fb*.akamaihd.net
```

Consultez [drduh/config/privoxy/config](https://github.com/drduh/config/blob/main/privoxy/config) et [drduh/config/privoxy/user.action](https://github.com/drduh/config/blob/main/privoxy/user.action) pour d'autres exemples Privoxy. Privoxy n'a **pas** besoin d'être redémarré après modification des règles de filtrage.

Pour vérifier que le trafic est bloqué ou redirigé, utilisez [curl](https://en.wikipedia.org/wiki/CURL) ou ouvrez l'interface Privoxy à l'adresse <http://p.p> dans un navigateur :

```console
$ ALL_PROXY=127.0.0.1:8118 curl example.com -IL | head
HTTP/1.1 403 Request blocked by Privoxy
Content-Length: 9001
Content-Type: text/html
Cache-Control: no-cache
Pragma: no-cache

$ ALL_PROXY=127.0.0.1:8118 curl github.com -IL | head
HTTP/1.1 302 Local Redirect from Privoxy
Location: https://github.com/
Content-Length: 0

HTTP/1.1 200 Connection established

HTTP/2 200
content-type: text/html; charset=utf-8
```

> [!NOTE]
> Les applications et services peuvent ignorer les paramètres de proxy. Les règles pf peuvent rediriger le trafic via un proxy sans configurer chaque application séparément.

<a name="browser"></a>

# Navigateur

Les navigateurs web créent des risques significatifs de sécurité et de confidentialité car ils téléchargent et exécutent régulièrement du contenu complexe et non fiable depuis Internet, notamment du JavaScript, des médias et des documents.

Une frontière de sécurité clé du navigateur est la [politique de même origine](https://en.wikipedia.org/wiki/Same-origin_policy) (SOP), qui empêche un site web de lire les données d'un autre site web. Un contournement peut exposer des données ou des actions d'autres sites dans le même profil de navigateur.

Certaines attaques de navigateur reposent sur la conviction de l'utilisateur d'installer un logiciel, d'accorder des autorisations ou d'utiliser une extension malveillante. Soyez particulièrement prudent avec les téléchargements inattendus, les invites d'extension et les demandes d'exécution de commandes dans le Terminal.

Les extensions de navigateur posent également un risque de sécurité important : une extension malveillante ou mal conçue peut compromettre tout ce qui se trouve dans le navigateur, y compris les identifiants. L'utilisation d'extensions de navigateur doit être limitée à celles strictement nécessaires, publiées uniquement par des développeurs dignes de confiance.

Utilisez des profils de navigateur séparés pour différentes identités et objectifs. Si possible, désactivez JavaScript et ne l'autorisez que sur les sites de confiance en utilisant les autorisations de site du navigateur.

[Mozilla Firefox](https://www.firefox.com/), [Google Chrome](https://www.google.com/chrome), [Safari](https://www.apple.com/safari) et [Tor Browser](https://www.torproject.org/download) sont des navigateurs populaires, chacun avec des fonctionnalités uniques et des objectifs individuels.

## Firefox

Firefox a modernisé des parties majeures de sa base de code grâce aux projets [Quantum](https://wiki.mozilla.org/Quantum) et [Photon](https://wiki.mozilla.org/Firefox/Photon/Updates). Quantum inclut le déplacement de certains composants de C++ vers [Rust](https://rust-lang.org/), un langage de programmation conçu pour améliorer la sécurité de la mémoire et la sécurité de la concurrence.

Firefox offre un modèle de sécurité comparable à Chrome, incluant un [programme de prime aux bogues](https://www.mozilla.org/security/bug-bounty/) pour la divulgation responsable des vulnérabilités. Firefox suit un cycle de publication de quatre semaines.

Voir [drduh/config/firefox.user.js](https://github.com/drduh/config/blob/main/firefox.user.js) et [arkenfox/user.js](https://github.com/arkenfox/user.js) pour les configurations recommandées. Voir aussi [NoScript](https://noscript.net/), une extension qui permet le blocage sélectif des scripts.

Firefox [met l'accent sur la confidentialité de l'utilisateur](https://www.mozilla.org/firefox/privacy). Il prend en charge la [protection contre le suivi](https://developer.mozilla.org/docs/Web/Privacy/Firefox_tracking_protection) en mode Navigation privée. La protection contre le suivi peut être activée pour le compte par défaut, bien qu'elle puisse perturber l'expérience de navigation sur certains sites web. Firefox en mode de protection stricte contre le suivi [randomise les empreintes](https://support.mozilla.org/kb/firefox-protection-against-fingerprinting) pour se défendre contre le suivi. Firefox propose des [profils](https://support.mozilla.org/kb/profile-manager-create-remove-switch-firefox-profiles) utilisateur séparés. La navigation peut également être délimitée avec [Multi-Account Containers](https://support.mozilla.org/kb/containers).

Firefox ne prend en charge que les Web Extensions via l'[API WebExtension](https://developer.mozilla.org/docs/Mozilla/Add-ons/WebExtensions), qui est très similaire à Chrome. La soumission de Web Extensions dans Firefox est gratuite. Les Web Extensions dans Firefox sont la plupart du temps open-source, bien que certaines Web Extensions soient propriétaires.

## Chrome

[Google Chrome](https://www.google.com/chrome/) est basé sur le projet open-source [Chromium](https://www.chromium.org/). Chrome inclut certains composants propriétaires, des mises à jour automatiques, un lecteur PDF, la prise en charge des codecs multimédias, le rapport de plantage et l'intégration du compte Google. Consultez les paramètres de confidentialité de Chrome et les politiques de confidentialité de Google pour plus d'informations.

Chrome offre une synchronisation de compte entre plusieurs appareils, y compris les identifiants ; les données sont chiffrées avec le mot de passe du compte.

Chrome a la plus grande part d'utilisation mondiale et est la plateforme cible préférée de la majorité des développeurs. Les technologies majeures sont basées sur les composants open-source de Chrome, tels que [node.js](https://nodejs.org/) qui utilise le [moteur V8 de Chrome](https://developers.google.com/v8) et le framework [Electron](https://electron.atom.io/), qui est basé sur Chromium et node.js. La vaste base d'utilisateurs de Chrome en fait la cible la plus attrayante pour les acteurs de menace et les chercheurs en sécurité. Malgré des attaques constantes, Chrome a conservé un bilan de sécurité impressionnant au fil des ans. Ce n'est pas un mince exploit.

Chrome offre des [profils séparés](https://www.chromium.org/user-experience/multi-profiles), un [bac à sable robuste](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/design/sandbox.md), des [mises à jour fréquentes](https://chromereleases.googleblog.com/), et possède des [identifiants impressionnants](https://www.chromium.org/Home/chromium-security/brag-sheet). De plus, Google propose un [programme de prime](https://bughunters.google.com/about/rules/5745167867576320/chrome-vulnerability-reward-program-rules) très lucratif pour signaler les vulnérabilités, ainsi que sa propre équipe [Project Zero](https://googleprojectzero.blogspot.com/). Cela signifie qu'un grand nombre de personnes très talentueuses et motivées auditent et sécurisent constamment le code de Chrome.

Envisagez de désactiver l'optimisation V8 (fonctionnalités de performance du moteur JavaScript) dans les paramètres du navigateur - voir [cette explication](https://microsoftedge.github.io/edgevr/posts/Super-Duper-Secure-Mode) pour les compromis de sécurité.

Bloquez les traqueurs avec [uBlock Origin Lite](https://chromewebstore.google.com/detail/ublock-origin-lite/ddkjiahejlhfcafbddmgiahcphecmpfh).

Désactivez [la prélecture DNS](https://www.chromium.org/developers/design-documents/dns-prefetching) (voir [Prélecture DNS et ses implications pour la vie privée](https://www.usenix.org/legacy/event/leet10/tech/full_papers/Krishnan.pdf)). Chrome [peut tenter](https://github.com/drduh/macOS-Security-and-Privacy-Guide/issues/350) de résoudre le DNS en utilisant les serveurs de noms publics `8.8.8.8` et `8.8.4.4` de Google.

Voir [Sécurité Chromium](https://www.chromium.org/Home/chromium-security) et [Confidentialité Chromium](https://www.chromium.org/Home/chromium-privacy) pour plus d'informations. Lisez [la politique de confidentialité de Google](https://policies.google.com/privacy) pour comprendre comment les informations personnelles sont collectées et utilisées.

## Safari

[Safari](https://www.apple.com/safari) est le navigateur intégré d'Apple et est intégré à macOS, offrant des améliorations d'efficacité énergétique sur le matériel Apple.

Safari a à la fois des composants open source et propriétaires. Safari est basé sur le moteur Web open source [WebKit](https://webkit.org/), qui est omniprésent dans l'écosystème macOS. WebKit est utilisé par les applications Apple telles que Mail, Books et l'App Store. Le moteur [Blink](https://www.chromium.org/blink) de Chrome est un fork de WebKit et les deux moteurs partagent un certain nombre de similitudes.

Safari inclut certaines fonctionnalités qui peuvent améliorer la sécurité et la confidentialité. [Les bloqueurs de contenu](https://webkit.org/blog/3476/content-blockers-first-look) permettent la création de règles de blocage de contenu sans utiliser JavaScript. Cette approche basée sur des règles améliore considérablement l'utilisation de la mémoire, la sécurité et la confidentialité. Safari 11 a introduit [la Prévention Intelligente du Suivi](https://webkit.org/blog/7675/intelligent-tracking-prevention), qui supprime les données de suivi stockées dans Safari après une période de non-interaction de l'utilisateur avec le site Web du traqueur. Safari peut randomiser l'empreinte du navigateur pour réduire le suivi. Safari ne prend pas en charge certaines fonctionnalités telles que WebUSB ou l'API Battery intentionnellement pour des raisons de sécurité et de confidentialité. Les onglets privés dans Safari ont des cookies et un cache isolés qui sont détruits lorsque vous fermez l'onglet. Safari prend également en charge les profils qui sont équivalents aux conteneurs multi-comptes de Firefox pour séparer les cookies et la navigation. Safari peut être rendu considérablement plus sécurisé avec [le mode verrouillage](#lockdown-mode), qui peut être désactivé par site. En savoir plus sur [la prévention du suivi](https://webkit.org/tracking-prevention/) dans Safari.

Les extensions Web dans Safari ont une option supplémentaire pour utiliser du code natif dans l'environnement sandbox de Safari, en plus des API d'extension Web. Les extensions Web dans Safari sont également distribuées via l'App Store d'Apple. La soumission à l'App Store présente l'avantage supplémentaire que le code de l'extension Web est audité par Apple. D'un autre côté, la soumission à l'App Store a un coût élevé. Les frais d'[abonnement développeur](https://developer.apple.com/support/compare-memberships) annuels coûtent 100 USD (contrairement aux frais de 5 USD de Chrome et à la soumission gratuite de Firefox). Le coût élevé est prohibitif pour la majorité des développeurs open source. En conséquence, Safari a très peu d'extensions parmi lesquelles choisir. Cependant, gardez à l'esprit le coût élevé lors de l'installation d'extensions. On s'attend à ce que la plupart des extensions Web aient un moyen de monétiser l'utilisation pour couvrir les coûts de développement. Évitez les extensions Web sans code open source disponible pour examen.

Safari synchronise les préférences utilisateur et les mots de passe avec [iCloud Keychain](https://support.apple.com/HT202303). La visualisation d'un mot de passe enregistré nécessite une authentification avec le mot de passe du compte de l'appareil actuel ou une autre méthode d'authentification configurée.

Safari implémente les nouvelles fonctionnalités Web plus lentement que Chrome ou Firefox, mais les correctifs de sécurité sont livrés rapidement via les mises à jour système.

Voir aussi [el1t/uBlock-Safari](https://github.com/el1t/uBlock-Safari/wiki/Disable-hyperlink-auditing-beacon) pour désactiver les balises d'audit des hyperliens.

<a name="web-browser-privacy"></a>

## Confidentialité du navigateur Web

Les sites Web déduisent des informations sur [le navigateur et l'appareil](https://developer.mozilla.org/docs/Web/API/Navigator) à partir des paramètres, des polices, de la taille de l'écran, de la langue, de l'adresse IP et du comportement du navigateur. Ces informations contribuent à [l'empreinte du navigateur](https://en.wikipedia.org/wiki/Device_fingerprint).

Pour plus d'informations sur la navigation consciente de la sécurité et les données envoyées par le navigateur, voir les ressources [HowTo: Privacy & Security Conscious Browsing](https://gist.github.com/atcuno/3425484ac5cce5298932), [browserleaks.com](https://browserleaks.com/), [Am I Unique?](https://amiunique.org/fingerprint) et [EFF Cover Your Tracks](https://coveryourtracks.eff.org/).

Pour réduire le suivi intersites, bloquez [les cookies tiers](https://en.wikipedia.org/wiki/Third-party_cookies).

[WebRTC](https://en.wikipedia.org/wiki/WebRTC) peut exposer les adresses du réseau local et les adresses associées aux connexions VPN. Dans Firefox et Chrome/Chromium, cette fonctionnalité peut être désactivée avec [uBlock Origin](https://github.com/gorhill/uBlock/wiki/Prevent-WebRTC-from-leaking-local-IP-address). Dans Safari, [le mode verrouillage](#lockdown-mode) [désactive également WebRTC](https://www.sevarg.net/2022/07/20/ios16-lockdown-mode-browser-analysis).

# Tor

[Tor](https://en.wikipedia.org/wiki/Tor_(network)) est un réseau d'anonymat qui peut fournir une confidentialité supplémentaire pendant la navigation. Le navigateur Tor est un navigateur modifié basé sur Firefox configuré pour utiliser le réseau Tor.

Téléchargez le navigateur Tor depuis [Tor Project](https://www.torproject.org/download/), à la fois les fichiers `dmg` et `asc` pour [vérification](https://support.torproject.org/tor-browser/getting-started/verifying-tor-browser/) :

```console
$ cd ~/Downloads

$ file tor-browser-macos-*
tor-browser-macos-15.0.17.dmg:     XZ compressed data, checksum NONE
tor-browser-macos-15.0.17.dmg.asc: PGP signature Signature (old)

$ gpg --verify tor-browser-macos-*.asc
[...]
gpg: Can't check signature: No public key

$ gpg --auto-key-locate nodefault,wkd --locate-keys torbrowser@torproject.org
gpg: key 0x4E2C6E8793298290: public key "Tor Browser Developers (signing key) <torbrowser@torproject.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1
pub   rsa4096/0x4E2C6E8793298290 2014-12-15 [C] [expires: 2027-07-15]
      Key fingerprint = EF6E 286D DA85 EA2A 4BA7  DE68 4E2C 6E87 9329 8290
uid                   [ unknown] Tor Browser Developers (signing key) <torbrowser@torproject.org>
sub   rsa4096/0x157432CF78A65729 2024-07-15 [S] [expires: 2026-10-26]
      Key fingerprint = CAAE 408A EBE2 288E 96FC  5D5E 1574 32CF 78A6 5729

$ gpg --verify tor-browser-macos-*.asc
gpg: assuming signed data in 'tor-browser-macos-15.0.17.dmg'
gpg: Signature made Sun Jun 28 15:35:20 2026 PDT
gpg:                using RSA key CAAE408AEBE2288E96FC5D5E157432CF78A65729
gpg: Good signature from "Tor Browser Developers (signing key) <torbrowser@torproject.org>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: EF6E 286D DA85 EA2A 4BA7  DE68 4E2C 6E87 9329 8290
     Subkey fingerprint: CAAE 408A EBE2 288E 96FC  5D5E 1574 32CF 78A6 5729
```

Assurez-vous que `Good signature from "Tor Browser Developers (signing key) <torbrowser@torproject.org>"` apparaît dans la sortie. L'avertissement est attendu si vous n'avez pas vérifié indépendamment et fait confiance localement à la clé de signature ; cela ne signifie pas en soi que la signature est invalide.

Consultez [Comment puis-je vérifier la signature du navigateur Tor ?](https://support.torproject.org/tbb/how-to-verify-signature/) pour plus d'informations.

Pour terminer l'installation du navigateur Tor, ouvrez l'image disque et faites-la glisser dans le dossier Applications, ou utilisez les commandes :

```bash
hdiutil mount tor-browser-macos-15.0.17.dmg
cp -r /Volumes/Tor\ Browser/Tor\ Browser.app/ ~/Applications/
```

Vérifiez que l'application a été signée par l'Apple Developer ID de The Tor Project (`MADPSAYN6T`) :

```console
$ spctl --assess -vv ~/Applications/Tor\ Browser.app
/Users/user1/Applications/Tor Browser.app: accepted
source=Notarized Developer ID
origin=Developer ID Application: The Tor Project, Inc (MADPSAYN6T)

$ pkgutil --check-signature ~/Applications/Tor\ Browser.app
Package "Tor Browser.app":
   Status: signed by a certificate trusted by macOS
   Certificate Chain:
    1. Developer ID Application: The Tor Project, Inc (MADPSAYN6T)
       Expires: 2028-10-11 17:57:46 +0000
       SHA256 Fingerprint:
           76 3C 89 02 ED CB AD 8E 59 86 1E 93 D3 05 5B 28 F9 04 0C 96 03 8B
           16 28 9F 38 64 ED 53 45 B4 DA
       ------------------------------------------------------------------------
    2. Developer ID Certification Authority
       Expires: 2031-09-17 00:00:00 +0000
       SHA256 Fingerprint:
           F1 6C D3 C5 4C 7F 83 CE A4 BF 1A 3E 6A 08 19 C8 AA A8 E4 A1 52 8F
           D1 44 71 5F 35 06 43 D2 DF 3A
       ------------------------------------------------------------------------
    3. Apple Root CA
       Expires: 2035-02-09 21:40:36 +0000
       SHA256 Fingerprint:
           B0 B1 73 0E CB C7 FF 45 05 14 2C 49 F1 29 5E 6E DA 6B CA ED 7E 2C
           68 C5 BE 91 B5 A1 10 01 F0 24
```

La commande `codesign` peut également être utilisée pour examiner la signature de code d'une application :

```console
$ codesign -dvv ~/Applications/Tor\ Browser.app
Executable=/Users/user1/Applications/Tor Browser.app/Contents/MacOS/firefox
Identifier=org.torproject.torbrowser
Format=app bundle with Mach-O universal (x86_64 arm64)
CodeDirectory v=20500 size=805 flags=0x10000(runtime) hashes=14+7 location=embedded
Signature size=9054
Authority=Developer ID Application: The Tor Project, Inc (MADPSAYN6T)
Authority=Developer ID Certification Authority
Authority=Apple Root CA
Timestamp=Jun 28, 2026 at 14:01:57
Notarization Ticket=stapled
Info.plist entries=27
TeamIdentifier=MADPSAYN6T
Runtime Version=15.5.0
Sealed Resources version=2 rules=13 files=208
Internal requirements count=1 size=188
```

Pour inspecter les détails du certificat d'une application signée, extrayez les certificats intégrés avec `codesign` et décodez-les avec `openssl` :

```console
$ codesign -d --extract-certificates ~/Applications/Tor\ Browser.app
Executable=/Users/user1/Applications/Tor Browser.app/Contents/MacOS/firefox

$ file codesign*
codesign0: Certificate, Version=3
codesign1: Certificate, Version=3
codesign2: Certificate, Version=3 Certificate, Version=02

$ openssl x509 -inform der -in codesign0 -subject -issuer -startdate -enddate -noout
subject= /UID=MADPSAYN6T/CN=Developer ID Application: The Tor Project, Inc (MADPSAYN6T)/OU=MADPSAYN6T/O=The Tor Project, Inc/C=US
issuer= /CN=Developer ID Certification Authority/OU=G2/O=Apple Inc./C=US
notBefore=Oct 11 17:57:47 2023 GMT
notAfter=Oct 11 17:57:46 2028 GMT

$ openssl x509 -inform der -in codesign0 -fingerprint -noout
SHA256 Fingerprint=76:3C:89:02:ED:CB:AD:8E:59:86:1E:93:D3:05:5B:28:F9:04:0C:96:03:8B:16:28:9F:38:64:ED:53:45:B4:DA
```

Le navigateur Tor peut utiliser des méthodes de connexion spéciales appelées [transports enfichables](https://support.torproject.org/tor-browser/circumvention/) pour rendre le trafic plus difficile à identifier ou à bloquer. Cela peut être fait en configurant un [relais](https://support.torproject.org/relays/) ou en utilisant un [pont](https://bridges.torproject.org/) existant.

Le réseau Tor offre [l'anonymat](https://www.privateinternetaccess.com/blog/2013/10/how-does-privacy-differ-from-anonymity-and-why-are-both-important/), ce qui n'est pas nécessairement la même chose que la confidentialité. Le réseau ne se défend pas contre un observateur global capable d'analyse de trafic et de corrélation. Voir aussi [Seeking Anonymity in an Internet Panopticon](https://bford.info/pub/net/panopticon-cacm.pdf) et [Traffic Correlation on Tor by Realistic Adversaries](https://www.ohmygodel.com/publications/usersrouted-ccs13.pdf).

Consultez [Tor Protocol Specification](https://spec.torproject.org/tor-spec/) pour plus d'informations.

Voir aussi [Invisible Internet Project (I2P)](https://geti2p.net/en/about/intro) et sa [comparaison avec Tor](https://geti2p.net/en/comparison/tor).

# VPN

Choisissez un fournisseur VPN ou une configuration auto-hébergée avec un protocole moderne documenté et des clients bien audités. Évitez les protocoles obsolètes tels que [PPTP](https://en.wikipedia.org/wiki/Point-to-Point_Tunneling_Protocol#Security) en faveur de [WireGuard](https://www.wireguard.com/) ou [OpenVPN](https://en.wikipedia.org/wiki/OpenVPN).

Il peut être utile de considérer l'emplacement géographique du fournisseur VPN. Voir la discussion plus approfondie dans [issue 114](https://github.com/drduh/macOS-Security-and-Privacy-Guide/issues/114).

## WireGuard

[WireGuard](https://www.wireguard.com/) est un protocole VPN avec une base de code réduite et une conception cryptographique [formellement vérifiée](https://www.wireguard.com/formal-verification/), plus simple à configurer et à auditer que IPsec ou OpenVPN.

Installez l'[application officielle WireGuard](https://apps.apple.com/us/app/wireguard/id1451685025) depuis l'App Store pour gérer les tunnels depuis la barre des menus, ou installez les outils en ligne de commande avec Homebrew :

```bash
brew install wireguard-tools
```

Générez une paire de clés pour le client :

```bash
umask 077

wg genkey | tee privatekey | wg pubkey > publickey
```

Créez une configuration de tunnel, en remplaçant les clés et adresses par les valeurs de votre propre serveur ou fournisseur :

```
[Interface]
PrivateKey = <contents of privatekey>
Address = 10.8.0.2/32
DNS = 10.8.0.1

[Peer]
PublicKey = <server public key>
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = vpn.example.org:51820
```

`AllowedIPs = 0.0.0.0/0, ::/0` achemine tout le trafic IPv4 et IPv6 via le tunnel ; des plages plus étroites permettent le split tunneling. Consultez [wg-quick(8)](https://git.zx2c4.com/wireguard-tools/about/src/man/wg-quick.8) pour toutes les options de configuration.

Importez le fichier de configuration dans l'app WireGuard et activez **À la demande** afin que le tunnel s'active dès que le Wi-Fi ou l'Ethernet est utilisé. Vous pouvez également activer le tunnel avec les outils en ligne de commande :

```bash
sudo wg-quick up /path/to/wg0.conf
```

Vérifiez que le tunnel est actif :

```bash
sudo wg show
```

Pour héberger vous-même un serveur WireGuard, consultez [trailofbits/algo](https://github.com/trailofbits/algo).

Certains clients VPN peuvent autoriser le trafic à sortir par une autre interface réseau si la connexion VPN tombe ou est interrompue. Consultez [scy/8122924](https://gist.github.com/scy/8122924) pour un exemple sur la façon d'autoriser le trafic uniquement via le VPN, ou utilisez des [règles pf](#packet-filter) pour restreindre le trafic à l'interface du tunnel.

Consultez également cet [aperçu technique](https://blog.timac.org/2018/0717-macos-vpn-architecture/) du client VPN L2TP/IPsec intégré à macOS et du client [IKEv2](https://en.wikipedia.org/wiki/IKEv2).

# PGP/GPG

PGP est une norme de chiffrement et de signature de données, en particulier pour le courrier électronique. Elle peut protéger le contenu des messages entre correspondants qui échangent et vérifient correctement les clés, mais elle ne protège pas les métadonnées telles que les destinataires et les objets des courriels.

GPG (GNU Privacy Guard) est un programme open source sous licence GPL conforme à la norme PGP. Il peut vérifier les signatures logicielles et chiffrer des fichiers [symétriquement](https://en.wikipedia.org/wiki/Symmetric-key_algorithm) ou à l'aide de [clés publiques](https://en.wikipedia.org/wiki/Public-key_cryptography).

Installez GnuPG avec Homebrew (`brew install gnupg`) ou installez [GPG Suite](https://gpgtools.org/).

Téléchargez [gpg.conf](https://github.com/drduh/YubiKey-Guide/blob/main/config/gpg.conf) pour utiliser les paramètres recommandés :

```bash
curl -o ~/.gnupg/gpg.conf \
  https://raw.githubusercontent.com/drduh/YubiKey-Guide/main/config/gpg.conf
```

Consultez [drduh/YubiKey-Guide](https://github.com/drduh/YubiKey-Guide) pour générer et gérer les identifiants.

<a name="email"></a>

# Courrier électronique

Le courrier électronique n'est pas conçu pour offrir une confidentialité forte par défaut : le contenu des messages peut être conservé par les fournisseurs de services, copié dans les boîtes aux lettres des destinataires, transféré ou exposé par la compromission d'un compte. Les métadonnées (y compris le destinataire, l'objet, les horodatages et les informations du serveur de messagerie) restent généralement visibles même lorsque le contenu du message est chiffré.

## Thunderbird

[Thunderbird](https://www.thunderbird.net/) est un client de messagerie gratuit et open source avec prise en charge standard de [IMAP](https://en.wikipedia.org/wiki/Internet_Message_Access_Protocol), [POP](https://en.wikipedia.org/wiki/Post_Office_Protocol), [CalDAV](https://en.wikipedia.org/wiki/CalDAV) et [CardDAV](https://en.wikipedia.org/wiki/CardDAV). C'est un choix approprié pour accéder au courrier et conserver des copies locales plutôt que de dépendre exclusivement du serveur distant d'un fournisseur.

Thunderbird inclut la prise en charge du chiffrement des courriels [OpenPGP](https://support.mozilla.org/kb/openpgp-thunderbird-howto-and-faq), qui peut protéger le contenu des messages et fournir des [signatures](https://www.gnupg.org/gph/en/manual.html#AEN136) cryptographiques. Vérifiez toujours les empreintes des clés publiques par un canal indépendant avant de vous fier à une clé pour des communications sensibles.

La [fonction de messages archivés](https://support.mozilla.org/kb/archived-messages) peut déplacer les messages hors des serveurs de messagerie distants vers un **Dossier local**, améliorant ainsi la confidentialité.

<a name="messengers"></a>

# Messageries

## XMPP

[XMPP](https://en.wikipedia.org/wiki/XMPP) est un [protocole ouvert](https://xmpp.org/extensions/) développé par l'[IETF](https://www.ietf.org/) qui prend en charge la messagerie fédérée multiplateforme. Il existe de nombreuses [options de clients](https://xmpp.org/getting-started/). Envisagez d'utiliser l'un des clients basés sur navigateur pour profiter du bac à sable du navigateur.

Selon le fournisseur, vous n'aurez peut-être besoin que d'un nom d'utilisateur et d'un mot de passe pour créer un compte.

XMPP n'est pas chiffré de bout en bout (E2EE) par défaut ; utilisez [OMEMO](https://omemo.top/) avec un client compatible.

## Signal

[Signal](https://signal.org/) est une messagerie E2EE populaire dont le protocole [double-ratchet](https://signal.org/docs/specifications/doubleratchet/) est utilisé par de nombreuses autres applications, notamment WhatsApp, Google Messages et Facebook Messenger.

Pour utiliser l'application de bureau Signal, Signal doit d'abord être installé sur un téléphone.

Consultez le [guide Signal](https://ssd.eff.org/module/how-to-use-signal) de l'EFF pour des conseils supplémentaires.

## iMessage

[iMessage](https://en.wikipedia.org/wiki/IMessage) est la messagerie propriétaire d'Apple. Elle nécessite un [compte Apple](#apple-account) pour être utilisée.

Activez la [vérification de la clé de contact](https://support.apple.com/118246) et vérifiez vos contacts.

iMessage peut être utilisé avec un [numéro de téléphone ou une adresse e-mail](https://support.apple.com/108758#help).

> [!WARNING]
> Par défaut, la sauvegarde iCloud est activée, ce qui stocke des copies des clés de chiffrement des messages sur [les serveurs d'Apple](https://support.apple.com/102651) sans E2EE. [Désactivez la sauvegarde iCloud](https://support.apple.com/guide/icloud/view-and-manage-backups-mm122d3ef202/1.0/icloud/1.0) ou activez la [Protection avancée des données](https://support.apple.com/guide/security/advanced-data-protection-for-icloud-sec973254c5f) pour éviter cela. Rappelez aux destinataires des messages de faire de même.

<a name="malware"></a>

# Logiciels malveillants

Consultez [Méthodes de persistance des logiciels malveillants sur Mac OS X](https://www.virusbulletin.com/uploads/pdf/conference/vb2014/VB2014-Wardle.pdf) pour découvrir comment les logiciels malveillants courants de macOS persistent.

Pour des exemples de capacités avancées des logiciels malveillants macOS, consultez le [matériel Hacking Team](https://www.schneier.com/blog/archives/2015/07/hacking_team_is.html). Pour une analyse plus approfondie, consultez [l'installation root pour MacOS](https://github.com/hackedteam/vector-macos-root), [le pilote de support pour Mac Agent](https://github.com/hackedteam/driver-macos) et [RCS Agent pour Mac](https://github.com/hackedteam/core-macos).

Consultez également [A Brief Analysis of an RCS Implant Installer](https://objective-see.com/blog/blog_0x0D.html) et [reverse.put.as](https://reverse.put.as/2016/02/29/the-italian-morons-are-back-what-are-they-up-to-this-time/).

<a name="downloading-software"></a>

## Téléchargement de logiciels

Les applications provenant de l'App Store ou [notariées par Apple](https://support.apple.com/guide/security/app-code-signing-process-sec3ad8e6e53/web) peuvent réduire le risque de logiciels malveillants, mais aucun de ces processus ne garantit la sécurité. Apple effectue des vérifications automatisées sur les applications notariées, et les applications de l'App Store [font l'objet d'une évaluation](https://developer.apple.com/app-store/review/guidelines/). Sinon, obtenez des logiciels auprès de sources identifiées par le développeur comme officielles, et vérifiez que le site web ou le téléchargement en ligne de commande utilise HTTPS.

<a name="app-sandbox"></a>

## Bac à sable d'app

Vérifiez si une application utilise le [bac à sable d'app](https://developer.apple.com/documentation/security/app_sandbox/protecting_user_data_with_app_sandbox) :

```bash
codesign --display --entitlements - \
  /System/Applications/Calculator.app 2>&1 |
    grep -A2 com.apple.security.app-sandbox
```

Lorsque le bac à sable d'app est activé, la sortie devrait être :

```console
[Key] com.apple.security.app-sandbox
[Value]
    [Bool] true
```

**Moniteur d'activité** peut également indiquer s'il est activé avec la colonne « Bac à sable » affichée :

```bash
defaults write com.apple.ActivityMonitor "UserColumnsPerTab v6.0" \
  -dict-add 0 '(Command, CPUUsage, CPUTime, Threads, IdleWakeUps, Architecture, GPUUsage, GPUTime, PID, UID, Sandbox, restricted)'
```

Les logiciels de l'App Store sont tenus d'utiliser le bac à sable d'app. Des applications comme Google Chrome utilisent leur propre [bac à sable](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/design/sandbox.md) et peuvent ne pas utiliser le bac à sable d'app.

> [!NOTE]
> Le bac à sable limite l'accès par défaut d'une application, mais les droits et les autorisations accordées par l'utilisateur peuvent étendre l'accès.

<a name="hardened-runtime"></a>

## Environnement d'exécution renforcé

Vérifiez si une application utilise l'[environnement d'exécution renforcé](https://developer.apple.com/documentation/security/hardened_runtime) :

```bash
codesign --display --verbose /path/to/application.app
```

Si l'environnement d'exécution renforcé est activé, `flags=0x10000(runtime)` apparaîtra dans la sortie.

Listez toutes les applications avec cette fonctionnalité activée :

```bash
for app in /Applications/*.app /System/Applications/*.app; do
  if codesign -dvv "$app" 2>&1 | grep -q 'flags=0x10000(runtime)'; then
    echo "$app"
  fi
done
```

**Moniteur d'activité** propose l'option d'afficher une colonne « Restreint » qui indique qu'un programme est empêché d'injecter du code via l'[éditeur de liens dynamique](https://pewpewthespells.com/blog/blocking_code_injection_on_ios_and_os_x.html) de macOS.

L'environnement d'exécution renforcé est un prérequis pour la notarisation des applications distribuées.

## Antivirus

Pour analyser des fichiers et des applications, envisagez de les téléverser sur [VirusTotal](https://www.virustotal.com/gui/home/upload), en gardant à l'esprit que cela les rend publiquement visibles.

macOS inclut un logiciel antivirus intégré appelé [XProtect](https://support.apple.com/guide/security/protecting-against-malware-sec469d47bd8), qui s'exécute en arrière-plan et met à jour automatiquement les signatures utilisées pour détecter les logiciels malveillants. Si un logiciel malveillant est détecté, XProtect tente de le supprimer et de le mettre en quarantaine.

Vérifiez l'état de mise à jour de XProtect :

```bash
xprotect version
```

Des applications comme [BlockBlock](https://objective-see.com/products/blockblock.html) ou [hazcod/maclaunch](https://github.com/hazcod/maclaunch) peuvent aider à prévenir ou détecter les logiciels malveillants persistants.

Les logiciels antivirus peuvent aider à détecter les malwares courants, mais ils peuvent aussi augmenter la surface d'attaque car ils s'exécutent souvent avec des privilèges système étendus. Certains produits peuvent également envoyer des données de télémétrie ou des échantillons au fournisseur.

Voir [Sophail: Applied attacks against Antivirus](https://lock.cmpxchg8b.com/sophailv2.pdf), [Analysis and Exploitation of an ESET Vulnerability](https://googleprojectzero.blogspot.ro/2015/06/analysis-and-exploitation-of-eset.html), [Popular Security Software Came Under Relentless NSA and GCHQ Attacks](https://theintercept.com/2015/06/22/nsa-gchq-targeted-kaspersky/), et [How Israel Caught Russian Hackers Scouring the World for U.S. Secrets](https://www.nytimes.com/2017/10/10/technology/kaspersky-lab-israel-russia-hacking.html).

## Gatekeeper

[Gatekeeper](https://support.apple.com/guide/security/gatekeeper-and-runtime-protection-sec5599b66df/web) vérifie la notarisation et la provenance des logiciels.

Gatekeeper avertit lors de l'ouverture d'une application non notarisée. Il peut être contourné en sélectionnant l'application listée dans **Réglages Système** > **Confidentialité et sécurité** après une tentative échouée.

Vérifier si Gatekeeper est activé :

```bash
spctl --status --verbose
```

<a name="system-integrity-protection"></a>

# Protection de l'intégrité du système

Pour vérifier que la Protection de l'intégrité du système est activée, utilisez la commande `csrutil status`, qui doit renvoyer : `System Integrity Protection status: enabled.` Sinon, [activez SIP](https://developer.apple.com/documentation/security/disabling_and_enabling_system_integrity_protection) en utilisant le [mode de récupération](https://support.apple.com/102518).

<a name="metadata-and-artifacts"></a>

# Métadonnées et artefacts

macOS attache des métadonnées ([attributs étendus APFS](https://en.wikipedia.org/wiki/Extended_file_attributes#macOS)) aux fichiers.

Certains attributs de métadonnées peuvent être consultés et supprimés avec les commandes [`mdls`](https://ss64.com/mac/mdls.html) et [`xattr`](https://ss64.com/mac/xattr.html). Les fichiers de préférences (`plist`) peuvent être lus et modifiés avec [`defaults`](https://ss64.com/mac/defaults.html).

Les métadonnées système et d'application se trouvent dans :

- `~/Library/Preferences/`
- `~/Library/Containers/<APP>/Data/Library/Preferences`
- `/Library/Preferences`

Informations Bluetooth :

- `/Library/Preferences/com.apple.Bluetooth.plist`

Cache des travaux d'impression :

- `/var/spool/cups`

Appareils iOS précédemment connectés :

- `/Users/$USER/Library/Preferences/com.apple.iPod.plist`
- `/Library/Preferences/com.apple.iPod.plist`

Préférences du Finder :

- `~/Library/Preferences/com.apple.finder.plist`
- `~/Library/Preferences/com.apple.sidebarlists.plist`

Données de diagnostic :

- `/var/db/CoreDuet`
- `/var/db/diagnostics`
- `/var/db/systemstats`
- `/var/db/uuidtext`
- `/var/log/DiagnosticMessages`

Suggestions d'orthographe et de langue :

- `~/Library/LanguageModeling`
- `~/Library/Spelling`
- `~/Library/Suggestions`

État de l'application :

- `~/Library/Saved\ Application\ State`
- `~/Library/Containers/<APPNAME>/Data/Library/Saved\ Application\ State`

Métadonnées de sauvegarde automatique :

- `~/Library/Containers/<APP>/Data/Library/Autosave\ Information`
- `~/Library/Autosave\ Information`

Données analytiques Siri :

- `~/Library/Assistant/SiriAnalytics.db`

Historique de QuickTime Player :

- `~/Library/Containers/com.apple.QuickTimePlayerX/Data/Library/Preferences/com.apple.QuickTimePlayerX.plist`

Métadonnées diverses :

- `~/Library/Containers/com.apple.appstore/Data/Library/Preferences/com.apple.commerce.knownclients.plist`
- `~/Library/Preferences/com.apple.commerce.plist`
- `~/Library/Preferences/com.apple.QuickTimePlayerX.plist`

<a name="authentication"></a>

# Authentification

L'[app Mots de passe](https://support.apple.com/guide/passwords/the-passwords-app-mchl901b1b95/mac) crée des [identifiants sécurisés](https://support.apple.com/guide/security/automatic-strong-passwords-secc84c811c4/web). Elle prend en charge les [clés d'accès](https://fidoalliance.org/passkeys/), qui sont des identifiants conçus pour résister au hameçonnage lorsqu'ils sont utilisés avec des services compatibles.

[KeePassXC](https://keepassxc.org/) est un gestionnaire de mots de passe open source et multiplateforme à envisager. Il prend en charge l'authentification forte avec des jetons matériels compatibles et une extension de navigateur pour saisir automatiquement les identifiants.

Des mots de passe mémorables peuvent être créés avec [Diceware](https://secure.research.vt.edu/diceware/).

Assurez-vous que les comptes en ligne ont l'[authentification multifacteur](https://en.wikipedia.org/wiki/Multi-factor_authentication) activée. Privilégiez une authentification résistante au hameçonnage, comme des jetons matériels ou des clés d'accès avec les services pris en charge. Les applications d'authentification sont préférables aux codes SMS, mais toute MFA est meilleure qu'une authentification par mot de passe seul.

[YubiKey](https://www.yubico.com/products/) est un jeton d'authentification matériel populaire. Il peut également stocker des clés pour le chiffrement, la signature et les tâches d'authentification - voir [drduh/YubiKey-Guide](https://github.com/drduh/YubiKey-Guide).

GnuPG peut également gérer les mots de passe et d'autres fichiers chiffrés - voir [drduh/Purse](https://github.com/drduh/Purse) et [drduh/pwd.sh](https://github.com/drduh/pwd.sh).

<a name="backup"></a>

# Sauvegarde

Chiffrez les fichiers localement avant de les sauvegarder sur des supports externes ou des services en ligne.

Suivez le [modèle de sauvegarde 3-2-1](https://www.cisa.gov/sites/default/files/publications/data_backup_options.pdf) : conservez trois copies des données importantes, stockez-les sur au moins deux types de supports différents et gardez au moins une copie dans un emplacement physique distinct.

## Time Machine

[Time Machine](https://support.apple.com/104984) est l'outil intégré pour gérer les sauvegardes sur macOS. Utilisez un disque externe ou un stockage réseau pour créer des sauvegardes [chiffrées](https://support.apple.com/guide/mac-help/keep-your-time-machine-backup-disk-secure-mh21241).

## GnuPG

[GnuPG](https://www.gnupg.org/) peut être utilisé avec un mot de passe ou une clé publique, la clé privée étant stockée sur [YubiKey](https://github.com/drduh/YubiKey-Guide).

Compressez et chiffrez un répertoire à l'aide d'un mot de passe :

```bash
tar zcvf - ~/Downloads |
  gpg -c > ~/Downloads/backup-$(date +%F-%H%M%S).tar.gz.gpg
```

Déchiffrer et décompresser le répertoire :

```bash
gpg -o ~/Downloads/decrypted-backup.tar.gz -d ~/Downloads/backup-*.tar.gz.gpg
tar zxvf ~/Downloads/decrypted-backup.tar.gz
```

<a name="disk-utility"></a>

## Utilitaire de disque

Des volumes chiffrés peuvent être créés à l'aide de [Utilitaire de disque](https://support.apple.com/guide/disk-utility/toc) ou `hdiutil` :

```bash
hdiutil create ~/Downloads/encrypted.dmg -encryption -size 50M -volname "secretStuff"
hdiutil mount ~/Downloads/encrypted.dmg
cp -v ~/Documents/passwords.txt /Volumes/secretStuff
hdiutil eject /Volumes/secretStuff
```

<a name="other"></a>

## Autres

[Tresorit](https://tresorit.com/) et [restic](https://restic.net/) peuvent également présenter un intérêt.

# Wi-Fi

Les noms de réseaux Wi-Fi, appelés SSID, et le comportement de sondage des appareils peuvent révéler des informations sur les réseaux à proximité et utilisés précédemment. [Les réseaux masqués](https://support.apple.com/guide/security/wi-fi-privacy-with-apple-devices-sec31e483abf/web#sec059998a98) n'offrent pas de confidentialité significative et peuvent inciter les appareils à les sonder activement.

Définissez une [adresse Wi-Fi privée](https://support.apple.com/guide/mac-help/use-a-private-wi-fi-address-on-mac-mchlb1cb3eb4/mac) pour réduire le suivi réseau.

Définissez la sécurité du réseau sans fil sur [WPA3](https://en.wikipedia.org/wiki/WPA3#WPA3). Suivez les [recommandations d'Apple](https://support.apple.com/102766) pour définir les réglages recommandés pour les routeurs et les points d'accès.

# SSH

Pour les connexions SSH distantes, utilisez une clé SSH unique protégée par une phrase de passe ou un jeton matériel, vérifiez les clés d'hôte du serveur et évitez l'authentification par mot de passe. Consultez [drduh/config/ssh_config](https://github.com/drduh/config/blob/main/ssh_config) pour les options client recommandées.

Un tunnel SSH peut acheminer en toute sécurité le trafic via un autre ordinateur, de manière similaire à un VPN. Pour utiliser Privoxy exécuté sur un hôte distant sur le port 8118 :

```bash
ssh -C -L 5555:127.0.0.1:8118 you@remote-host.tld
sudo networksetup -setwebproxy "Wi-Fi" 127.0.0.1 5555
sudo networksetup -setsecurewebproxy "Wi-Fi" 127.0.0.1 5555
```

Ou pour utiliser une connexion SSH comme [proxy SOCKS](https://www.mikeash.com/ssh_socks.html) :

```bash
ssh -NCD 3000 you@remote-host.tld
```

Par défaut, macOS n'a pas [l'ouverture de session à distance](https://support.apple.com/guide/mac-help/allow-a-remote-computer-to-access-your-mac-mchlp1066/mac) (serveur SSH) activée.

Pour activer SSH et autoriser les connexions entrantes, utilisez **Réglages Système** > **Général** > **Partage** ou la commande :

```bash
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```

Désactivez l'authentification par mot de passe et envisagez de [renforcer](https://stribika.github.io/2015/01/04/secure-secure-shell.html) davantage la configuration du serveur SSH. Consultez [drduh/config/sshd_config](https://github.com/drduh/config/blob/main/sshd_config) pour les options recommandées.

Confirmez que le serveur SSH est en cours d'exécution :

```bash
sudo lsof -Pni TCP:22
```

<a name="physical-access"></a>

# Accès physique

Ne laissez pas l'ordinateur sans surveillance dans des lieux dangereux. Un attaquant expérimenté disposant d'un accès physique non supervisé pourrait installer un [enregistreur de frappe matériel](https://trmm.net/Thunderstrike_31c3) pour enregistrer les frappes au clavier, y compris les mots de passe. L'utilisation d'un Mac avec un clavier intégré ou un clavier Bluetooth rend cela plus difficile, car de nombreuses versions prêtes à l'emploi de cette attaque sont conçues pour être branchées entre un clavier USB et l'ordinateur.

Pour réduire les conséquences d'un vol pendant qu'un Mac est utilisé, envisagez d'utiliser [buskill/buskill-app](https://github.com/buskill/buskill-app) ou [Lennolium/swiftGuard](https://github.com/Lennolium/swiftGuard).

Envisagez d'acheter un filtre de confidentialité pour écran lors d'une utilisation en public.

[Du vernis à ongles](https://trmm.net/Glitter) et des sceaux d'inviolabilité peuvent être appliqués sur les composants afin de détecter toute altération.

<a name="monitoring"></a>

# Surveillance

<a name="logs"></a>

## Journaux

Surveillez les journaux système avec [Console](https://support.apple.com/guide/console/toc) ou la commande `log stream`.

Affichez tous les journaux système des 5 dernières minutes au format syslog classique :

```bash
log show --last 5m --style syslog
```

Sortie sous forme de tableau JSON :

```bash
log show --last 5m --style json
```

Sortie JSON délimitée par des retours à la ligne (un objet JSON par ligne) ; requis pour [`jq`](https://jqlang.org/) :

```bash
log show --last 5m --style ndjson
```

Filtrez les messages par sous-système, par exemple depuis `com.apple.SoftwareUpdate` :

```bash
log show --last 1h --style json \
  --predicate 'subsystem == "com.apple.SoftwareUpdate"'
```

Filtrez les messages par contenu, par exemple contenant `failed` :

```bash
log show --last 1h --predicate 'eventMessage CONTAINS "failed"'
```

Filtrage des messages sans distinction de casse :

```bash
log show --last 1h --predicate 'eventMessage CONTAINS[c] "error"'
```

Comptez les messages d'erreur distincts avec `jq` :

```bash
log show --last 1h --style ndjson \
  --predicate 'eventMessage CONTAINS[c] "error"' |
  jq -r '.eventMessage' | sort | uniq -c | sort -nr | head -100
```

Exportez les journaux vers une archive, qui peut être ouverte avec Console :

```bash
log collect --last 1h --output ~/Downloads/logs-$(date +%F-%H%M).logarchive
```

## DTrace

[System Integrity Protection](https://github.com/drduh/macOS-Security-and-Privacy-Guide#system-integrity-protection) interfère avec [DTrace](https://en.wikipedia.org/wiki/DTrace) et doit être [partiellement désactivé](https://eclecticlight.co/2024/08/21/controlling-system-integrity-protection-using-csrutil-a-reference/) avant utilisation.

- `iosnoop` surveille les E/S disque
- `opensnoop` surveille les ouvertures de fichiers
- `execsnoop` surveille les processus
- `errinfo` surveille les appels système échoués
- `dtruss` surveille tous les appels système

Voir `man -k dtrace` pour plus d'informations.

<a name="processes"></a>

## Processus

Lister les processus en cours d'exécution avec [Moniteur d'activité](https://support.apple.com/guide/activity-monitor/toc) ou la commande `ps`.

<a name="endpoint-security"></a>

## Sécurité des points de terminaison

Inspecter l'exécution des processus en temps réel avec `eslogger` et [`jq`](https://jqlang.org/) :

```bash
printf 'TIME\t\t\t\tPID\tPPID\tUID\tCOMMAND\n'
sudo eslogger exec | jq -r '
  [ .time,
    .process.audit_token.pid,
    .process.ppid,
    .process.audit_token.euid,
   (.event.exec.args | join(" "))
  ] | @tsv'
```

Formater et imprimer les événements en JSON, en les enregistrant également dans un fichier journal daté pour une analyse ultérieure :

```bash
mkdir ~/eslogs
sudo eslogger exec | jq -c --unbuffered '
  {
    time: .time,
    pid:  .process.audit_token.pid,
    ppid: .process.ppid,
    uid:  .process.audit_token.euid,
    sys:  .process.is_platform_binary,
    team: .process.team_id,
    sign: .process.signing_id,
    path: .process.executable.path,
    cmd:  .event.exec.args
  }' |
  tee -a ~/eslogs/exec-$(hostname)-$(date +%Y%m%d%H%M).log
```

Inspecter les journaux pour toutes les commandes `curl` :

```bash
jq 'select(.cmd | index("curl")) | .cmd | join(" ")' ~/eslogs/exec-*.log
```

Inclure les identifiants d'utilisateur et d'application exec, trier par nombre :

```bash
jq -s --arg command "curl" '
  map(
    select(.cmd | index($command))
    | {
        uid,
        aid: (.team // .sign),
        cmd: (.cmd | join(" "))
      }
  )
  | group_by([.uid, .aid, .cmd])
  | map({
      cnt: length,
      uid: .[0].uid,
      aid: .[0].aid,
      cmd: .[0].cmd
    })
  | sort_by(-.cnt)
' ~/eslogs/exec-*.log
```

**Nouveau !** [drduh/mess](https://github.com/drduh/mess) - un système de sécurité gratuit et open source pour votre Mac personnel basé sur `eslogger`.

<a name="install-history"></a>

## Historique des installations

Afficher l'historique d'installation des paquets et des mises à jour système :

```bash
system_profiler SPInstallHistoryDataType
```

Formaté avec [`jq`](https://jqlang.org/) :

```bash
system_profiler SPInstallHistoryDataType -json |
  jq -r '.SPInstallHistoryDataType[]
    | [(._name            // "n/a"),
       (.install_version  // "n/a"),
       (.install_date     // "n/a"),
       (.package_source   // "n/a")]
    | @tsv' | column -t -s $'\t'
```

L'historique des mises à jour logicielles peut également être affiché avec :

```bash
softwareupdate --history
```

<a name="network"></a>

## Réseau

Lister les connexions réseau ouvertes avec [`lsof`](https://www.unix.com/man_page/osx/8/lsof/) :

```bash
sudo lsof -Pni
```

Afficher les statistiques des protocoles réseau avec [`netstat`](https://www.unix.com/man_page/osx/1/netstat/) :

```bash
sudo netstat -n -p tcp
sudo netstat -n -p udp | sort | uniq
```

Surveiller les sockets TCP/UDP et la table de routage avec [`nettop`](https://www.unix.com/man_page/osx/1/nettop/) :

```bash
nettop -m tcp
nettop -m udp
nettop -m route
```

### Wireshark

[Wireshark](https://www.wireshark.org/) peut être utilisé en ligne de commande avec [`tshark`](https://www.wireshark.org/docs/man-pages/tshark.html).

Remplacez `en0` par l'interface active affichée par `networksetup -listallhardwareports`.

> [!IMPORTANT]
> Capturez le trafic uniquement sur les réseaux et systèmes que vous êtes autorisé à surveiller.

Surveiller le DNS :

```bash
/Applications/Wireshark.app/Contents/MacOS/tshark -i en0 \
  -Y "dns.flags.response == 1" -Tfields \
  -e frame.time_delta \
  -e dns.qry.name \
  -e dns.a \
  -Eseparator=,
```

Surveiller le HTTP :

```bash
/Applications/Wireshark.app/Contents/MacOS/tshark -i en0 \
  -Y "http.request or http.response" -Tfields \
  -e ip.dst \
  -e http.request.full_uri \
  -e http.request.method \
  -e http.response.code \
  -e http.response.phrase \
  -Eseparator=/s
```

Surveiller les certificats TLS :

```bash
/Applications/Wireshark.app/Contents/MacOS/tshark -i en0 \
  -Y "ssl.handshake.certificate" -Tfields \
  -e ip.src \
  -e x509sat.uTF8String \
  -e x509sat.printableString \
  -e x509sat.universalString \
  -e x509sat.IA5String \
  -e x509sat.teletexString \
  -Eseparator=/s -Equote=d
```

<a name="miscellaneous"></a>

# Divers

<a name="screensaver"></a>

## Économiseur d'écran

Configurer l'écran pour qu'il se verrouille dès que l'économiseur d'écran démarre :

```bash
defaults write com.apple.screensaver askForPassword -int 1
defaults write com.apple.screensaver askForPasswordDelay -int 0
```

<a name="diagnostic-data"></a>

## Données de diagnostic

Désactiver [Données de diagnostic et d'utilisation](https://support.apple.com/guide/mac-help/share-analytics-information-mac-apple-mh27990).

Désactiver le rapporteur de plantage (la boîte de dialogue qui apparaît après le plantage d'une application et propose de signaler le problème à Apple) :

```bash
defaults write com.apple.CrashReporter DialogType none
```

<a name="media-player"></a>

## Lecteur multimédia

Utiliser [QuickTime Player](https://en.wikipedia.org/wiki/Quicktime_player), l'application multimédia intégrée, pour lire les fichiers musicaux et vidéo. Elle utilise [App Sandbox](https://developer.apple.com/documentation/security/app_sandbox/protecting_user_data_with_app_sandbox), [Hardened Runtime](https://developer.apple.com/documentation/xcode/configuring-the-hardened-runtime), et bénéficie du [Signed System Volume](https://support.apple.com/guide/security/signed-system-volume-security-secd698747c9/web) dans le cadre du système de base.

<a name="file-handlers"></a>

## Gestionnaires de fichiers

Gérer les [gestionnaires de fichiers par défaut](https://support.apple.com/guide/mac-help/choose-an-app-to-open-a-file-on-mac-mh35597) pour réduire le risque d'ouverture de certains types de fichiers.

Modifier l'application par défaut pour les fichiers de script afin que leur ouverture affiche leur contenu au lieu de les exécuter.

Dans le Finder, localisez et sélectionnez un fichier .sh, faites un clic droit dessus et sélectionnez Lire les informations ou appuyez sur <kbd>Command</kbd> + <kbd>I</kbd>. Dans la section « Ouvrir avec », sélectionnez TextEdit dans le menu déroulant. S'il n'est pas répertorié, sélectionnez « Autre... » puis Applications > TextEdit.app. Sélectionnez « Tout modifier... » et confirmez en sélectionnant Continuer.

<a name="finder-options"></a>

## Options du Finder

Afficher les fichiers cachés et le dossier Bibliothèque dans le Finder :

```bash
defaults write com.apple.finder AppleShowAllFiles -bool true
chflags nohidden ~/Library
```

Afficher toutes les extensions de nom de fichier :

```bash
defaults write NSGlobalDomain AppleShowAllExtensions -bool true
```

Ne pas enregistrer les documents dans iCloud par défaut :

```bash
defaults write NSGlobalDomain NSDocumentSaveNewDocumentsToCloud -bool false
```

<a name="custom-umask"></a>

## umask personnalisé

Définir un [umask personnalisé](https://support.apple.com/101914) :

```bash
sudo launchctl config user umask 077
```

Redémarrez, puis créez un fichier/répertoire et vérifiez les autorisations (la valeur par défaut de macOS autorise l'accès en lecture 'group/other') :

```console
$ ls -ld umask*
drwx------@ 2 user1 staff  64 Jul 26 12:00 umask.dir
-rw-------@ 1 user1 staff  32 Jul 26 12:00 umask.txt
```

<a name="keyboard-entry"></a>

## Saisie au clavier

Activer la [saisie sécurisée au clavier](https://support.apple.com/guide/terminal/use-secure-keyboard-entry-trml109) dans Terminal (cela peut interférer avec des applications telles que [TextExpander](https://smilesoftware.com/textexpander/secure-input)).

<a name="network-hardening"></a>

## Renforcement du réseau

Désactiver les [annonces multicast Bonjour](https://www.tenable.com/audits/items/CIS_Apple_macOS_10.13_v1.1.0_Level_2.audit:d9dcee7e4d2b8d2ee54f437158992d88) (cela désactive également les fonctionnalités AirPlay et AirPrint) :

```bash
sudo defaults write /Library/Preferences/com.apple.mDNSResponder \
  NoMulticastAdvertisements -bool YES
```

[Désactiver Handoff](https://support.apple.com/guide/mac-help/change-airdrop-handoff-settings-mchl6a407f99) et les fonctionnalités [Bluetooth](https://support.apple.com/guide/mac-help/turn-bluetooth-on-or-off-blth1008).

## Sudoers

macOS inclut cette ligne dans `/etc/sudoers` :

```bash
Defaults env_keep += "HOME MAIL"
```

Cela empêche sudo de modifier HOME lorsque les privilèges sont élevés. Par conséquent, les programmes exécutés avec sudo peuvent utiliser les fichiers de configuration du répertoire personnel de l'utilisateur appelant. Envisagez de supprimer `HOME` de `env_keep`, sauf si ce comportement est spécifiquement requis.

Pour conserver la commodité d'un utilisateur root disposant du répertoire personnel d'un utilisateur non root, ajoutez une ligne export à `/var/root/.zshrc`, par exemple :

```bash
export HOME=/Users/user1
```

<a name="related-software"></a>

# Logiciels associés

Logiciel | Catégorie | Description
:-: | :-: | :-
[lynis](https://github.com/CISOfy/lynis) | Scanner d'audit | Outil d'audit de sécurité multiplateforme qui vérifie la configuration du système et suggère des améliorations de durcissement.
[zentral](https://github.com/zentralopensource/zentral) | Plateforme de surveillance et de configuration | Serveur de gestion de parc pour les points de terminaison macOS ; collecte l'inventaire, les événements, les audits et les résultats osquery. Mieux adapté aux environnements gérés.
[osquery](https://github.com/osquery/osquery) | Moteur de surveillance | Expose l'état du système d'exploitation sous forme de tables SQL, permettant des vérifications reproductibles des processus, utilisateurs, éléments de démarrage, connexions réseau, etc.
[pareto-mac](https://github.com/paretoSecurity/pareto-mac) | Scanner d'audit | Application de barre des menus qui vérifie un Mac par rapport aux recommandations de sécurité de base courantes.

<a name="additional-resources"></a>

# Ressources supplémentaires

- [Présentation de la sécurité et de la confidentialité d'iCloud](https://support.apple.com/102651)
- [Guide d'autodéfense contre la surveillance de l'EFF](https://ssd.eff.org/)
- [beerisgood/macOS_Hardening](https://github.com/beerisgood/macOS_Hardening)
- [Blog Reverse Engineering macOS](https://reverse.put.as/)
- [Ressources de rétro-ingénierie](http://samdmarshall.com/re.html)
- [Blog Malwarebytes](https://www.malwarebytes.com/blog)
- [Blog d'Objective-See](https://objective-see.com/blog.html)
- [Référentiels CIS](https://www.cisecurity.org/benchmark/apple_os/)
- [Apple Open Source](https://opensource.apple.com/)
- [iOS, l'avenir de macOS, liberté, sécurité et confidentialité dans un environnement mondial de plus en plus hostile](https://gist.github.com/iosecure/357e724811fe04167332ef54e736670d)
