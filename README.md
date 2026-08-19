# growthcds-decap-poc

Proof-of-concept: Decap CMS bolted onto a copy of the plain-HTML growthcds.com.au
site, with a client-editable homepage content layer that auto-deploys via GitHub Pages.

This is a **test copy**, detached from the real `shbng-luke/growthcds` repo/site —
nothing here touches the live site or its git history.

## What's here

- `index.html` etc. — the static site, unmodified except for `data-cms-field`
  hooks on 7 fields and a small hydration `<script>` before `</body>` in
  `index.html` that fetches `content/home.json` and fills them in at runtime.
- `content/home.json` — the single editable content file (hero headline/subtext,
  a hero stat, phone number, one testimonial).
- `admin/index.html` + `admin/config.yml` — the Decap CMS admin panel.

## One-time setup still required (manual, needs account access this session doesn't have)

### 1. Register a GitHub OAuth App

Decap's GitHub backend needs a real OAuth App (not a GitHub App) to log editors in.

1. Go to **https://github.com/organizations/thriiv-digital/settings/applications**
   (or your personal https://github.com/settings/developers if testing under your
   own account instead) → **New OAuth App**.
2. Fill in:
   - **Application name**: `growthcds-decap-poc` (anything)
   - **Homepage URL**: `https://thriiv-digital.github.io/growthcds-decap-poc/`
   - **Authorization callback URL**:
     `https://growthcds-decap-oauth.mattwilson-au.workers.dev/callback`
3. Click **Register application**, then **Generate a new client secret**.
4. Copy the **Client ID** and **Client Secret** — you'll need them in step 2.

### 2. Set the OAuth proxy Worker's secrets

The Worker (`growthcds-decap-oauth`, already deployed) reads these at runtime.
From the `decap-oauth-worker/` project directory, run each of these yourself
(they prompt for the value interactively, so the secret never has to be typed
into chat or a shell one-liner):

```bash
npx wrangler secret put GITHUB_CLIENT_ID
npx wrangler secret put GITHUB_CLIENT_SECRET
```

### 3. Give the CMS editor access to the repo

Whoever will log into `/admin` needs at least **write** access to
`thriiv-digital/growthcds-decap-poc` (Decap commits straight to the branch in
`admin/config.yml` — there's no separate reviewer step in this default setup).

### 4. Enable GitHub Pages

In the repo's **Settings → Pages**, set source to **Deploy from a branch** →
`main` / `/ (root)`. First deploy takes a minute or two; after that, every
push (including CMS commits) rebuilds automatically.

## Testing the loop

1. Visit `https://thriiv-digital.github.io/growthcds-decap-poc/admin/`.
2. Log in with GitHub (OAuth popup).
3. Open **Homepage**, change a field (e.g. the phone number), **Save**.
4. Decap commits directly to `main` as the logged-in user.
5. Wait for the Pages build to finish (Actions tab, or the small deployment
   indicator on the repo's front page), then reload the live site — the field
   should show the new value.
