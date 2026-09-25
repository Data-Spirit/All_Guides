<!-- File Version -->

> **Version : 5.2** \
> *Last modification : 2026-09-25*

<!-- TITLE -->
<a id="top"></a>
<h1 align="center">🔐 Syncing the Battle.net Authenticator with a Third-Party Password Manager 🔐</h1>

<!-- last verified date -->
<p align="center"><em><b>- Last verified working: September 2026 -</b></em></p>

<!-- BADGES -->
<div align="center">

[![License: CC BY-NC-SA 4.0][badge_license]][url_license]
[![Guide : BNet_2FA_Sync][badge_guide]][github_repo]
[![Proton][badge_proton]][url_proton]

</div>

> **Goal**: have an 8-digit TOTP code that works **both** in the official Battle.net app **and** in a third-party password manager (Proton Pass, 1Password, Bitwarden, etc.), on the **same authenticator** — without creating two separate ones.

> [!IMPORTANT]
> This guide relies on an **unofficial, undocumented** Blizzard API. It's not a supported method, and it can stop working at any time without notice if Blizzard changes its infrastructure. This content is not affiliated with Blizzard Entertainment in any way, and is provided "as is," with no guarantee. Use it knowingly and at your own risk.

---

<!-- TABLE OF CONTENTS -->
<details open>
<summary><b>📑 Table of Contents</b></summary>

<br>

