# whitneyindustries.com — OAuth branding site for MW Goo Bot

Static two-page site required by Google to publish the `mwgoobot@gmail.com` OAuth app
(the one `gog` uses) to production. Publishing kills the 7-day refresh-token expiry.

**Registrar:** Namecheap · **Default host:** apex `whitneyindustries.com` (via GitHub Pages)

## Files

| File | Purpose |
|---|---|
| `index.html` | App homepage (describes the app — Google requires this) |
| `privacy.html` | Privacy policy (discloses Google-data access — required) |
| `CNAME` | GitHub Pages custom-domain binding (currently `whitneyindustries.com`) |

## Publish runbook

### 0. Apex vs subdomain (decision)
Default here: **apex** (`https://whitneyindustries.com`). If you'd rather keep the apex
free for a future "real" site, use `bot.whitneyindustries.com` instead: edit `CNAME` to
match, and in step 2 add `CNAME bot → kramimus.github.io.` instead of A records.

### 1. GitHub Pages
```bash
# on your machine, under your GitHub account:
gh repo create whitneyindustries.com --public --source=. --push
# or classic: create repo `whitneyindustries.com`, git remote add + push
```
- Repo → Settings → Pages → Build and deployment: **Deploy from branch**, `main`, `/ (root)`
- Custom domain: `whitneyindustries.com` → Save (the `CNAME` file already declares this)
- Check **Enforce HTTPS** once the cert provisions (may take minutes–hours after DNS)

### 2. Namecheap DNS (Domain List → Manage → Advanced DNS)
Delete any parking/redirect records Namecheap added, then add:

| Type | Host | Value | TTL |
|---|---|---|---|
| A | `@` | `185.199.108.153` | Automatic |
| A | `@` | `185.199.109.153` | Automatic |
| A | `@` | `185.199.110.153` | Automatic |
| A | `@` | `185.199.111.153` | Automatic |
| CNAME | `www` | `kramimus.github.io.` | Automatic |

Verify: `dig +short whitneyindustries.com` returns the four 185.199.x.x addresses
(propagation usually 5–30 min).

### 3. Search Console domain verification
- https://search.google.com/search-console → **Add property → Domain** → `whitneyindustries.com`
  (Domain type covers all subdomains; don't pick URL-prefix)
- Copy the `google-site-verification=…` TXT record it shows
- Namecheap Advanced DNS → Add record: `TXT` · `@` · `<verification string>` · Automatic
- Back in Search Console → **Verify**

### 4. GCP wiring (the project that owns the gog OAuth client)
Console → **Google Auth Platform / APIs & Services → OAuth consent screen (Branding)**:

- App name: `MW Goo Bot`
- User support email: `mwgoobot@gmail.com` (or your personal — must be a real inbox)
- Application homepage: `https://whitneyindustries.com`
- Privacy policy link: `https://whitneyindustries.com/privacy.html`
- Authorized domains: add `whitneyindustries.com`
- Skip the logo (only displays after verification — not needed)

### 5. Publish
Same console area → **Audience / Publishing status: Testing** → **PUBLISH APP** →
*Push to production* → confirm. When Google offers verification, decline/skip:
single-user personal apps don't need it (cost of skipping = bypassable
"unverified app" screen on *future* consent flows only).

### 6. Post-publish check
Existing tokens are unaffected — verify nothing broke:
```bash
gog gmail search 'newer_than:1d' --max 3
```

## Troubleshooting
- **Pages 404 after DNS**: wait for propagation, confirm the four A records, re-check
  Settings → Pages shows the custom domain + cert
- **Search Console TXT not verifying**: DNS can take up to a few hours; retry later
- **Consent screen rejects the privacy URL**: it must be the exact same URL on the
  homepage link and the console field — `https://whitneyindustries.com/privacy.html`
