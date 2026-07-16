# 🔐 Garena Authentication Flow Specification

A complete, reverse‑engineered technical specification of the **Garena authentication flow**, covering SSO, CODM (Call of Duty Mobile) OAuth, DataDome anti‑bot protection, game connections, and the proprietary password hashing algorithm.

This document is intended for **security research**, **authorized penetration testing**, and **educational purposes** only.

---

## 📋 Overview

The Garena ecosystem uses a complex multi‑domain authentication flow involving:

- **SSO (Single Sign‑On)** – `sso.garena.com` and `account.garena.com`
- **CODM OAuth** – `100082.connect.garena.com`
- **DataDome Anti‑Bot** – `datadome.garena.com`
- **Game Connections API** – regional shop domains (e.g., `shop.garena.sg`, `termgame.com`, etc.)
- **Password Hashing** – a custom AES‑ECB + SHA256 algorithm

This specification documents the exact HTTP requests, headers, cookies, parameters, and responses required to authenticate and retrieve user data.

---

## 📁 Endpoints & Domains

| Domain | Purpose |
|--------|---------|
| `sso.garena.com` | Prelogin, Login (SSO) |
| `account.garena.com` | Account info, session management |
| `100082.connect.garena.com` | CODM OAuth grant & token exchange |
| `api-delete-request-aos.codm.garena.co.id` | CODM callback & user info |
| `authgop.garena.com` | Shop access token grant |
| `shop.garena.sg` (regional) | Game connections API |
| `datadome.garena.com` | DataDome cookie generation (anti‑bot) |

---

## 🔄 Authentication Flow

| Step | Action | Endpoint | Description |
|------|--------|----------|-------------|
| 1 | **Prelogin** | `sso.garena.com/api/prelogin` | Get `v1` and `v2` parameters for hashing. |
| 2 | **Login** | `sso.garena.com/api/login` | Submit hashed password; receive `sso_key`. |
| 3 | **Account Init** | `account.garena.com/api/account/init` | Fetch user profile, email, phone, binds, shell balance, login history. |
| 4 | **CODM Grant** | `100082.connect.garena.com/oauth/token/grant` | Get authorization `code` for CODM. |
| 5 | **Token Exchange** | `100082.connect.garena.com/oauth/token/exchange` | Exchange `code` for `access_token`. |
| 6 | **CODM Callback** | `api-delete-request-aos.codm.garena.co.id/oauth/callback/` | Extract `codm-delete-token`. |
| 7 | **CODM Check Login** | `api-delete-request-aos.codm.garena.co.id/oauth/check_login/` | Get CODM user info (level, IGN, region, UID). |
| 8 | **Game Connections** | Regional shop domains | (Optional) Fetch roles for all connected Garena games (Free Fire, ROV, Delta Force, etc.). |

---

## 🔑 Password Hashing Algorithm

The password is **not** sent in plain text. Garena uses a custom hashing process:

1. **passmd5** = `MD5(urldecode(password))`
2. **inner_hash** = `SHA256(passmd5 + v1)`
3. **outer_hash** = `SHA256(inner_hash + v2)`
4. **final** = `AES_ECB_encrypt(passmd5, key=outer_hash)` → hex (first 32 characters)

This final hash is sent as the `password` parameter in the login request.

---

## 🎮 Game IDs & Mappings

| App ID | Game Name |
|--------|-----------|
| `100082` | CODM |
| `100067` | Free Fire |
| `100055` / `100054` | ROV |
| `100151` | Delta Force |
| `100070` | Speed Drifters |
| `100130` | Black Clover M |
| `100105` | Garena Undawn |
| `100057` | AOV |

*(See the full specification for complete mapping per region.)*

---

## 🛡️ DataDome Anti‑Bot

DataDome is used to prevent automated requests. The spec includes the full request payload for generating a valid `datadome` cookie, including the dynamic `jspl`, `cid`, and `ddk` fields.

---

## 📦 Requirements

This is a **specification document** – no code is required to read it. However, if you intend to implement it, you will need:

- Python 3.8+
- `requests` library
- `pycryptodome` (for AES‑ECB encryption)
- Basic understanding of HTTP, cookies, and JSON.

---

## ⚠️ Disclaimer

This document is provided **solely for educational and research purposes**. Unauthorised use of this information to access accounts without permission is illegal and unethical. The author assumes no responsibility for misuse.
