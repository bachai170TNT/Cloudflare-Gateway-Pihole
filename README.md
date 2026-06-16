**[English](README.md)** | **[Tiếng Việt](docs/vi.md)**

![Cloudflare Gateway](https://github.com/luxysiv/Cloudflare-Gateway-Pihole/assets/46205571/b8b7b12b-2fd8-4978-8e3c-2472a4167acb)

# Cloudflare Gateway DNS Filter — Block + Allow

> Pihole-style DNS ad blocking using Cloudflare Gateway Zero Trust, with a dedicated Allow rule that takes precedence over blocking.

`For Devs, Ops, and everyone who hates Ads.`

---

## How it works

This script manages two sets of Cloudflare Gateway DNS rules in a single workflow run:

| Rule | Action | Precedence | Source |
| :--- | :--- | :--- | :--- |
| `[AdBlock-DNS-Filters] Block Ads` | block | 1000 | `adlist.ini` + `dynamic_blacklist.txt` |
| `[AdAllow-DNS-Filters] Allow` | allow | 999 *(higher priority)* | `whitelist.ini` + `dynamic_whitelist.txt` |

Because the Allow rule has a **lower precedence number** (999 < 1000), Cloudflare evaluates it first — so whitelisted domains are always resolved even if they appear in a blocklist. There is no need to subtract the whitelist from the block list in code.

The script also enforces Cloudflare Gateway's **free tier limit of 300 lists** across both rules combined. If your sources would require more than 300 lists, the script stops before making any changes.

---

## Setup

### 1. Fork this repository

### 2. Get your Cloudflare credentials

- **Account ID** — found in the URL after `https://dash.cloudflare.com/`:
  `https://dash.cloudflare.com/?to=/:account/workers`

- **API Token** — create at `https://dash.cloudflare.com/profile/api-tokens` with these 3 permissions:
  1. `Account › Zero Trust : Edit`
  2. `Account › Account Firewall Access Rules : Edit`
  3. `Account › Access: Apps and Policies : Edit`

### 3. Add Repository Secrets

Go to `https://github.com/<username>/Cloudflare-Gateway-DNS-Filter/settings/secrets/actions` and add:

| Secret | Value |
| :--- | :--- |
| `CF_API_TOKEN` | Your Cloudflare API Token |
| `CF_IDENTIFIER` | Your Cloudflare Account ID |

### 4. Configure your lists

**Block list** — edit [`lists/adlist.ini`](./lists/adlist.ini):
```ini
[Ad-Urls]
Adguard = https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
```

**Allow list** — edit [`lists/whitelist.ini`](./lists/whitelist.ini):
```ini
[Allow-Urls]
MyAllow = https://example.com/my-whitelist.txt
```

Both files accept plain URLs (one per line) or the `[Section] key = url` INI format.

### 5. (Optional) Add lists via GitHub Actions Variables

Go to `https://github.com/<username>/Cloudflare-Gateway-DNS-Filter/settings/variables/actions` and add:

| Name | Value |
| :--- | :--- |
| `ADLIST_URLS` | Space-separated block list URLs |
| `WHITELIST_URLS` | Space-separated allow list URLs |
| `DYNAMIC_BLACKLIST` | Extra domains to block (one per line) |
| `DYNAMIC_WHITELIST` | Extra domains to allow (one per line) |

Using Variables keeps your custom lists safe when you pull updates to the repo.

You can also edit the local dynamic files directly:
- [`lists/dynamic_blacklist.txt`](./lists/dynamic_blacklist.txt)
- [`lists/dynamic_whitelist.txt`](./lists/dynamic_whitelist.txt)

---

## Supported list formats

```
# Plain URL
https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
```
```ini
# INI format
[Ad-Urls]
Adguard = https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
```

Supported domain list styles: hosts files, AdBlock/uBlock syntax, plain domain lists.

> **Note for Vietnamese users:** Use **DNS filter lists**, not browser extension filter lists. Browser lists contain cosmetic rules that cause errors when used as DNS blocklists.

---

## Running locally (Termux or any terminal)

### Method 1 — Clone from GitHub

```sh
# Install dependencies
yes | pkg upgrade
yes | pkg install python-pip git

# Clone your fork
git clone https://github.com/<username>/Cloudflare-Gateway-DNS-Filter.git
cd Cloudflare-Gateway-DNS-Filter

# Edit credentials
nano .env
```

Run the sync:
```sh
python -m src run
```

Delete all managed lists and rules:
```sh
python -m src leave
```

### Method 2 — Download ZIP

1. Download the ZIP from the GitHub page → **Code → Download ZIP**.
2. Unzip and edit `.env`, `lists/adlist.ini`, `lists/whitelist.ini`.
3. Open Termux:

```sh
yes | pkg upgrade
yes | pkg install python-pip
termux-setup-storage
cd storage/downloads/Cloudflare-Gateway-DNS-Filter-main
python -m src run
```

---

## Workflow schedule

The workflow runs automatically every **Friday at midnight UTC** and on every push. You can also trigger it manually from the **Actions** tab.

To keep GitHub Actions running indefinitely (it auto-disables after 60 days of no pushes), use a Cloudflare Worker to trigger the workflow on a schedule:

```javascript
addEventListener('scheduled', event => {
  event.waitUntil(handleScheduledEvent());
});

async function handleScheduledEvent() {
  const GITHUB_TOKEN = 'YOUR_GITHUB_TOKEN';   // needs workflow permission, no expiry
  const GITHUB_USER  = 'YOUR_USERNAME';
  const GITHUB_REPO  = 'YOUR_REPO_NAME';
  const WORKFLOW_ID  = 'main.yml';

  const url = `https://api.github.com/repos/${GITHUB_USER}/${GITHUB_REPO}/actions/workflows/${WORKFLOW_ID}/dispatches`;

  const res = await fetch(url, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${GITHUB_TOKEN}`,
      'Content-Type': 'application/json',
      'User-Agent': 'Cloudflare-Worker-Trigger',
    },
    body: JSON.stringify({ ref: 'main' }),
  });

  if (!res.ok) throw new Error(`Dispatch failed: ${res.status}`);
  console.log('GitHub Action triggered successfully');
}
```

Set a **Cron Trigger** in your Cloudflare Worker settings (e.g. `0 0 * * 5`).

---

## Deleting all managed resources

Change the workflow step command to `leave` and run the workflow once:

```yml
- name: Cloudflare Gateway Zero Trust
  run: python -m src leave
```

This removes all lists and rules created by this script from your Cloudflare account.

---

## Limits

| Limit | Value |
| :--- | :--- |
| Domains per list | 1,000 |
| Total lists (block + allow combined) | ≤ 300 (free tier) |
| Domains total (300 × 1,000) | ≤ 300,000 |

The script calculates the number of lists needed before making any API calls and stops with a clear error if the limit would be exceeded.

---

## Credits

> Thanks to [@nhubaotruong](https://github.com/nhubaotruong) for contributions.

> Original README by [@minlaxz](https://github.com/minlaxz).

🥂 Cheers! 🍻
