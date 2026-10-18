# 🔗 Shortly (auto-upload edition)

A free URL shortener on GitHub Pages. No sign-in. New links are saved automatically: a small Cloudflare Worker commits them to `links.json` in your repo for you.

## How it works

- **`links.json` in your repo is the database.** It's public, like the rest of the repo, but the destination URLs inside it are encrypted (AES-256-GCM) with a key only the Worker knows. Anyone reading the file sees aliases and scrambled text, not where links go.
- **The Worker (`worker.js`) is the middleman.** It holds a GitHub token privately and makes the commits. The token never goes in the website, because anything in a public page can be copied.
- **Create links with no account.** Alias uniqueness is checked by the Worker against the real file, so an existing alias can't be overwritten.
- **Edit/Delete:** your browser makes a secret key. Links store only a hash of it. Only someone with the same key can edit or delete that link. Open **🔑 My key** in the link list to copy it, and paste it there on another device (**Use key**) to keep control of your links.
- Short links look like `https://<username>.github.io/<repo>/my-alias`.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The app. |
| `404.html` | Identical copy of `index.html`. GitHub Pages serves it for `/repo/my-alias`, which makes the redirect work. |
| `config.js` | Your Worker URL. |
| `privacy.html`, `terms.html`, `cookies.html`, `refunds.html` | Privacy policy, terms and conditions, cookie and storage policy, and refund policy. Linked from the footer. |
| `LICENSE` | All rights reserved. |
| `favicon.svg`, `apple-touch-icon.png` | The site icon (browser tab and phone home screen). |
| `links.json` | The database. Starts empty. |
| `worker.js` | Goes on Cloudflare, not in the repo (you can still keep a copy there). |

## Setup (about 15 minutes, all free)

1. **GitHub repo:** create a public repo (for example `shorturl`) and upload `index.html`, `404.html`, `config.js`, `links.json`, `favicon.svg`, `apple-touch-icon.png` and `README.md`. Turn on **Settings → Pages** (main branch, `/ (root)`).
2. **GitHub token:** go to **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**. Under *Repository access* choose **Only select repositories** and pick this repo. Under *Permissions → Repository permissions* set **Contents: Read and write**. Copy the token. Keep it private.
3. **Cloudflare Worker:** make a free account at cloudflare.com (no card needed). Go to **Workers & Pages → Create → Create Worker**, deploy it, then **Edit code**, replace everything with `worker.js`, and deploy.
4. In the Worker's **Settings → Variables and Secrets**, add:
   - `GH_TOKEN` (type: Secret): the token from step 2
   - `GH_REPO`: `yourname/yourrepo`
   - `GH_BRANCH`: `main`
   - `ALLOWED_ORIGIN`: `https://yourname.github.io`
   - `ENC_KEY` (type: Secret): a long random string you make up (30+ characters). It encrypts every destination URL. Keep a copy somewhere safe: if you lose or change it, existing links can no longer be decrypted.
5. Copy the Worker's URL (like `https://shortly.yourname.workers.dev`) into `config.js` and commit.
6. Open your site and create a link.

## Limitations

- **Encryption hides URLs from the public file, not from the Worker.** The Worker decrypts a URL for anyone who asks for that exact alias (that is how redirects work), so anyone who knows or guesses an alias can find its destination. People can't list everyone's destinations, though: the site only shows you the destinations of your own links. You, as the Worker owner, can decrypt everything.

- **Cloudflare is an outside service.** Auto-upload needs one. A token can't safely live in a public page.
- **Anyone can create links**, including spam or links to harmful sites, and every one becomes a commit in your repo history. There is no rate limit. Delete bad entries by editing `links.json` on GitHub.
- **Ownership by key is not an account.** If someone loses their key (clears browser data without copying it), they can't edit those links any more. You, as repo owner, can still edit `links.json`.
- **GitHub API limits.** The Worker caches reads for about 15 seconds to stay within them. Very heavy traffic could hit limits.
- **Renaming an alias breaks the old short link.**
- Reserved aliases: `admin`, `api`, `new`, `links`.
- Short links are served from GitHub's 404 page, so link checkers and some bots may see a 404 status.

## License

Copyright (c) 2026 karcha3. All rights reserved. This is not open-source software: you may view it and use the live site, but you may not copy, modify, host or redistribute it. See `LICENSE`.

## Accessibility and privacy notes

- The site uses no cookies, ads, analytics, third-party scripts, fonts or embeds, and has no images other than its own icon. Because nothing needs consent, there is no cookie banner. **If you ever add analytics, ads or embeds, add a consent banner and update the policies first.**
- Text and form-border colors were checked against WCAG 2.1 AA contrast (4.5:1 for text, 3:1 for input and button borders) in both light and dark themes.
- All form fields have labels, buttons have descriptive names, the create form works with the Enter key, and edit fields support Enter (save) and Escape (cancel). Test with a keyboard and a screen reader before relying on this.
- The policies are plain-language templates written for how this site works. They are not legal advice. Have a lawyer review them if you run this commercially.