- [🧭 Overview](#sec-1)
- [⚠️ Before You Start](#sec-2)
- [📋 Step 1 — Detach the Current Authenticator](#sec-3)
- [🔑 Step 2 — Get an SSO Token](#sec-4)
- [🎫 Step 3 — Exchange the SSO Token for a Bearer Token](#sec-5)
- [🛠️ Step 4 — Attach a New Authenticator and Retrieve the Secret](#sec-6)
- [🔄 Step 5 — Convert the Secret and Import It Into the Password Manager](#sec-7)
- [📱 Step 6 — Link the Official Battle.net App to the Same Authenticator](#sec-8)
- [✅ Step 7 — Final Verification](#sec-9)
- [🧹 Post-Procedure Cleanup](#sec-10)
- [🕳️ Pitfalls We Hit (and What We Learned)](#sec-11)
- [🔁 Does This Work for Other Services?](#sec-12)
- [📖 Glossary](#sec-13)
- [🙏 Credits & Sources](#sec-14)

</details>

<!-- TLDR -->
<details>
<summary><b>📝 TL;DR — quick summary (click to expand)</b></summary>

<br>

1. **Detach** the current authenticator on `account.battle.net` → Security
2. Get an **SSO Token** by logging in via `account.battle.net/login/en/?ref=localhost` (find `ST=` in the URL)
3. Exchange that token for a **Bearer Token** via `oauth.battle.net/oauth/sso`
4. Attach a new authenticator via `authenticator-rest-api.bnet-identity.blizzard.net/v1/authenticator` → get `serial`, `restoreCode`, `deviceSecret`
5. Convert `deviceSecret` to base32 → build the URL `otpauth://totp/Battle.net?secret=...&digits=8` → import it into the password manager
6. Reopen the Battle.net app → enter `serial` + `restoreCode` when asked
7. Test a login to confirm

</details>

---

<a id="sec-1"></a>
## 🧭 Overview

> [!NOTE]
> Battle.net uses a standard TOTP system (like Google Authenticator) but doesn't expose it directly in its official app — there's no way to scan a QR code or retrieve the secret from the interface. To get that secret and duplicate it in a third-party manager, you have to go through Blizzard's official API, from the command line.

<details open>
<summary><b>The Process in 4 Moves</b></summary>

<br>

1. **Detach** the current authenticator *(website)*
2. **Create a new one via the API** *(curl)* → get the secret in plain text
3. **Import** this secret into the password manager
4. **Link** the official app to this same authenticator *(serial + restore code)*

</details>

Result: the official app (push notifications) and the password manager (manual backup code) both point to **the same** authenticator.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-2"></a>
## ⚠️ Before You Start

> [!CAUTION]
> **Read this table before anything else — it determines whether the whole procedure succeeds.**

| 🔑 Requirement | 📝 Why |
|---|---|
| 📱 **Up-to-date phone number** in *Account Details* (not in *Security*) | Your safety net if something goes wrong — enables SMS Protect |
| 🕵️ **Browser in private/incognito mode** | Avoids session conflicts when retrieving the token (see box below) |
| 💻 **Local terminal (PowerShell, CMD, etc.)** | The whole procedure happens locally — nothing should go through a third party |
| 📝 **Text editor or notepad open** (e.g. Notepad++) | To paste in the sensitive values as you retrieve them (token, secret…) |

> [!CAUTION]
> Between the detach step and the end of reconfiguring the app, your account is temporarily **without an active authenticator**. Plan to do the whole procedure in one go, without a long interruption.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-3"></a>
## 📋 Step 1 — Detach the Current Authenticator

On [account.battle.net][url_battlenet]:

**Security → Blizzard Authenticator → Update → Remove Authenticator**

Confirm using the code sent by SMS, or the security code from the current authenticator.

> [!TIP]
> Don't confuse this with the *"Deactivate on this device"* button found **in the app**. That one doesn't detach anything on the account side — it just makes the local app inactive. Only the **Detach/Remove** button on the website actually removes the authenticator from the account.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-4"></a>
## 🔑 Step 2 — Get an SSO Token

> [!TIP]
> Keep **a PowerShell terminal already open and ready to go** next to your browser from now on — step 3 needs to follow quickly after this one (see box below), so there's no point wasting time opening a terminal afterward.

1. Go to: `https://account.battle.net/login/en/?ref=localhost`
2. Log in with your Battle.net account
3. You'll land on a **404 page — that's normal**, ignore it
4. In the address bar, find the `ST=` parameter and copy everything after it

Expected format: `EU-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx-xxxxxxxxx` (or `US-...` depending on your region)

> [!WARNING]
> **The token expires very quickly (single-use / near-instant).**
>
> Chain login → copy → next command **as fast as possible**, without pausing. Even a delay of a few dozen seconds can be enough to invalidate it (`{"error":"invalid_token","error_description":"Invalid SSO token."}`).
>
> **That's why private browsing is recommended**: if you were already logged in in a normal window when generating this link, the token can end up recycled/invalid. A private window guarantees a fresh session every time.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-5"></a>
## 🎫 Step 3 — Exchange the SSO Token for a Bearer Token

👉 In your terminal:

```powershell
curl.exe -X POST "https://oauth.battle.net/oauth/sso" -H "content-type: application/x-www-form-urlencoded; charset=utf-8" -d "client_id=baedda12fe054e4abdfc3ad7bdea970a&grant_type=client_sso&scope=auth.authenticator&token=SSO_TOKEN_TO_REPLACE"
```

Replace `SSO_TOKEN_TO_REPLACE` with your token from step 2.

> [!WARNING]
> **Make sure to use `curl.exe`, not just `curl`**, in PowerShell — `curl` is often aliased to `Invoke-WebRequest`, which doesn't handle the options the same way and can make the command fail silently or with a misleading error.

Expected response:
```json
{"access_token":"BEARER_TOKEN_TO_REPLACE","token_type":"bearer","expires_in":7775999,"scope":"auth.authenticator","sub":"..."}
```

The `access_token` field is your **Bearer Token**.

> [!NOTE]
> **You can relax now.** Unlike the SSO Token, the Bearer Token doesn't expire in a few seconds — it stays valid for several weeks. Once you have it, there's no more urgency: take your time for the rest.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-6"></a>
## 🛠️ Step 4 — Attach a New Authenticator and Retrieve the Secret

👉 Still in your terminal:

```powershell
curl.exe -X POST "https://authenticator-rest-api.bnet-identity.blizzard.net/v1/authenticator" -H "accept: application/json" -H "Authorization: Bearer BEARER_TOKEN_TO_REPLACE"
```

Replace `BEARER_TOKEN_TO_REPLACE` with the token you got from the `access_token` field in step 3.

Expected response:
```json
{"serial":"SERIAL_VALUE","restoreCode":"RESTORE_CODE_VALUE","deviceSecret":"DEVICE_SECRET_VALUE","timeMs":0,"requireHealup":false}
```

> [!IMPORTANT]
> **This is THE key moment of the whole procedure.** It's the one and only time `deviceSecret` is given to you in plain text. Immediately save the three values (`serial`, `restoreCode`, `deviceSecret`) to a local text file until you finish the procedure.
>
> For example, in **Notepad++**: `File → New` (or `Ctrl + N`), paste the three values, and leave the file open until the end of step 6 — you'll delete it during [final cleanup](#sec-10).

This request **already attaches** the new authenticator to the account, directly via the official API — no extra action needed on the website side, it's done.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-7"></a>
## 🔄 Step 5 — Convert the Secret and Import It Into the Password Manager

1. Convert `deviceSecret` (hexadecimal) to **base32** — for example via [cryptii.com/pipes/hex-to-base32][url_cryptii] *(a free online tool, no install needed, that handles this conversion with a simple copy-paste)*
2. Build the standard TOTP URL:

```
otpauth://totp/Battle.net?secret=BASE32_SECRET_VALUE&digits=8
```

3. Import this URL into the **TOTP / 2FA** field of your password manager (Proton Pass, 1Password, Bitwarden…)
4. Check that an **8-digit** code appears and rotates every **30 seconds**

> [!TIP]
> `digits=8` is essential — Battle.net uses 8 digits, unlike the 6-digit standard used by most services (Google, Discord…). Some older or limited TOTP apps (classic Google Authenticator, for example) don't support this format — check that your tool supports it before switching to it as your main method.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-8"></a>
## 📱 Step 6 — Link the Official Battle.net App to the Same Authenticator

1. Open the Battle.net app on your phone
2. Since the account now has an authenticator (attached in step 4) but the local app has never known about it, it will show you a screen asking for:
   - Email address or phone number
   - **Serial number**
   - **Restore code**
3. Enter the **`serial`** and **`restoreCode`** you got in step 4 (the new ones, not old values)
4. Confirm

If the app accepts it, it's now linked to the **same authenticator identity** as the one imported into your password manager.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-9"></a>
## ✅ Step 7 — Final Verification

Do a real login test (website or game client) with the code generated by your password manager. If it's accepted → the sync is confirmed.

You can also visually compare the two manual codes (app vs. manager) at the same moment: they should be exactly identical.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-10"></a>
## 🧹 Post-Procedure Cleanup

| ✅ To Do | 📝 Why |
|---|---|
| Delete the local text file with `serial` / `restoreCode` / `deviceSecret` in plain text | This info is now in the app and the manager — no need for the raw copy anymore |
| Clear the terminal history (`Clear-History` in PowerShell + delete the PSReadLine history file) | Otherwise the tokens/commands from the session stay visible there |
| *(Optional, "belt and suspenders")* Keep the `restoreCode` in a separate secure note, in addition to the TOTP | Useful if you ever need to link a 3rd device without redoing the whole curl procedure |

Commands to run in PowerShell to clean everything up:

```powershell
Clear-History
Remove-Item (Get-PSReadLineOption).HistorySavePath -ErrorAction SilentlyContinue
```

> [!NOTE]
> If you store `restoreCode` and/or `deviceSecret` in a note in your password manager, keep in mind that it becomes visible to anyone you share that item with (vault-sharing features).
>
> Not an issue for strictly personal use, but worth planning for if you might share the item later.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-11"></a>
## 🕳️ Pitfalls We Hit (and What We Learned)

<table>
<tr><td width="30%">🩺 <b>Symptom</b></td><td>🔍 <b>Real Cause</b></td></tr>
<tr><td><code>bna : term not recognized</code></td><td>Python's <code>Scripts</code> folder isn't in the <code>PATH</code></td></tr>
<tr><td><code>ModuleNotFoundError: pkg_resources</code></td><td><code>pkg_resources</code> was removed from <code>setuptools</code> starting with version 82 — an old Python package can still depend on it</td></tr>
<tr><td><code>502 Bad Gateway</code> on <code>mobile-service.blizzard.com</code></td><td><b>Legacy endpoint permanently dead</b> on Blizzard's side since the app merger — don't keep trying it, use the current API (<code>authenticator-rest-api.bnet-identity.blizzard.net</code>)</td></tr>
<tr><td><code>{"error":"invalid_token"}</code></td><td>Expired token (too much delay between login and command, or a session already open elsewhere when generating the link)</td></tr>
</table>

> [!NOTE]
> **General lesson**: on a topic this fluid (an undocumented API, changing with every app update), always prioritize the **most recent user reports** you can find over an old tutorial, however detailed — a method that worked a year ago may well be dead today, and conversely a feature announced as removed may have been reintroduced.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-12"></a>
## 🔁 Does This Work for Other Services?

The general principle (detach → recreate via API/script → retrieve the raw secret → reimport on both sides) applies in theory to any service that:
- uses standard TOTP under the hood,
- but doesn't expose a QR code / manual secret in its official app,
- and has an API (even an undocumented one) that lets you attach a new authenticator.

Since every service has its own API and its own code format (number of digits, hashing algorithm), the exact commands will differ — but the research approach (looking for **recent** community reports, a tool or an up-to-date endpoint) stays the same.

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-13"></a>
## 📖 Glossary

| 🔑 Term | 📖 Definition |
|---|---|
| **SSO Token** | Temporary (near-instant) session token obtained by logging into the Battle.net website — used only to get the Bearer Token |
| **Bearer Token** | Access token for the Blizzard API, valid for several weeks, used to authorize the authenticator-attach action |
| **Serial** (serial number) | Unique identifier for the authenticator, specific to each device/registration |
| **Restore Code** | Secret code that lets you link a device to an authenticator already attached to the account, without creating a new one |
| **Device Secret** | The underlying raw TOTP secret — this value (converted to base32) is what lets you generate the 8-digit codes in any compatible manager |

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

<a id="sec-14"></a>
## 🙏 Credits & Sources

This method builds on work from the community, in particular:

- [`python-bna` — issue #42][github_bna_issue], which documents the curl method used in this guide
- The contributors behind this method: `@BillyCurtis`, `@Gigafrost`, and `@digikwal` for putting it together and testing it
- [`digikwal/bliz_totp`][github_bliz_totp], a Python script that automates these same steps

<p align="right"><sub><a href="#top">⬆️</a></sub></p>

---

This guide is distributed under the [**CC BY-NC-SA 4.0**][url_license] license.

<!-- ==================== REFERENCE VARIABLES ==================== -->

<!-- Badges (shields.io images) -->
[badge_license]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg
[badge_guide]: https://img.shields.io/badge/Guide%20%3A-BNet_2FA_Sync-blue?style=flat&logo=mdbook&logoColor=white&logoSize=auto&label=Guide%20%3A&labelColor=black&color=darkcyan
[badge_proton]: https://img.shields.io/badge/Proton-6D4AFF?style=flat&logo=proton&logoColor=white&logoSize=auto

<!-- External URLs (third-party services, non-GitHub) -->
[url_license]: https://creativecommons.org/licenses/by-nc-sa/4.0/
[url_proton]: https://proton.me
[url_cryptii]: https://cryptii.com/pipes/hex-to-base32
[url_battlenet]: https://account.battle.net

<!-- GitHub links & local repo files -->
[github_user]: https://github.com/Data-Spirit
[github_repo]: https://github.com/Data-Spirit/All_Guides
[github_bna_issue]: https://github.com/jleclanche/python-bna/issues/42
[github_bliz_totp]: https://github.com/digikwal/bliz_totp
