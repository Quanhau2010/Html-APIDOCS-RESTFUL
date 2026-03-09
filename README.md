# APIDOCS-RESTFUL

> **Interactive API Documentation Dashboard** — A zero-dependency, single-file HTML dashboard for browsing, testing, and documenting RESTful API endpoints. Driven entirely by two JSON config files.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
  - [settings.json](#settingsjson)
  - [info.json](#infojson)
- [How It Works](#how-it-works)
- [Supported HTTP Methods](#supported-http-methods)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Dependencies](#dependencies)
- [Browser Support](#browser-support)
- [Customization](#customization)

---

## Overview

APIDOCS-RESTFUL is a lightweight, **no-build, no-framework** API documentation interface. Drop it alongside two JSON files on any static file server and you get a fully functional API explorer — no Node.js, no bundler, no backend required.

```
┌─────────────────────────────────────────────────────┐
│  Static file server (Apache / Nginx / GitHub Pages) │
│                                                     │
│   index.html  ←── reads ──→  settings.json          │
│                              info.json              │
└─────────────────────────────────────────────────────┘
```

---

## Features

| Feature | Description |
|---|---|
| **Zero dependencies** | Pure HTML + CSS + Vanilla JS. No npm, no build step |
| **Data-driven** | All content comes from `settings.json` & `info.json` |
| **Live API testing** | Execute GET/POST requests directly from the browser |
| **JSON syntax highlighting** | Color-coded response viewer |
| **Image response support** | Renders `image/*` responses inline |
| **Collapsible sidebar** | Category-grouped endpoint navigation |
| **Full-text search** | Filter endpoints by name or description (`Ctrl+K`) |
| **Notification system** | Read/unread alerts with `localStorage` persistence |
| **Responsive** | Mobile-first, works on all screen sizes |
| **Keyboard accessible** | `Ctrl+K` to search, `Escape` to close |

---

## Project Structure

```
apidocs-restful/
├── index.html        # Main dashboard (single file, self-contained)
├── settings.json     # UI config: name, version, links, notifications
├── info.json         # API catalog: categories and endpoint definitions
└── README.md
```

> ⚠️ All three files must be served from the **same origin**. Opening `index.html` directly via `file://` will fail due to browser CORS restrictions on `fetch()`.

---

## Quick Start

**1. Clone or download the project**

```bash
git clone https://github.com/your-username/apidocs-restful.git
cd apidocs-restful
```

**2. Serve with any static server**

```bash
# Python 3
python -m http.server 8080

# Node.js (npx, no install needed)
npx serve .

# PHP
php -S localhost:8080
```

**3. Open in browser**

```
http://localhost:8080
```

**4. Edit `settings.json` and `info.json`** to add your own project name, links, and API endpoints.

---

## Configuration

### `settings.json`

Controls the dashboard UI: branding, header image, external links, and notification alerts.

```json
{
  "name": "My API",
  "version": "v2.1.0",
  "description": "REST API for the My App platform",
  "header": {
    "status": "All systems operational",
    "imageSrc": "/assets/banner.png"
  },
  "links": [
    { "name": "GitHub",       "url": "https://github.com/your-org/your-repo" },
    { "name": "Postman Docs", "url": "https://postman.com/..." }
  ],
  "notifications": [
    {
      "title": "v2.1.0 Released",
      "message": "New batch endpoint added. See /api/batch for details."
    }
  ]
}
```

**Field reference:**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | `string` | ✅ | Dashboard title shown in navbar and sidebar |
| `version` | `string` | ✅ | Version badge displayed next to the title |
| `description` | `string` | ✅ | Subtitle shown in the hero section |
| `header.status` | `string` | ❌ | Status text shown in the online pill indicator |
| `header.imageSrc` | `string \| string[]` | ❌ | Hero banner image URL. Pass an array to pick one randomly on each load |
| `links` | `array` | ❌ | External links rendered as buttons in the hero section |
| `links[].name` | `string` | ✅ | Button label |
| `links[].url` | `string` | ✅ | Link destination (opens in new tab) |
| `notifications` | `array` | ❌ | In-app alert messages. Read state is persisted in `localStorage` |
| `notifications[].title` | `string` | ✅ | Alert heading |
| `notifications[].message` | `string` | ✅ | Alert body text |

---

### `info.json`

Defines the full API catalog — all categories and their endpoint items.

```json
{
  "categories": [
    {
      "name": "Authentication",
      "items": [
        {
          "name": "Login",
          "desc": "Authenticate user and return access token",
          "method": "POST",
          "path": "/api/auth/login",
          "author": "dev-team"
        },
        {
          "name": "Get Profile",
          "desc": "Fetch current user profile",
          "method": "GET",
          "path": "/api/user/profile?uid=",
          "author": "dev-team"
        }
      ]
    },
    {
      "name": "Posts",
      "items": [
        {
          "name": "Get All Posts",
          "desc": "Returns paginated list of posts",
          "method": "GET",
          "path": "/api/posts?page=1&limit=10"
        }
      ]
    }
  ]
}
```

**Field reference:**

| Field | Type | Required | Description |
|---|---|---|---|
| `categories` | `array` | ✅ | Top-level groupings. Auto-sorted alphabetically |
| `categories[].name` | `string` | ✅ | Category label shown in sidebar |
| `categories[].items` | `array` | ✅ | Endpoint list. Auto-sorted alphabetically by name |
| `items[].name` | `string` | ✅ | Endpoint display name |
| `items[].desc` | `string` | ❌ | Short description shown in the test modal |
| `items[].method` | `string` | ✅ | HTTP method: `GET`, `POST`, `PUT`, `DELETE`, `PATCH` |
| `items[].path` | `string` | ✅ | Endpoint path. Query string keys auto-generate input fields |
| `items[].author` | `string` | ❌ | Optional author / team tag |

#### Path & Query Parameter Handling

The dashboard parses the `path` field to detect parameters automatically:

**No query string** — request fires immediately when clicked:
```
/api/health
```

**Query string present** — input fields are generated for each key; user fills in values then clicks **Execute**:
```
/api/search?query=&page=1
```
Generates two inputs: `query` (empty) and `page` (pre-filled with `1`).

**POST with params** — query keys become JSON body fields instead of URL params:
```
/api/users/create?name=&email=&role=
```

---

## How It Works

```
User clicks endpoint in sidebar
        │
        ▼
openModal() — reads dataset from DOM element
        │
        ├── Has query params? ──Yes──► Render input fields
        │                              User fills values → clicks Execute
        │                                      │
        └── No params? ────────────────────────┘
                                        │
                                        ▼
                                  execReq(url, method)
                                        │
                              ┌─────────┴──────────┐
                              │ Content-Type?       │
                        image/*               application/json
                              │                     │
                         <img> inline        Syntax-highlighted
                           rendered           JSON pre block
```

**Notification persistence** uses `localStorage` with the key `api_term_notifs`. Read/unread state survives page refreshes.

---

## Supported HTTP Methods

| Method | Badge Color | Common Use |
|---|---|---|
| `GET` | 🟢 Green | Read / fetch data |
| `POST` | 🔵 Cyan | Create / submit data |
| `PUT` | 🟡 Amber | Full update / replace |
| `DELETE` | 🔴 Red | Delete a resource |
| `PATCH` | 🟣 Purple | Partial update |

---

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl + K` / `⌘ + K` | Open sidebar search |
| `Escape` | Close search or close modal |
| Click backdrop | Close modal |

---

## Dependencies

All dependencies are loaded from CDN — no local installation required.

| Resource | Purpose | Source |
|---|---|---|
| **IBM Plex Mono** | Body & code font | Google Fonts |
| **VT323** | Display / number font | Google Fonts |
| **Share Tech Mono** | Label / UI font | Google Fonts |
| **Font Awesome 6.5** | Copy & check icons | cdnjs.cloudflare.com |

No JavaScript frameworks. No CSS preprocessors. No build tools.

---

## Browser Support

| Browser | Support |
|---|---|
| Chrome / Edge 88+ | ✅ Full |
| Firefox 90+ | ✅ Full |
| Safari 14+ | ✅ Full |
| Mobile Chrome / Safari | ✅ Responsive layout |

Requires: `fetch`, `navigator.clipboard`, `URLSearchParams`, `localStorage` — all standard in modern browsers.

---

## Customization

### Change theme colors

All colors are CSS custom properties at the top of `index.html`:

```css
:root {
  --black:   #000000;   /* Background */
  --green:   #00ff41;   /* Primary accent */
  --amber:   #ffb300;   /* Version badge / secondary */
  --cyan:    #00e5ff;   /* POST method badge */
  --red:     #ff3131;   /* Error / DELETE badge */
}
```

### Disable CRT visual effects

Comment out or remove these three CSS blocks inside `<style>`:

```css
/* Remove scanlines */
body::before { ... }

/* Remove CRT vignette */
body::after { ... }

/* Remove noise overlay */
.noise { ... }
```

### Deploy to GitHub Pages

1. Push all files to a public repository
2. Go to **Settings → Pages → Source: `main` branch / root**
3. Visit `https://your-username.github.io/repo-name/`

---

## License

MIT — free to use, modify, and distribute.

---

*APIDOCS-RESTFUL — built with ♥*
