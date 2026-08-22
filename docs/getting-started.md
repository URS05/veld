# Getting started

Get a live, interactive API documentation page running in under five minutes â€” no build step, no dependencies.

---

## Prerequisites

- A static file server, or Node.js â‰¥ 18 (for the CLI).
- Your `api.json` file. Not sure about the format? Start from [`examples/minimal/api.json`](../examples/minimal/api.json).

---

## Option A â€” CLI (recommended)

```bash
npx veld init my-docs
cd my-docs
```

This copies `src/` and a minimal `api.json` into `my-docs/`. Open `index.html` with any static server:

```bash
npx serve .
# or
python -m http.server
```

Then open [http://localhost:3000](http://localhost:3000) (or whichever port your server uses).

---

## Option B â€” Manual

1. Download or clone this repository.
2. Copy the contents of `src/` into your project folder.
3. Place your `api.json` in the same folder, next to `index.html`.
4. Serve the folder with any static server.

```
my-docs/
â”œâ”€â”€ index.html
â”œâ”€â”€ assets/
â”‚   â”œâ”€â”€ images/
â”‚   â”œâ”€â”€ scripts/
â”‚   â”‚   â”œâ”€â”€ pretty-json.js
â”‚   â”‚   â”œâ”€â”€ constants.js
â”‚   â”‚   â”œâ”€â”€ endpoints.js
â”‚   â”‚   â”œâ”€â”€ inputs.js
â”‚   â”‚   â””â”€â”€ script.js
â”‚   â””â”€â”€ styles/
â”‚       â””â”€â”€ styles.css
â””â”€â”€ api.json        â† your file
```

---

## Edit your `api.json`

Open `api.json` and describe your API. The minimal structure looks like this:

```json
{
  "settings": {
    "title": "My API",
    "version": "1.0.0",
    "baseUrl": "/api"
  },
  "tables": [
    {
      "name": { "singular": "user", "plural": "users" },
      "description": "Registered users.",
      "uri": "/users",
      "uriParameters": [
        { "name": "id", "type": "integer", "defaultValue": 1, "isPrimary": true }
      ],
      "schema": { "id": "integer", "name": "string", "email": "string" }
    }
  ]
}
```

Reload the page â€” the UI updates immediately. No rebuild needed.

---

## Validate before you publish

Run the built-in linter to catch schema errors before they reach the browser:

```bash
node bin/validate-api.mjs path/to/api.json
```

Exit code `0` means all clear. Exit code `1` means there are errors to fix.

See [`docs/validator.md`](./validator.md) for the full list of checks.

---

## Next steps

| Goal | Doc |
| --- | --- |
| Customize colors, logo, dark mode | [`docs/theming.md`](./theming.md) |
| Add non-CRUD endpoints | [`docs/custom-endpoints.md`](./custom-endpoints.md) |
| Deploy to GitHub Pages, Vercelâ€¦ | [`docs/deployment.md`](./deployment.md) |
| Full schema reference | [`docs/schema-reference.md`](./schema-reference.md) |

