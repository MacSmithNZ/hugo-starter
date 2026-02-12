# Hugo Starter

A Hugo module that provides base layouts, Tailwind CSS v4, SEO metadata, and responsive navigation. Import it into any Hugo project to get a production-ready foundation.

## What This Module Provides

| Feature | Details |
|---------|---------|
| Tailwind CSS v4 | Native Hugo integration via `css.TailwindCSS`, auto-minified in production |
| Typography plugin | `prose` class for styled Markdown content |
| SEO meta tags | Open Graph, Twitter Cards, JSON-LD, canonical URLs |
| Responsive nav | Desktop links + mobile hamburger menu, driven by `menus.main` |
| Favicons | `.ico`, `.svg`, and Apple touch icon placeholders |
| Security headers | Netlify `_headers` with sensible defaults |
| Sitemap | Auto-generated at `/sitemap.xml` |
| RSS | Auto-generated at `/index.xml` |
| 404 page | Custom error page |
| Archetype | Default front matter with SEO fields (`description`, `image`) |

## Prerequisites

- [Hugo](https://gohugo.io/installation/) v0.128.0+
- [Go](https://go.dev/doc/install) v1.18+ (required for Hugo modules)
- [Node.js](https://nodejs.org/) v18+

## Usage

### 1. Create a new Hugo site

```bash
hugo new site my-site
cd my-site
```

### 2. Initialize a Go module

```bash
hugo mod init github.com/your-user/my-site
```

### 3. Install npm dependencies

The consuming project needs Tailwind CSS installed locally:

```bash
npm init -y
npm install --save-dev tailwindcss @tailwindcss/cli @tailwindcss/typography
```

### 4. Configure your site

Replace `hugo.toml` with the following, customizing the values for your project:

```toml
baseURL = 'https://example.com/'
languageCode = 'en'
title = 'My Site'
enableRobotsTXT = true

# --- Tailwind CSS build stats ---
[build]
  [build.buildStats]
    enable = true

# --- Module import ---
[[module.imports]]
  path = 'github.com/MacSmithNZ/hugo-starter'

# --- Mounts (required for Tailwind class detection) ---
[[module.mounts]]
  source = 'assets'
  target = 'assets'
[[module.mounts]]
  source = 'hugo_stats.json'
  target = 'assets/notwatching/hugo_stats.json'
  disableWatch = true

# --- Site params (used by SEO partial) ---
[params]
  description = 'My site description'
  author = 'My Name'
  ogImage = '/images/og-default.jpg'

  [params.social]
    twitter = ''

# --- Navigation (used by header partial) ---
[[menus.main]]
  name = 'Home'
  url = '/'
  weight = 1
[[menus.main]]
  name = 'About'
  url = '/about/'
  weight = 2

# --- Sitemap & RSS ---
[sitemap]
  changefreq = 'weekly'
  priority = 0.5

[outputs]
  home = ['HTML', 'RSS']

[markup.goldmark.renderer]
  unsafe = false
```

### 5. Fetch the module and start developing

```bash
hugo mod get
hugo server
```

Your site will be available at `http://localhost:1313/`.

### 6. Build for production

```bash
hugo --minify
```

## Module Structure

These are the files provided by the module:

```
archetypes/
└── default.md                # Default front matter with SEO fields

layouts/
├── _default/
│   ├── baseof.html           # Base template (all pages extend this)
│   ├── single.html           # Single content pages
│   └── list.html             # List/section pages
├── 404.html                  # Custom 404 page
├── index.html                # Homepage template
└── partials/
    ├── css.html              # Tailwind CSS processing via Hugo Pipes
    ├── head.html             # <head> (meta, favicons, CSS)
    ├── seo.html              # Open Graph, Twitter Card, JSON-LD
    ├── header.html           # Responsive nav with mobile hamburger
    └── footer.html           # Site footer

assets/
└── css/
    └── main.css              # Tailwind CSS entry point

static/
├── _headers                  # Netlify security headers
├── favicon.ico               # Placeholder favicon (16/32/48px)
├── favicon.svg               # Placeholder SVG favicon
└── apple-touch-icon.png      # Placeholder Apple touch icon (180x180)
```

## Overriding Templates

To customize any layout, create the same file path in your project's `layouts/` directory. Hugo will use your version instead of the module's.

For example, to customize the homepage:

```bash
mkdir -p layouts
```

Then create `layouts/index.html`:

```html
{{ define "main" }}
  <h1>My custom homepage</h1>
{{ end }}
```

To override the CSS entry point, create `assets/css/main.css` in your project:

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";
@source "hugo_stats.json";

/* Your custom styles */
```

## Tailwind CSS

Tailwind v4 is integrated natively via Hugo's [`css.TailwindCSS`](https://gohugo.io/functions/css/tailwindcss/) function — no PostCSS required.

- **Entry file:** `assets/css/main.css`
- **Processing:** `layouts/partials/css.html` handles compilation, minification (production), and fingerprinting
- **Class detection:** Hugo generates `hugo_stats.json` with all CSS classes used across templates, which Tailwind reads via the `@source` directive

The [`@tailwindcss/typography`](https://tailwindcss.com/docs/typography-plugin) plugin is included for styling rendered Markdown content with the `prose` class.

## SEO

The `seo.html` partial automatically generates:

- **Canonical URL**
- **Open Graph** tags (`og:title`, `og:description`, `og:url`, `og:image`, `og:locale`, etc.)
- **Twitter Card** tags (summary large image)
- **JSON-LD** structured data (`WebSite` schema)

### Per-page SEO

Set `description` and `image` in a page's front matter to override the site-level defaults. The default archetype includes these fields automatically when you run `hugo new`.

The `image` field is resolved in order: page bundle resource, global resource (`assets/`), then static path or URL. All of these work:

```yaml
# Page bundle resource (e.g., content/posts/my-post/cover.jpg)
image: "cover.jpg"

# Global resource from assets/ folder
image: "images/og-banner.jpg"

# Static file or external URL
image: "/images/og.jpg"
image: "https://example.com/image.jpg"
```

## Site Params Reference

These params are used by the module's partials:

| Param | Used by | Purpose |
|-------|---------|---------|
| `params.description` | `head.html`, `seo.html` | Fallback meta description |
| `params.author` | `seo.html` | Site author |
| `params.ogImage` | `seo.html` | Default Open Graph image |
| `params.social.twitter` | `seo.html` | Twitter `@` handle for cards |
| `menus.main` | `header.html` | Navigation menu items |

## Deploying to Netlify

The module includes a `static/_headers` file with security headers that Netlify picks up automatically. To deploy, add a `netlify.toml` to your consuming project:

```toml
[build]
  command = "hugo --minify"
  publish = "public"

[build.environment]
  HUGO_VERSION = "0.155.3"
  GO_VERSION = "1.20"
  NODE_VERSION = "20"

[context.production.environment]
  HUGO_ENV = "production"
```

### Included security headers

The `_headers` file sets the following on all routes:

| Header | Value |
|--------|-------|
| `X-Frame-Options` | `DENY` |
| `X-Content-Type-Options` | `nosniff` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` |

To override, create your own `static/_headers` in the consuming project.

## Replacing Favicons

The included favicons are placeholders. Replace them by adding your own files to your project's `static/` directory:

- `static/favicon.ico` — 16/32/48px multi-size ICO
- `static/favicon.svg` — scalable SVG version
- `static/apple-touch-icon.png` — 180x180px PNG

Tools like [favicon.io](https://favicon.io/) or [realfavicongenerator.net](https://realfavicongenerator.net/) can generate all formats from a single image.

## License

ISC
