<!-- Version du fichier 
> Version : 5.2
> Dernière modification : 2026-09-25
-->

<!-- TITRE -->
<a id="top"></a>
<h1 align="center">🔐 Synchroniser l'Authenticator Battle.net avec un gestionnaire de mots de passe tiers 🔐</h1>

<!-- date verif -->
<p align="center"><em><b>- Dernière vérification fonctionnelle : septembre 2026 -</b></em></p>

<!-- BADGES -->
<div align="center">

[![License: CC BY-NC-SA 4.0][badge_license]][url_license]
[![Guide : BNet_2FA_Sync][badge_guide]][github_repo]
[![Proton][badge_proton]][url_proton]

</div>

> **Objectif** : avoir un code TOTP à 8 chiffres qui fonctionne **à la fois** dans l'app officielle Battle.net **et** dans un gestionnaire de mots de passe tiers (Proton Pass, 1Password, Bitwarden, etc.), sur le **même authentificateur** — sans en créer deux distincts.

> [!IMPORTANT]
> Ce guide s'appuie sur une **API non officielle et non documentée** de Blizzard. Ce n'est pas une méthode supportée, elle peut cesser de fonctionner à tout moment sans préavis si Blizzard modifie son infrastructure. Ce contenu n'est affilié à Blizzard Entertainment d'aucune manière, et est fourni "tel quel", sans aucune garantie. À utiliser en connaissance de cause et à tes propres risques.

---

<!-- SOMMAIRE -->
<details open>
<summary><b>📑 Sommaire</b></summary>

<br>

