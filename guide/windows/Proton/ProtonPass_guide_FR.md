<!-- Version du fichier
> Version : 1.0
> Dernière modification : 2026-10-03
-->

<div align="center">

# 🔐 Sécuriser Proton Pass au maximum

![Guide](https://img.shields.io/badge/type-guide-blue)
![Outil](https://img.shields.io/badge/source-Proton%20Pass-6D4AFF)
![Licence](https://img.shields.io/badge/licence-CC%20BY--NC--SA%204.0-lightgrey)

*Installation, options par défaut et activation progressive de toutes les fonctionnalités de sécurisation du compte et de l'application.*

</div>

---

<details>
<summary><strong>📑 Sommaire</strong></summary>

- [0. Introduction](#0-introduction)
- [Partie A — Réglages au niveau du compte Proton](#partie-a--réglages-au-niveau-du-compte-proton)
  - [1. Création du compte et installation](#1-création-du-compte-et-installation)
  - [2. Menu « Sécurité et vie privée »](#2-menu--sécurité-et-vie-privée-)
  - [3. Menu « Récupération »](#3-menu--récupération-)
    - [3.1 Options de réinitialisation du mot de passe](#31-options-de-réinitialisation-du-mot-de-passe)
    - [3.2 Options de récupération de données](#32-options-de-récupération-de-données)
    - [3.3 Options de récupération avancées](#33-options-de-récupération-avancées)
  - [4. Menu « Compte et mot de passe »](#4-menu--compte-et-mot-de-passe-)
- [Partie B — Réglages au niveau de Proton Pass lui-même](#partie-b--réglages-au-niveau-de-proton-pass-lui-même)
  - [5. Verrouillage local](#5-verrouillage-local)
  - [6. Exporter et sauvegarder le coffre](#6-exporter-et-sauvegarder-le-coffre)
- [7. Checklist récapitulative](#7-checklist-récapitulative)

</details>

---

## 0. Introduction

Ce guide couvre **uniquement Proton Pass** — son installation, ses réglages par défaut, et l'activation progressive de toutes les fonctionnalités disponibles pour une sécurisation maximale du compte et du coffre.

> [!NOTE]
> Tout ce qui concerne la création d'un mot de passe maître robuste, les passphrases Diceware, ou le chiffrement de fichiers pour vos sauvegardes (GPG, archives, etc.) est traité dans un guide séparé et complémentaire. Ce guide-ci part du principe que vous avez déjà un mot de passe maître solide en tête — pas comment le fabriquer.

Le guide est découpé en deux grandes parties, qui reflètent une distinction importante à garder en tête tout du long :

- **Partie A** — les réglages qui vivent au niveau du **compte Proton** (accessibles depuis `account.proton.me`, et qui s'appliquent à tous les services Proton que vous utilisez, pas seulement Pass).
- **Partie B** — les réglages qui vivent au niveau de **Proton Pass lui-même**, propres à chaque interface (app web, extension, desktop, mobile).

Confondre les deux est une source fréquente de réglages mal faits ou oubliés sur une interface sans qu'on s'en rende compte.

---

## Partie A — Réglages au niveau du compte Proton

### 1. Création du compte et installation

1. Créez votre compte sur [proton.me](https://proton.me).
2. Installez Proton Pass selon vos usages :
   - **Extension navigateur** (Chrome, Firefox, Edge, Brave, Safari)
   - **Application desktop** (Windows, macOS, Linux)
   - **Application mobile** (iOS, Android)
   - **Application web** (`pass.proton.me`), accessible sans rien installer
3. Avant de toucher à quoi que ce soit, faites un premier tour des paramètres par défaut.

> [!IMPORTANT]
> Rien n'est activé au-delà du strict minimum à la création du compte — ni le 2FA sur le compte, ni les méthodes de récupération avancées, ni le verrouillage local de Proton Pass. C'est un choix délibéré de Proton (ils préfèrent vous laisser choisir plutôt que présumer), mais ça veut dire que tout ce qui suit demande une action volontaire de votre part.

---

### 2. Menu « Sécurité et vie privée »

Accessible depuis `account.proton.me`, ce menu regroupe plusieurs fonctionnalités de surveillance et de contrôle :

- **Proton Sentinel** *(fonction premium)* — programme de protection avancée combinant apprentissage automatique et analystes de sécurité humains pour détecter et stopper les tentatives de piratage de compte. Non disponible sur l'offre gratuite.
- **Dark Web Monitoring** *(fonction premium)* — scanne le dark web à la recherche de vos identifiants Proton qui auraient fuité, et vous alerte le cas échéant. Non disponible sur l'offre gratuite.
- **Surveillance du compte** — journal d'activité de connexion, avec une option pour afficher les événements détaillés.
- **Vie privée et collecte de données** — contrôle la quantité de télémétrie/données d'usage collectées par Proton.

> [!TIP]
> Désactivez la collecte de données dans « Vie privée et collecte de données » si vous souhaitez minimiser ce que Proton collecte sur votre usage. C'est le point le plus souvent oublié de ce menu, et celui qui a le plus d'impact direct sur votre confidentialité.

---

### 3. Menu « Récupération »

Avant d'activer quoi que ce soit ici, il faut comprendre une distinction essentielle — c'est elle qui structure tout ce menu chez Proton.

> [!WARNING]
> **Réinitialiser le mot de passe** et **récupérer les données** sont deux choses différentes. Réinitialiser le mot de passe vous permet seulement de vous reconnecter. Si vous n'avez aucune méthode de récupération des données en plus, vous perdrez l'accès à tout ce qui était chiffré avant la réinitialisation — tout votre coffre Proton Pass inclus. C'est le piège le plus courant : penser qu'un email de secours suffit, puis découvrir après coup qu'il ne permet que de rouvrir un compte... vide.

#### 3.1 Options de réinitialisation du mot de passe

*Permettent de récupérer l'accès à votre compte Proton, mais ne permettent pas de récupérer vos données chiffrées.*

- **Vérification par email** — une adresse email de secours pour recevoir un lien de réinitialisation.
- **Vérification par SMS** — un numéro de téléphone pour recevoir un code de réinitialisation.

> [!NOTE]
> Si l'envoi du SMS échoue alors que le numéro et l'indicatif pays sont corrects, vérifiez qu'un VPN actif n'est pas connecté à un serveur d'un pays différent de celui du numéro — c'est une cause fréquente d'échec silencieux de la vérification.

#### 3.2 Options de récupération de données

*Permettent de déverrouiller vos données chiffrées si vous perdez votre mot de passe.*

- **Sauvegarde des données de l'appareil** — restauration possible depuis un appareil déjà connecté et de confiance.
- **Fichier de récupération** — fichier téléchargeable à conserver en lieu sûr, hors ligne.
- **Contacts de récupération de données** — permet de désigner une personne de confiance pouvant vous aider à récupérer l'accès. Optionnel, pertinent uniquement si vous avez quelqu'un à qui confier ce rôle.

#### 3.3 Options de récupération avancées

*Ces méthodes couvrent à la fois la réinitialisation du mot de passe et la récupération des données — ce sont les plus complètes.*

- **Phrase de récupération** — la pièce maîtresse de tout le système. Équivalent d'une seed phrase crypto : à imprimer ou écrire à la main, jamais à laisser en clair sur un disque connecté.
- **Réinitialisation lorsque connecté** — permet de changer le mot de passe depuis une session déjà active, sans passer par les méthodes ci-dessus.
- **Connexion par QR code** — permet de connecter un nouvel appareil en scannant un code depuis un appareil déjà connecté.
- **Accès d'urgence** *(fonction premium)* — accès au compte accordé à un tiers de confiance après un délai défini, en cas d'indisponibilité prolongée.

> [!TIP]
> La phrase de récupération doit être conservée **en clair, sur papier**, dans un endroit physiquement sûr — pas uniquement dans une archive chiffrée. C'est le seul secret qui, en cas de coup dur, doit rester accessible sans dépendre d'une chaîne de déchiffrement supplémentaire.

---

### 4. Menu « Compte et mot de passe »

- **Mode deux mots de passe** — sépare le mot de passe de connexion du mot de passe qui déchiffre les données. Fonctionnalité avancée qui *ajoute* une étape à la connexion : pertinente pour un usage très spécifique, pas pour réduire la friction au quotidien.
- **Vérification des mots de passe** — rappel périodique pour s'assurer que vous n'avez pas oublié votre mot de passe. N'apporte pas de protection en soi, c'est un filet de sécurité de confort.
- **Application d'authentification (2FA/TOTP)** — c'est ici qu'on sécurise le compte Proton lui-même avec un second facteur. Utilisez une app authenticator **externe** (Aegis, ou équivalent) plutôt que Proton Pass : stocker le 2FA du compte qui protège Proton Pass *dans* Proton Pass recrée exactement le problème de circularité qu'on évite par ailleurs pour le mot de passe maître.
- **Clé de sécurité (FIDO2/U2F)** — amélioration optionnelle, plus résistante au phishing que le TOTP. Pas indispensable pour démarrer si vous n'avez pas de clé physique sous la main ; l'authenticator app suffit largement comme première étape.

> [!IMPORTANT]
> C'est le point le plus souvent oublié de toute cette configuration : on sécurise soigneusement tous les comptes *via* Proton Pass, mais on oublie de sécuriser Proton lui-même avec un 2FA. Si ce n'est pas encore fait, c'est la priorité absolue de cette section.

N'oubliez pas de sauvegarder les **codes de secours** générés à l'activation du 2FA — même niveau de criticité que la phrase de récupération, même emplacement physique.

---

## Partie B — Réglages au niveau de Proton Pass lui-même

### 5. Verrouillage local

Proton Pass ne redemande pas systématiquement le mot de passe complet du compte pour un usage quotidien : chaque interface propose son propre verrouillage local, à configurer **séparément sur chacune**.

| Interface | Emplacement | Détails |
|---|---|---|
| **Extension navigateur** | Menu hamburger (☰) → Paramètres → onglet Sécurité | Cochez Code PIN, saisissez un code à 6 chiffres, choisissez une durée avant verrouillage auto |
| **Application desktop** | Icône d'engrenage → Sécurité → Code PIN | Même logique que l'extension, réglage indépendant |
| **Application web** | Proposé automatiquement lors d'une connexion complète avec le mot de passe maître | N'apparaît pas dans un menu persistant — c'est une invite ponctuelle au moment de la connexion |
| **Applications mobile/desktop** | Biométrie (empreinte, Face ID) | Gérée via les réglages de l'appareil (OS), pas une option Proton Pass dédiée |

Le PIN est protégé contre le brute-force : après trois tentatives échouées, une reconnexion complète est exigée.

> [!WARNING]
> **Ne stockez jamais votre mot de passe maître Proton à l'intérieur de Proton Pass.** Ça recrée la même circularité que pour le 2FA du compte : la clé qui ouvre le coffre ne doit pas être rangée dans le coffre. Les demandes ponctuelles du mot de passe maître pour des actions sensibles, même avec un PIN bien configuré, sont voulues — c'est une protection, pas un bug à contourner.

<details>
<summary>⚠️ Point de vigilance — VPN et reconnexions fréquentes</summary>

Si vous utilisez un VPN et que l'application (notamment desktop) vous demande de vous reconnecter entièrement et de reconfigurer vos réglages de façon inhabituellement fréquente, une cause plausible est un changement d'adresse IP sortante perçu comme un nouvel appareil/une nouvelle localisation, déclenchant une revalidation complète de session.

**Pour vérifier l'hypothèse** : restez connecté sur un seul pays/serveur VPN pendant quelques jours et observez si le problème disparaît. Ce n'est pas un comportement documenté officiellement à ce jour — à traiter comme hypothèse plausible plutôt que fait confirmé.

</details>

---

### 6. Exporter et sauvegarder le coffre

Proton Pass propose un export natif de votre coffre, chiffré au format PGP, directement depuis l'application (pas besoin d'un outil tiers pour cette étape précise).

1. Depuis l'app web, l'extension ou l'app desktop (l'export n'est pas disponible sur mobile) : **Paramètres → Export**.
2. Choisissez le format **chiffré (PGP)** plutôt qu'un export non protégé.
3. Le fichier obtenu est prêt à être archivé et protégé en aval.

> [!NOTE]
> La protection supplémentaire de ce fichier exporté (archivage, chiffrement en couche additionnelle, répartition sur plusieurs supports) est traitée dans le guide complémentaire sur le chiffrement de fichiers — ce guide-ci s'arrête à l'export natif fourni par Proton Pass.

Refaites cet export régulièrement, et à chaque changement significatif (nouveau mot de passe maître, ajout d'entrées importantes).

---

## 7. Checklist récapitulative

- [ ] Compte Proton créé, Proton Pass installé sur les interfaces utilisées
- [ ] Collecte de données minimisée (Sécurité et vie privée)
- [ ] Vérification par email activée
- [ ] Vérification par SMS activée (en l'absence de VPN actif au moment de la validation)
- [ ] Sauvegarde des données de l'appareil activée
- [ ] Fichier de récupération téléchargé et mis en lieu sûr
- [ ] Phrase de récupération générée, notée en clair sur papier
- [ ] Réinitialisation lorsque connecté activée
- [ ] Connexion par QR code activée
- [ ] Application d'authentification (2FA) activée sur le compte, via un outil externe à Proton Pass
- [ ] Codes de secours du 2FA sauvegardés sur papier
- [ ] Code PIN configuré sur chaque interface utilisée (extension, desktop, web)
- [ ] Biométrie activée sur mobile/desktop si disponible
- [ ] Mot de passe maître retiré du coffre Proton Pass
- [ ] Export PGP du coffre réalisé et archivé

---

<div align="center">

*Guide publié sous licence [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.fr).*

</div>
