![CF_logo_stacked_whitetype](https://github.com/luxysiv/Cloudflare-Gateway-Pihole/assets/46205571/b8b7b12b-2fd8-4978-8e3c-2472a4167acb)

**[English](README.md)** | **[Việt Nam](docs/vi.md)**

# Pihole styled, but using Cloudflare Gateway
`For Devs, Ops, and everyone who hates Ads.`

Create your ad blocklist using Cloudflare Gateway.

### Credit goes there
---

> Thanks a lot to [@nhubaotruong](https://github.com/nhubaotruong) for his contributions.

> Readme by [@minlaxz](https://github.com/minlaxz).

>> Added dynamic domain filter (whitelist and blacklist) idea (please check `ini` files, as you may also need to modify those).
>>> Added dynamic domain filter (whitelist and blacklist) to Actions variables (please check [dynamic_blacklist.txt](./lists/dynamic_blacklist.txt) and [dynamic_whitelist.txt](./lists/dynamic_whitelist.txt). to know examples to add `Value*`).Use `DYNAMIC_BLACKLIST` and `DYNAMIC_WHITELIST` for `Name*` in Actions variables 

### Supported styles
---
* Two kinds of lists in ini files: white list [whitelist.ini](./lists/whitelist.ini) and block list [adlist.ini](./lists/adlist.ini).

```ini
https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
```
or
```ini
[Ad-Urls]
Adguard = https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
```

### Custom URLs
---
* Add to file:
  > White list [whitelist.ini](./lists/whitelist.ini) and block list [adlist.ini](./lists/adlist.ini).

* Add to GitHub Action variables:
  > `Name*`
  >> `ADLIST_URLS` or `WHITELIST_URLS`.

  > `Value*` `URLs list`
  >> Example:
  ```text
  https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
  https://raw.githubusercontent.com/hagezi/dns-blocklists/main/wildcard/light-onlydomains.txt
  ```

* You should add your ad list and whitelist to Action variables. If you update your fork, your custom list will not be lost.

### How to set this up?
---
1. Fork this repository to your account.
2. Grab your **Cloudflare Account ID** (found after `https://dash.cloudflare.com/`) from ➞ https://dash.cloudflare.com/?to=/:account/workers.
3. Create your **API Token** from ➞ https://dash.cloudflare.com/profile/api-tokens with 3 permissions:
   1. `Account.Zero Trust : Edit`
   2. `Account.Account Firewall Access Rules : Edit`
   3. `Account.Access: Apps and Policies : Edit`

4. Add **Repository Secrets** to your forked repository:
`➞ https://github.com/<username>/<forked-repository>/settings/secrets/actions`
   1. Set **Cloudflare Account ID** to `CF_IDENTIFIER`.
   2. Set **API Token** to `CF_API_TOKEN`.

### Schedule
---
> Due to a limited 2-month commitment from GitHub Actions, you can create and paste this code to run on Cloudflare Workers. Remember, GitHub Tokens generate with no expiration and all permissions.

```javascript
addEventListener('scheduled', event => {
  event.waitUntil(handleScheduledEvent());
});

async function handleScheduledEvent() {
  const GITHUB_TOKEN = 'YOUR_GITHUB_TOKEN_HERE';
  try {
    const dispatchResponse = await fetch('https://api.github.com/repos/YOUR_USER_NAME/YOUR_REPO_NAME/actions/workflows/main.yml/dispatches', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${GITHUB_TOKEN}`,
        'Content-Type': 'application/json',
        'User-Agent': 'Mozilla/5.0',
      },
      body: JSON.stringify({
        ref: 'main'
      }),
    });

    if (!dispatchResponse.ok) throw new Error('Failed to dispatch workflow');
  } catch (error) {
    console.error('Error handling scheduled event:', error);
  }
}
```
>> Remember to set up Cloudflare Workers triggers.

### How to set up using Termux?
---

To use this tool on the **GOAT** [Termux](https://github.com/termux/termux-app/releases/latest), follow the steps below. If you are already familiar with setting up Python and the basics, you can skip this section.

#### Method 1:

1. Open Termux and run the following commands one by one:

```sh
yes | pkg upgrade
yes | pkg install python-pip
yes | pkg install git
# Clone your forked repo #
git clone https://github.com/<username>/<repo-name>.git
```

2. Navigate to the cloned repository folder:

```sh
cd <repo-name>
```

3. Edit the `.env` file (required):

```sh
nano .env
```

After editing, press `CTRL + X`, then `Y`, and `ENTER` to save the file.

4. Run the command to upload (update) your DNS list:

```sh
python -m src
```

#### Method 2:

1. Download the ZIP file of the repository from the 'Code' button on the GitHub page and select 'Download ZIP'.

2. Unzip the downloaded file.

3. Edit the values in `.env` and `adlist.ini` etc...

4. Open Termux and enter the following commands to set up Python and necessary tools:

```sh
yes | pkg upgrade
yes | pkg install python-pip
termux-setup-storage
```

5. Allow Termux to access storage.

6. Navigate to the folder containing the unzipped source code:

```sh
cd storage/downloads/Cloudflare-Gateway-Pihole-main
```

7. Run the command to upload (update) your DNS list:

```sh
python -m src
```

If you encounter issues during setup, you can refer to [termux-change-repo](https://wiki.termux.com/wiki/Package_Management) for changing Termux repositories.

### Note
---
* The **limit** of `Cloudflare Gateway Zero Trust` free is **300k domains**, so remember to pay attention to the workflow logs. If it is exceeded, the script will stop.

* If you have uploaded lists using another script, you should delete them using the delete feature of the uploaded script or delete them manually.

* I have updated the feature to delete lists when you no longer need to use the script. Go to [__main__.py](src/__main__.py) as follows:

```python
if __name__ == "__main__":
    cloudflare_manager = CloudflareManager(PREFIX, MAX_LISTS, MAX_LIST_SIZE)
    # cloudflare_manager.run()
    cloudflare_manager.leave() # Leave script 
```

Note from [@minlaxz](https://github.com/minlaxz):
1. Domain list style: I personally preferred the second one in blacklist styles, which is more readable and concise.
2. Dynamic domain list: You can also update your dynamic (fluid) whitelist and blacklist using [dynamic_blacklist.txt](./lists/dynamic_blacklist.txt) and [dynamic_whitelist.txt](./lists/dynamic_whitelist.txt).
3. Deprecated using `.env`: Setting sensitive information inside a public repository is considered too dangerous, since any unwanted person could easily steal your Cloudflare credentials from that `.env` file.

🥂🥂 Cheers! 🍻🍻
===