- [🧭 Vue d'ensemble](#sec-1)
- [⚠️ Avant de commencer](#sec-2)
- [📋 Étape 1 — Détacher l'authentificateur actuel](#sec-3)
- [🔑 Étape 2 — Récupérer un SSO Token](#sec-4)
- [🎫 Étape 3 — Échanger le SSO Token contre un Bearer Token](#sec-5)
- [🛠️ Étape 4 — Attacher un nouvel authentificateur et récupérer le secret](#sec-6)
- [🔄 Étape 5 — Convertir le secret et l'importer dans le gestionnaire de mots de passe](#sec-7)
- [📱 Étape 6 — Relier l'app officielle Battle.net au même authentificateur](#sec-8)
- [✅ Étape 7 — Vérification finale](#sec-9)
- [🧹 Nettoyage post-manip](#sec-10)
- [🕳️ Pièges rencontrés (et ce qu'on en a appris)](#sec-11)
- [🔁 Transposable à d'autres services ?](#sec-12)
- [📖 Glossaire](#sec-13)
- [🙏 Crédits & sources](#sec-14)

</details>

<!-- TLDR -->
<details>
<summary><b>📝 TL;DR — résumé express (clique pour dérouler)</b></summary>

<br>

1. **Détacher** l'authentificateur actuel sur `account.battle.net` → Sécurité
2. Récupérer un **SSO Token** en se connectant via `account.battle.net/login/en/?ref=localhost` (repérer `ST=` dans l'URL)
3. Échanger ce token contre un **Bearer Token** via `oauth.battle.net/oauth/sso`
4. Attacher un nouvel authentificateur via `authenticator-rest-api.bnet-identity.blizzard.net/v1/authenticator` → récupérer `serial`, `restoreCode`, `deviceSecret`
5. Convertir `deviceSecret` en base32 → construire l'URL `otpauth://totp/Battle.net?secret=...&digits=8` → importer dans le gestionnaire de mots de passe
6. Rouvrir l'app Battle.net → renseigner `serial` + `restoreCode` quand elle le demande
7. Tester une connexion pour confirmer

</details>

---

<a id="sec-1"></a>
## 🧭 Vue d'ensemble

> [!NOTE]
> Battle.net utilise un système TOTP classique (comme Google Authenticator) mais ne l'expose pas directement dans son app officielle — impossible d'y scanner un QR code ou de récupérer le secret depuis l'interface. Pour obtenir ce secret et le dupliquer dans un gestionnaire tiers, il faut passer par l'API officielle de Blizzard, en ligne de commande.

<details open>
<summary><b>Le principe en 4 mouvements</b></summary>

<br>

1. **Détacher** l'authentificateur actuel *(site web)*
2. **En créer un nouveau via l'API** *(curl)* → on récupère le secret en clair
3. **Importer** ce secret dans le gestionnaire de mots de passe
4. **Relier** l'app officielle à ce même authentificateur *(serial + code de restauration)*

</details>

Résultat : l'app officielle (notifications push) et le gestionnaire de mots de passe (code manuel de secours) pointent vers **le même** authentificateur.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-2"></a>
## ⚠️ Avant de commencer

> [!CAUTION]
> **Lis ce tableau avant toute chose — il conditionne la réussite de toute la manip.**

| 🔑 Prérequis | 📝 Pourquoi |
|---|---|
| 📱 **Numéro de téléphone à jour** dans *Détails du compte* (pas dans *Sécurité*) | Ton filet de sécurité si la manip tourne mal — active le SMS Protect |
| 🕵️ **Navigateur en mode privé/navigation privée** | Évite les conflits de session lors de la récupération du token (voir encadré plus bas) |
| 💻 **Terminal local (PowerShell, CMD, etc.)** | Toute la manip se fait en local, rien ne doit transiter par un tiers |
| 📝 **Éditeur de texte ou bloc-notes ouvert** (ex. Notepad++) | Pour y coller au fil de l'eau les valeurs sensibles récupérées (token, secret…) |

> [!CAUTION]
> Entre l'étape de détachement et la fin de la reconfiguration de l'app, ton compte est temporairement **sans authentificateur actif**. Prévois de faire toute la manip d'une traite, sans interruption prolongée.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-3"></a>
## 📋 Étape 1 — Détacher l'authentificateur actuel

Sur [account.battle.net][url_battlenet] :

**Sécurité → Blizzard Authenticator → Mettre à jour → Supprimer l'authenticator**

Confirme via le code envoyé par SMS ou par le code de sécurité de l'authentificateur actuel.

> [!TIP]
> Ne confonds pas avec le bouton *"Désactiver sur cet appareil"* qu'on trouve **dans l'app**. Ce dernier ne détache rien côté compte — il rend juste l'app locale inactive. Seul le bouton **Détacher/Supprimer** du site web supprime réellement l'authentificateur du compte.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-4"></a>
## 🔑 Étape 2 — Récupérer un SSO Token

> [!TIP]
> Garde dès maintenant **un terminal PowerShell déjà ouvert et prêt à l'emploi** à côté de ton navigateur — l'étape 3 doit s'enchaîner vite après celle-ci (voir encadré plus bas), autant ne pas perdre de temps à ouvrir un terminal après coup.

1. Va sur : `https://account.battle.net/login/en/?ref=localhost`
2. Connecte-toi avec ton compte Battle.net
3. Tu atterris sur une page **404 — c'est normal**, ignore-la
4. Dans la barre d'adresse, repère le paramètre `ST=` et copie tout ce qui suit

Format attendu : `EU-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx-xxxxxxxxx` (ou `US-...` selon ta région)

> [!WARNING]
> **Le token expire très vite (usage unique / quasi-immédiat).**
>
> Enchaîne connexion → copie → commande suivante **le plus rapidement possible**, sans pause. Un délai, même de quelques dizaines de secondes, peut suffire à l'invalider (`{"error":"invalid_token","error_description":"Invalid SSO token."}`).
>
> **C'est pour ça qu'on recommande la navigation privée** : si tu étais déjà connecté dans une fenêtre normale au moment de générer ce lien, le token peut être recyclé/invalide. Une fenêtre privée garantit une session fraîche à chaque tentative.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-5"></a>
## 🎫 Étape 3 — Échanger le SSO Token contre un Bearer Token

👉 Dans ton terminal :

```powershell
curl.exe -X POST "https://oauth.battle.net/oauth/sso" -H "content-type: application/x-www-form-urlencoded; charset=utf-8" -d "client_id=baedda12fe054e4abdfc3ad7bdea970a&grant_type=client_sso&scope=auth.authenticator&token=NOM_SSO_A_REMPLACER"
```

Remplace `NOM_SSO_A_REMPLACER` par ton token de l'étape 2.

> [!WARNING]
> **Utilise bien `curl.exe`, pas juste `curl`** sous PowerShell — `curl` est souvent un alias vers `Invoke-WebRequest`, qui ne gère pas les options de la même façon et peut faire échouer la commande silencieusement ou avec une erreur trompeuse.

Réponse attendue :
```json
{"access_token":"NOM_BEARER_A_REMPLACER","token_type":"bearer","expires_in":7775999,"scope":"auth.authenticator","sub":"..."}
```

Le champ `access_token` est ton **Bearer Token**.

> [!NOTE]
> **Tu peux souffler.** Contrairement au SSO Token, le Bearer Token n'expire pas en quelques secondes — il reste valide plusieurs semaines. Une fois que tu l'as en main, plus aucune urgence : prends ton temps pour la suite.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-6"></a>
## 🛠️ Étape 4 — Attacher un nouvel authentificateur et récupérer le secret

👉 Toujours dans ton terminal :

```powershell
curl.exe -X POST "https://authenticator-rest-api.bnet-identity.blizzard.net/v1/authenticator" -H "accept: application/json" -H "Authorization: Bearer NOM_BEARER_A_REMPLACER"
```

Remplace `NOM_BEARER_A_REMPLACER` par le token obtenu dans le champ `access_token` à l'étape 3.

Réponse attendue :
```json
{"serial":"NOM_SERIAL","restoreCode":"NOM_RESTORE_CODE","deviceSecret":"NOM_DEVICE_SECRET","timeMs":0,"requireHealup":false}
```

> [!IMPORTANT]
> **C'est LE moment clé de toute la manip.** C'est la seule et unique fois où le `deviceSecret` t'est communiqué en clair. Sauvegarde immédiatement les trois valeurs (`serial`, `restoreCode`, `deviceSecret`) dans un fichier texte local le temps de finir la procédure.
>
> Par exemple, sous **Notepad++** : `Fichier → Nouveau` (ou `Ctrl + N`), colle les trois valeurs, et laisse ce fichier ouvert jusqu'à la fin de l'étape 6 — tu le supprimeras au [nettoyage final](#sec-10).

Cette requête **attache déjà** le nouvel authentificateur au compte, directement via l'API officielle — pas besoin d'action supplémentaire côté site web, c'est fait.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-7"></a>
## 🔄 Étape 5 — Convertir le secret et l'importer dans le gestionnaire de mots de passe

1. Convertis `deviceSecret` (hexadécimal) en **base32** — par exemple via [cryptii.com/pipes/hex-to-base32][url_cryptii] *(outil en ligne gratuit, sans installation, qui gère cette conversion en un copier-coller)*
2. Construis l'URL TOTP standard :

```
otpauth://totp/Battle.net?secret=NOM_SECRET_BASE32&digits=8
```

3. Importe cette URL dans le champ **TOTP / 2FA** de ton gestionnaire de mots de passe (Proton Pass, 1Password, Bitwarden…)
4. Vérifie qu'un code à **8 chiffres** apparaît et tourne toutes les **30 secondes**

> [!TIP]
> `digits=8` est indispensable — Battle.net utilise 8 chiffres, contrairement au standard à 6 chiffres de la plupart des services (Google, Discord…). Certaines apps TOTP anciennes ou limitées (Google Authenticator classique, par exemple) ne gèrent pas ce format — vérifie que ton outil le supporte avant de basculer dessus en usage principal.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-8"></a>
## 📱 Étape 6 — Relier l'app officielle Battle.net au même authentificateur

1. Ouvre l'app Battle.net sur ton téléphone
2. Comme le compte a maintenant un authentificateur (attaché à l'étape 4) mais que l'app locale n'en a jamais eu connaissance, elle va te présenter un écran demandant :
   - Adresse e-mail ou numéro de téléphone
   - **Numéro de série**
   - **Code de restauration**
3. Renseigne le **`serial`** et le **`restoreCode`** obtenus à l'étape 4 (les nouveaux, pas d'anciennes valeurs)
4. Valide

Si l'app accepte, elle est maintenant liée à la **même identité d'authentificateur** que celle importée dans ton gestionnaire de mots de passe.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-9"></a>
## ✅ Étape 7 — Vérification finale

Fais un vrai test de connexion (site web ou client de jeu) avec le code généré par ton gestionnaire de mots de passe. S'il est accepté → la synchro est confirmée.

Tu peux aussi comparer visuellement les deux codes manuels (app vs gestionnaire) au même instant : ils doivent être strictement identiques.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-10"></a>
## 🧹 Nettoyage post-manip

| ✅ À faire | 📝 Pourquoi |
|---|---|
| Supprimer le fichier texte local avec `serial` / `restoreCode` / `deviceSecret` en clair | Ces infos sont maintenant dans l'app et le gestionnaire — plus besoin de la copie brute |
| Vider l'historique du terminal (`Clear-History` sous PowerShell + supprimer le fichier d'historique PSReadLine) | Les tokens/commandes de la session y restent visibles sinon |
| *(Optionnel, "ceinture et bretelles")* Conserver le `restoreCode` dans une note sécurisée à part, en plus du TOTP | Utile si tu dois relier un jour un 3ᵉ appareil, sans repasser par toute la manip curl |

Commandes à rentrer dans Powershell pour tout nettoyer :

```powershell
Clear-History
Remove-Item (Get-PSReadLineOption).HistorySavePath -ErrorAction SilentlyContinue
```

> [!NOTE]
> Si tu stockes `restoreCode` et/ou `deviceSecret` dans une note de ton gestionnaire de mots de passe, garde en tête que ça devient visible par quiconque tu partagerais cet item (fonctionnalités de partage de vault). 
>
> Pas un problème pour un usage strictement personnel, mais à anticiper si tu envisages de partager l'item plus tard.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-11"></a>
## 🕳️ Pièges rencontrés (et ce qu'on en a appris)

<table>
<tr><td width="30%">🩺 <b>Symptôme</b></td><td>🔍 <b>Cause réelle</b></td></tr>
<tr><td><code>bna : terme non reconnu</code></td><td>Le dossier <code>Scripts</code> de Python n'est pas dans le <code>PATH</code></td></tr>
<tr><td><code>ModuleNotFoundError: pkg_resources</code></td><td><code>pkg_resources</code> a été retiré de <code>setuptools</code> à partir de la version 82 — un vieux paquet Python peut en dépendre encore</td></tr>
<tr><td><code>502 Bad Gateway</code> sur <code>mobile-service.blizzard.com</code></td><td>Endpoint <b>legacy définitivement mort</b> côté Blizzard depuis la fusion de l'app — ne pas s'acharner dessus, utiliser l'API actuelle (<code>authenticator-rest-api.bnet-identity.blizzard.net</code>)</td></tr>
<tr><td><code>{"error":"invalid_token"}</code></td><td>Token expiré (trop de délai entre connexion et commande, ou session déjà ouverte ailleurs au moment de générer le lien)</td></tr>
</table>

> [!NOTE]
> **Leçon générale** : sur un sujet aussi mouvant (API non documentée, changée au fil des mises à jour de l'app), privilégier systématiquement les **retours d'utilisateurs les plus récents** trouvés en cherchant plutôt qu'un tutoriel ancien, même détaillé — une méthode qui marchait il y a un an peut très bien être morte aujourd'hui, et inversement une fonctionnalité annoncée disparue peut avoir été réintroduite.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-12"></a>
## 🔁 Transposable à d'autres services ?

Le principe général (détacher → recréer via API/script → récupérer le secret brut → réimporter des deux côtés) s'applique en théorie à tout service qui :
- utilise un TOTP standard sous le capot,
- mais n'expose pas de QR code / secret manuel dans son app officielle,
- et dispose d'une API (même non documentée) permettant d'attacher un nouvel authentificateur.

Chaque service ayant sa propre API et son propre format de code (nombre de chiffres, algorithme de hachage), les commandes précises seront différentes — mais la démarche de recherche (chercher les retours **récents** de la communauté, un outil ou un endpoint à jour) reste la même.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-13"></a>
## 📖 Glossaire

| 🔑 Terme | 📖 Définition |
|---|---|
| **SSO Token** | Jeton de session temporaire (quasi-immédiat) obtenu en se connectant au site Battle.net — sert uniquement à obtenir le Bearer Token |
| **Bearer Token** | Jeton d'accès à l'API Blizzard, valide plusieurs semaines, utilisé pour autoriser l'action d'attachement d'authentificateur |
| **Serial** (numéro de série) | Identifiant unique de l'authentificateur, propre à chaque appareil/enregistrement |
| **Restore Code** (code de restauration) | Code secret permettant de relier un appareil à un authentificateur déjà attaché au compte, sans en créer un nouveau |
| **Device Secret** | Le secret TOTP brut sous-jacent — c'est cette valeur (convertie en base32) qui permet de générer les codes à 8 chiffres dans n'importe quel gestionnaire compatible |

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-14"></a>
## 🙏 Crédits & sources

Cette méthode s'appuie sur le travail de la communauté, en particulier :

- [`python-bna` — issue #42][github_bna_issue], qui documente la méthode curl utilisée dans ce guide
- Les contributeurs à l'origine de cette méthode : `@BillyCurtis`, `@Gigafrost`, et `@digikwal` pour la synthèse et les tests
- [`digikwal/bliz_totp`][github_bliz_totp], un script Python qui automatise ces mêmes étapes

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

Ce guide est distribué sous licence [**CC BY-NC-SA 4.0**][url_license].

<!-- ==================== VARIABLES DE RÉFÉRENCE ==================== -->

<!-- Badges (images shields.io) -->
[badge_license]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg
[badge_guide]: https://img.shields.io/badge/Guide%20%3A-BNet_2FA_Sync-blue?style=flat&logo=mdbook&logoColor=white&logoSize=auto&label=Guide%20%3A&labelColor=black&color=darkcyan
[badge_proton]: https://img.shields.io/badge/Proton-6D4AFF?style=flat&logo=proton&logoColor=white&logoSize=auto

<!-- External URLs (services tiers, hors GitHub) -->
[url_license]: https://creativecommons.org/licenses/by-nc-sa/4.0/
[url_proton]: https://proton.me
[url_cryptii]: https://cryptii.com/pipes/hex-to-base32
[url_battlenet]: https://account.battle.net

<!-- GitHub links & local repo files -->
[github_user]: https://github.com/Data-Spirit
[github_repo]: https://github.com/Data-Spirit/All_Guides
[github_bna_issue]: https://github.com/jleclanche/python-bna/issues/42
[github_bliz_totp]: https://github.com/digikwal/bliz_totp
