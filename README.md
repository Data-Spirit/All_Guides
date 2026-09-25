<!-- Version du fichier 
> Version : 1.0
> Dernière modification : 2026-09-25
-->

<!-- BANNIERE CENTREE -->
<div align="center">

![All_Guides Banner][github_banner]

</div>

<!-- TITRE CENTRE + Sous-Titre CENTRE -->
<h1 align="center">All_Guides</h1>

<p align="center">
  <em><b>- Veritas Lux Mea -</b></em>
</p>

<!-- BADGES CENTRES + LIENS HYPERTEXT INCLUS -->
<div align="center">

[![License: CC BY-NC-SA 4.0][badge_license]][url_license]
[![Docs / Tutos][badge_doccount]][github_repo]
[![Repo][badge_repo]][github_repo]

</div>

> Une bibliothèque de savoir personnelle, en perpétuelle expansion : guides, tutoriels et prompts IA rassemblés en un seul endroit, pour Windows, Linux, le web et au-delà.

---

## ✨ Présentation

> **All_Guides** est un repo pensé comme une véritable encyclopédie du savoir personnel : un seul et même endroit où compiler, réunir et collectionner un maximum de **guides et tutoriels** — pour des applications Windows/Linux, des services web, des sites internet — ainsi qu'un maximum de **prompts pour IA** et de savoir autour des prompts et des agents IA.
>
> Ce repo n'a pas vocation à rester figé : il grandit au fil des besoins et des découvertes. À terme, il pourra aussi accueillir des guides dédiés aux jeux vidéo.

---

## 📚 Glossaire des guides

<div align="center">

| Catégorie | Guide | Description |
|:---:|---|---|
| 🪟 Windows | [Wand-Enhancer (FR)][guide_wand_fr] | Guide décrivant comment compiler `Wand-Enhancer.exe` soi-même. |
| 🪟 Windows | Guide Battle.net 2FA sync [(FR)][guide_Bnet_2fa_fr] & [(EN)][guide_Bnet_2fa_en]| Guide décrivant comment synchroniser l'authenticator  `Battle.net` sur un logiciel tierce. (Proton dans ce cas) |
| 🐧 Linux | [VirtualBox + SecureBoot][guide_virtualbox] | Tutoriel décrivant comment installer VirtualBox avec le SecureBoot activé. |
| 🤖 IA & Prompts | [Prompts][guide_prompts] | Collection de prompts pour IA — *(à compléter)*. |

</div>

---

## 🔗 Liens utiles

<details open><summary><b>Liste :</b></summary>

- [Documentation Officielle GitHub][url_github_docs] _(fr)_
- [Table des Emotes GitHub][url_emoji_cheatsheet] _(en)_
- [Site qui vous aide à choisir une Licence adaptée à votre projet OpenSource][url_chooselicense] _(en)_
- [Site pour créer les vignettes d'informations et de suivi : "shields.io"][url_shieldsio] _(en)_
- [Bibliothèque de Polices d'écriture][url_nerdfonts] _(en)_
- _...à compléter..._

</details>

---

## 📁 Structure du repo

```
All_Guides/
├── asset/img/          → Bannière et visuels du repo
├── guide/              → Guides et tutoriels classés par OS / sujet
│   ├── windows/
│   └── linux/
├── prompts/            → Prompts IA et documentation associée
├── LICENSE.md
└── README.md           → this file
```

---

## 📜 Licence

> Le contenu original de ce repo, créé par **Spirit**, est distribué sous licence [CC BY-NC-SA 4.0][url_license]. Voir [LICENSE.md][github_license] pour les termes complets.
>
> Les logiciels, services, sites et outils tiers mentionnés dans les guides restent la propriété de leurs éditeurs et ayants droit respectifs. Ce repo ne fait que documenter et référencer leur usage.

<!-- ============================== -->
<!--    Link & Badge Definitions    -->
<!-- ============================== -->

<!-- Badges (shields.io images) -->
[badge_license]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg
[badge_doccount]: https://img.shields.io/github/directory-file-count/Data-Spirit/All_Guides?color=informational&label=Docs%20%2F%20Tutos&type=dir
[badge_repo]: https://img.shields.io/badge/Repo%20%3A-All_Guides-blue?style=flat&logo=github&logoColor=white&logoSize=auto&label=Repo%20%3A&labelColor=grey&color=mediumseagreen

<!-- External URLs (services tiers, hors GitHub) -->
[url_license]: https://creativecommons.org/licenses/by-nc-sa/4.0/
[url_github_docs]: https://docs.github.com/fr/get-started
[url_emoji_cheatsheet]: https://github.com/ikatyang/emoji-cheat-sheet/blob/master/README.md
[url_chooselicense]: https://choosealicense.com/
[url_shieldsio]: https://shields.io/
[url_nerdfonts]: https://www.nerdfonts.com/#home

<!-- GitHub links & local repo files -->
[github_repo]: https://github.com/Data-Spirit/All_Guides
[github_license]: ./LICENSE.md
[github_user]: https://github.com/Data-Spirit
[github_banner]: ./asset/img/banner_02.webp
<!-- url_lien_absolu: https://raw.githubusercontent.com/Data-Spirit/All_Guides/main/asset/img/banner_02.webp -->


<!-- Language online guides (GitHub Pages) -->
[guide_wand_fr]: https://data-spirit.github.io/All_Guides/guide/windows/wand/Guide%20-%20Creer%20WandEnhancer_exe_FR.html
[guide_Bnet_2fa_fr]: https://github.com/Data-Spirit/All_Guides/blob/main/guide/windows/Battle.Net/BattleNet_2FA_Sync_FR.md
[guide_Bnet_2fa_en]: https://github.com/Data-Spirit/All_Guides/blob/main/guide/windows/Battle.Net/BattleNet_2FA_Sync_EN.md
[guide_virtualbox]: ./guide/linux/VirtualBox%2BSecureBoot/guide-VirtualBox%2BSecureBoot.md
[guide_prompts]: ./prompts