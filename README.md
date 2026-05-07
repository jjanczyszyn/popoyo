# popoyo.co

Landing page for Popoyo — surf, moto rentals, concierge, beauty, childcare and cash delivery on Nicaragua's Emerald Coast.

## Deploy

This is a single static `index.html` — no build step.

### GitHub Pages (recommended for `popoyo.co`)

1. Push these files to the root of the `popoyo` repo on GitHub.
2. Go to **Settings → Pages**.
3. Source: **Deploy from branch** → **main** → **/ (root)**.
4. Custom domain: `popoyo.co` (the `CNAME` file is already in this repo).
5. Add these DNS records at your domain registrar:
   - `A` records on `popoyo.co` pointing to GitHub Pages IPs:
     `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
   - `CNAME` record on `www.popoyo.co` → `<your-github-username>.github.io`
6. Back in GitHub Pages settings, tick **Enforce HTTPS** once the cert provisions (a few minutes).

### Push from terminal

```bash
git clone https://github.com/<you>/popoyo.git
cd popoyo
# drop these files in
git add .
git commit -m "Initial landing page"
git push
```

### Or upload via web UI

Go to `github.com/<you>/popoyo` → **Add file → Upload files** → drag the contents of this bundle in → commit.

## Files

- `index.html` — the landing page (self-contained, uses Google Fonts CDN)
- `CNAME` — tells GitHub Pages to serve at `popoyo.co`
- `robots.txt` — allow crawlers, point to sitemap
- `sitemap.xml` — single-page sitemap for SEO

## Subdomains

The page links to:

- `moto.popoyo.co` — Karen & JJ Moto Rental
- `cash.popoyo.co` — Cash delivery
- `beauty.popoyo.co` — In-home beauty
- `childcare.popoyo.co` — Nannies & babysitting
- `concierge.popoyo.co` — Full concierge

Set each as its own DNS record / project on whatever host you use for those.

## Contact

- WhatsApp: +505 8975 0052
- Email: hello@popoyo.co
