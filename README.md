# Vitrupes Pludmale

A trilingual tourism website about Vitrupe beach on the Vidzeme coast of Latvia.

**Live site:** [manalasdungeon.lv/vitrupe](https://manalasdungeon.lv/vitrupe/)
**Author:** Marko Nakari ([GitHub](https://github.com/ManalasDungeon) · [LinkedIn](https://www.linkedin.com/in/manalainen/))

The project started as a web publishing exercise during my software developer studies at Vamia. I later rebuilt it from scratch as a real, indexed website with a modern design, multilingual content and a moderated guestbook.

> This repository documents what the site does and how it was built. The source code is not published here, but it is available on request.

---

## Features

- **Three languages:** Latvian (default), English and Finnish, with clean URLs (`/vitrupe/`, `/vitrupe/en/`, `/vitrupe/fi/`) and a flag dropdown language switcher
- **Modern landing page:** full-screen hero, navbar that turns solid on scroll, scroll-reveal animations, parallax quote and a floating beach-sound button
- **Two gallery pages** with a responsive grid, image zoom and an in-page lightbox
- **Traditional Latvian visual theme:** carmine red, amber and linen colours, Playfair Display and Nunito Sans fonts, and woven-band (prievīte) section dividers
- **Moderated guestbook:** visitors can leave a message and optionally suggest their own service or attraction with a link; new entries wait for admin approval
- **SEO:** hreflang, canonical URLs, multilingual sitemap, Open Graph and Twitter cards, and schema.org `TouristAttraction` structured data
- **Performance:** Bootstrap 5.3 without jQuery (JavaScript cut from 252 KB to 80 KB), automatic cache busting, long-term browser caching and gzip

## Tech stack

| Area | Technologies |
|---|---|
| Front end | HTML5, CSS3, Bootstrap 5.3.3, vanilla JavaScript |
| Back end | PHP |
| Database | MySQL (guestbook) |
| Server | Apache: mod_rewrite, mod_expires, mod_headers, mod_deflate |
| Hosting | Shared hosting with cPanel |

---

## Project structure

```
vitrupe/
├── index.php             Landing page
├── kuvat.php             Gallery, page 1
├── kuvat2.php            Gallery, page 2
├── lang.php              Language selection, page_url() and asset() helpers
├── lang/
│   ├── lv.php            Latvian translations
│   ├── en.php            English translations
│   └── fi.php            Finnish translations
├── partials/
│   ├── head.php          Meta, SEO and social tags
│   ├── nav.php           Navbar and language switcher
│   ├── footer.php        Footer and scripts
│   └── jsonld.php        schema.org structured data
├── css/
│   ├── latviatyyli.css   Shared colour theme
│   ├── index.css         Modern layout (landing and gallery pages)
│   ├── kuvatyylit.css    Gallery styles
│   └── jakaja.css        Section dividers
├── js/
│   └── vitrupe.js        Navbar state, scroll reveal, sound toggle, lightbox
├── images/
├── audio/
├── .htaccess             Language URL rewrites and redirects
└── sitemap.xml           Multilingual sitemap
```

The guestbook lives at the site root, outside the `vitrupe/` folder:

```
viesu_gramata.php / .css  Public guestbook
viesu_admin.php / .css    Password-protected moderation panel
```

---

## How the multilingual URLs work

Latvian is served from the folder root. English and Finnish get a path prefix, which `.htaccess` rewrites internally to a `?lang=` parameter:

| Public URL | Served internally as |
|---|---|
| `/vitrupe/` | `index.php` (Latvian) |
| `/vitrupe/en/` | `index.php?lang=en` |
| `/vitrupe/fi/kuvat.php` | `kuvat.php?lang=fi` |
| `/vitrupe/en/css/index.css` | `css/index.css` |

Asset requests under `/en/` and `/fi/` are rewritten back to the real folders, so templates can keep relative paths. The language is read from the URL only, not from a cookie, so the Latvian root always shows Latvian.

Old `.html` addresses are redirected to the `.php` pages with 301 redirects.

---

## Guestbook

Security features:

- CSRF tokens on all forms
- Honeypot field and rate limiting against spam
- IP addresses stored only as SHA-256 hashes
- Output escaped before rendering; only `http`/`https` links are made clickable, with `rel="nofollow noopener ugc"`
- Optional link and category fields validated and whitelisted
- Admin login with a bcrypt hash (`password_hash` / `password_verify`), no plain-text password
- New entries are saved as unapproved and published only after moderation

### Database table

```sql
CREATE TABLE viesu_gramata (
  id           INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  nosaukums    VARCHAR(60)   NOT NULL,
  zina         VARCHAR(1000) NOT NULL,
  saite        VARCHAR(255)  NULL,
  tips         VARCHAR(30)   NULL,
  izveidots    DATETIME      NOT NULL,
  ip_hash      CHAR(64)      NOT NULL,
  apstiprinats TINYINT(1)    NOT NULL DEFAULT 0,
  INDEX (izveidots)
) DEFAULT CHARSET=utf8mb4;
```

---

## Hosting and deployment

The site runs on shared hosting managed through cPanel and is deployed by uploading files to the web root. Apache `mod_rewrite` handles the language URLs, and the root `.htaccess` adds browser caching, gzip compression and hotlink protection for images.

The admin panel has a first-run setup: when no admin hash is configured, it shows a form that generates a bcrypt hash for the configuration, so no plain-text password is ever stored. The sitemap is referenced from `robots.txt` at the domain root, and the site is indexed by search engines.

---

## Image workflow

Phone photos are prepared before upload:

- Resized to 1800 px wide and saved as JPEG at quality 82
- EXIF metadata removed, including GPS location
- File names kept ASCII-only, because non-ASCII names returned 404 errors on the server

This took the second gallery from 32.9 MB to 3.4 MB.

---

## Usage

Photos and audio on the site may not be reused without permission.
