# Veld

**Static, dependency-free API documentation driven by a single JSON file.**

[![npm](https://img.shields.io/npm/v/Veld-docs)](https://www.npmjs.com/package/Veld-docs)
[![license](https://img.shields.io/github/license/Lauwed/Veld-docs)](./LICENSE)
[![CI](https://github.com/Lauwed/Veld-docs/actions/workflows/ci.yml/badge.svg)](https://github.com/Lauwed/Veld-docs/actions)

[Live demo](https://lauwed.github.io/Veld-docs/)

<img width="1710" height="1004" alt="Veld preview" src="https://github.com/user-attachments/assets/583c95d1-bc84-4363-b95c-2e36298e8bbe" />

> [!NOTE]
> Most of the non-source files in this repository (documentation, CI workflows, CLI scripts, and configuration) were largely generated with the help of [Claude](https://claude.ai). The source files (`src/`) were written by hand and reviewed carefully. If you spot anything insecure or incorrect anywhere in the repo, please feel free to [open an issue](https://github.com/Lauwed/Veld-docs/issues) or submit a PR.

## What is Veld?

If you have ever built a REST API and then had to write documentation for it separately, you know how painful that is. You change an endpoint, forget to update the docs, and suddenly your teammates are confused. Veld solves this by making `api.json` your single source of truth. You describe your API once in that file, and Veld generates a fully interactive documentation page from it automatically, with no build tools, no framework, and no server-side code required.

The whole thing runs as pure HTML, CSS, and vanilla JavaScript. You can host it on GitHub Pages, Netlify, Vercel, or literally any static file server.

## Why This Approach Makes Sense

Most documentation tools like Swagger UI require you to write a full OpenAPI specification, which has a steep learning curve. Veld's `api.json` format is intentionally simpler. If you know what a REST resource looks like, you can describe it in Veld in under a minute.

The trade-off is that Veld is less exhaustive than OpenAPI. It does not support every possible HTTP pattern. But for standard CRUD APIs, it covers everything you need and gets out of your way.

## Architecture

Here is a high-level view of how the system is structured:

```
api.json  (your data source)
    |
    v
index.html  (static HTML shell, no framework)
    |
    v
script.js  (fetches api.json on page load, orchestrates everything)
    |
    +-- endpoints.js     generates CRUD endpoint panels from table definitions
    +-- inputs.js        renders form inputs (text, number, select, file upload)
    +-- pretty-json.js   syntax highlights JSON in responses
    +-- constants.js     maps HTTP status codes and input type labels
```

The CLI lives separately in `bin/Veld.mjs` and handles four tasks:

```
bin/Veld.mjs
    |
    +-- init       scaffolds a new docs folder from a template
    +-- serve      starts a local HTTP server (pure node:http, no Express)
    +-- validate   runs the schema linter in bin/validate-api.mjs
    +-- sync       updates src/ files to the latest installed version
```

Nothing in the browser talks to a backend at runtime. The page fetches `api.json` once on load, builds the entire UI in memory, and that is it.

## Quickstart

```bash
# Scaffold a new docs folder
npx Veld-docs init my-docs

# Edit your API schema
cd my-docs && $EDITOR api.json

# Preview locally
npx Veld-docs serve .
```

Open [http://localhost:3000](http://localhost:3000). That is all you need to get started.

## Manual Setup

If you prefer not to use the CLI, you can copy the `src/` folder directly into your project. Drop your `api.json` next to `index.html` and serve it with any static file server.

```
my-docs/
  index.html
  api.json          your file
  assets/
    images/
    scripts/
      pretty-json.js
      constants.js
      endpoints.js
      inputs.js
      script.js
    styles/
      styles.css
```

## The api.json Format

The entire documentation is driven by this one file. It has two main sections: `settings` and `tables`.

### settings

Controls global things like the page title, version badge, base URL, logo, and theme colors.

```json
{
  "settings": {
    "title": "My API",
    "version": "1.0.0",
    "baseUrl": "/api",
    "logo": "./assets/images/logo.svg",
    "favicon": "./assets/images/favicon.svg",
    "theme": {
      "accent": "#1d9e75",
      "colorGet": "#1a5fa8",
      "colorPost": "#6b3a9e",
      "colorPut": "#b84a0e",
      "colorDelete": "#a8192e",
      "fontBody": "\"Inter\", sans-serif"
    },
    "darkTheme": {
      "accent": "#2ec48f"
    }
  }
}
```

The `theme` and `darkTheme` blocks are optional. If you provide them, Veld injects them as CSS custom properties at runtime so you do not have to touch any CSS.

### tables

Each entry in `tables` represents a REST resource (think: a database table or a model). Veld automatically generates the standard five endpoints for it: GET list, GET single, POST, PUT, and DELETE.

```json
{
  "tables": [
    {
      "name": { "singular": "item", "plural": "items" },
      "description": "A simple item in the system.",
      "uri": "/items",
      "uriParameters": [
        { "name": "id", "type": "integer", "defaultValue": 1, "isPrimary": true }
      ],
      "schema": {
        "id": "integer",
        "name": "string"
      }
    }
  ]
}
```

You can also restrict which endpoints are generated using `includedEndpoints`, add query parameters for list endpoints via `listParameters`, define expected request bodies, and write out example responses for each status code. The full field reference is in [`docs/schema-reference.md`](docs/schema-reference.md).

## How It Works at Runtime

When the browser opens `index.html`, `script.js` fetches `api.json` immediately. It then:

1. Reads `settings` and applies the title, version badge, logo, favicon, and theme CSS variables to the document.
2. Reads `tables` and calls `generateTablesEndpoints()` from `endpoints.js`, which converts each table definition into a list of endpoint objects with method, URI, parameters, body schema, and response examples.
3. Renders each endpoint as a collapsible `<details>` panel in the DOM. Inside each panel there is a live form where you can fill in URI parameters, query parameters, or a request body and hit Fetch to actually call your API.
4. Handles auth via a Bearer token stored in `sessionStorage`. You enter it once in the auth section and it gets attached to every request that has `tokenRequired: true`.

The fetch response is displayed inline using `pretty-json.js`, which adds syntax highlighting to the raw JSON.

## CLI Reference

| Command | What it does |
| --- | --- |
| `Veld-docs init [dir] [--template <name>]` | Scaffolds a new docs folder. If you omit arguments, it runs an interactive wizard. |
| `Veld-docs serve [dir] [--port <number>]` | Starts a local static file server. Default port is 3000. |
| `Veld-docs validate [path]` | Runs the linter on `api.json`. Exit code 0 means no errors, exit code 1 means something needs fixing. |
| `Veld-docs sync [dir]` | Overwrites only the Veld source files in an existing docs folder. Your `api.json` and custom assets are untouched. |

The CLI also checks for newer versions on npm in the background every time you run it. If an update is available, it prints a notice after the command finishes. This check has a 3-second timeout and fails silently so it never blocks your workflow.

## Templates

When you run `Veld-docs init`, you can pick from three starting templates:

| Template | Description |
| --- | --- |
| `minimal` | Two tables and a basic schema. Good starting point if you want to build from scratch. |
| `blog` | A realistic CRUD API with posts, users, tags, pagination, and authentication endpoints. |
| `ecommerce` | An advanced example with multipart file uploads, order state machines, and full auth flows. |

These live in the `examples/` folder. Reading the `ecommerce` example is honestly one of the best ways to understand everything Veld can do.

## Validating Your Schema

Before you deploy, run:

```bash
npx Veld-docs validate path/to/api.json
```

The validator checks for things like missing required fields, invalid types, duplicate endpoint definitions, and malformed URI parameters. It exits with code 0 if everything is clean and code 1 if there are errors to fix. You can also integrate it into a GitHub Actions workflow so it runs automatically on every push. See [`docs/validator.md`](docs/validator.md) for the full list of checks and a sample workflow config.

## Theming

Colors, fonts, and dark mode are all controlled from the `settings.theme` and `settings.darkTheme` blocks in `api.json`. No CSS editing required. The available properties are:

| Key | CSS variable it maps to |
| --- | --- |
| `accent` | `--accent` |
| `accentLight` | `--accent-light` |
| `accentDark` | `--accent-dark` |
| `colorGet` | `--blue` (GET method color) |
| `colorPost` | `--purple` (POST method color) |
| `colorPut` | `--orange` (PUT method color) |
| `colorDelete` | `--red` (DELETE method color) |
| `fontBody` | `--font-body` |
| `fontMono` | `--font-mono` |

See [`docs/theming.md`](docs/theming.md) for a complete walkthrough.

## Custom Endpoints

Not every API follows standard CRUD patterns. Veld supports custom endpoints for cases like login, password reset, file upload callbacks, or anything else that does not map cleanly to a resource. You define them under a `customEndpoints` array inside a table entry. See [`docs/custom-endpoints.md`](docs/custom-endpoints.md) for examples.

## Deployment

Since the output is a static folder with no server-side dependencies, you can deploy it anywhere:

- **GitHub Pages**: Push the docs folder to a `gh-pages` branch or configure it in repo settings.
- **Vercel / Netlify**: Drag and drop the folder or connect your repo.
- **Cloudflare Pages**: Same idea, works out of the box.

See [`docs/deployment.md`](docs/deployment.md) for step-by-step instructions for each platform.

## Documentation Index

| File | What it covers |
| --- | --- |
| [`docs/getting-started.md`](docs/getting-started.md) | CLI and manual setup walkthrough |
| [`docs/schema-reference.md`](docs/schema-reference.md) | Complete `api.json` field reference |
| [`docs/theming.md`](docs/theming.md) | Colors, fonts, logo, dark mode |
| [`docs/custom-endpoints.md`](docs/custom-endpoints.md) | Non-CRUD endpoints and file uploads |
| [`docs/validator.md`](docs/validator.md) | Linter usage and GitHub Actions integration |
| [`docs/deployment.md`](docs/deployment.md) | GitHub Pages, Vercel, Netlify, Cloudflare Pages |

## Project Structure Reference

```
Veld-docs/
  bin/
    Veld.mjs          CLI entry point (init, serve, validate, sync commands)
    validate-api.mjs    Schema linter
  src/
    index.html          Static HTML shell
    assets/
      scripts/
        script.js       Main orchestrator. Fetches api.json and wires up the UI.
        endpoints.js    Converts table definitions into endpoint panel objects
        inputs.js       Renders form controls for parameters and request bodies
        pretty-json.js  Syntax highlights JSON output
        constants.js    HTTP status code labels and input type mappings
      styles/
        styles.css      All UI styles
      images/
  examples/
    minimal/            Minimal two-table example
    blog/               Realistic CRUD blog API
    ecommerce/          Advanced example with uploads and auth
  docs/                 Markdown documentation files
  scripts/
    pre-commit          Git hook (runs formatter before commits)
    pre-push            Git hook (runs tests before push)
```

## Contributing

Contributions are welcome. Read [`CONTRIBUTING.md`](CONTRIBUTING.md) for the setup instructions, commit conventions, and pull request process.

## License

[MIT](./LICENSE) (c) Lauwed

