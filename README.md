# 189th Technical Docs

Internal technical documentation for the 189th clan: ClanGuard Bot, custom Battlefield 6 Portal game modes, and supporting tooling.

Published at: <https://dklaver15.github.io/the189th-docs/>

## Local development

```bash
python -m venv .venv
source .venv/bin/activate         # on macOS/Linux
# .\.venv\Scripts\activate         # on Windows
pip install -r requirements.txt

mkdocs serve
```

Open <http://127.0.0.1:8000>. The site rebuilds on save.

## Deployment

Pushing to `main` triggers `.github/workflows/deploy-docs.yml`, which builds the site with `mkdocs build --strict` and publishes to GitHub Pages.

To enable Pages on first setup:

1. Repository → **Settings** → **Pages**
2. **Source**: GitHub Actions
3. Push to `main` and watch the **Actions** tab

## Structure

```
docs/
├── index.md                        # Landing
├── clanguard-bot/                  # Discord bot docs
├── portal-modes/                   # Vendetta, CTF, etc.
└── portal-scripting/               # Cross-cutting Portal scripting reference
```

Edit any page and push — the site rebuilds in ~1 minute.

## Adding a page

1. Create `docs/section/new-page.md`
2. Add it to the `nav:` block in `mkdocs.yml`
3. Push

## Conventions

- Use Material admonitions for callouts: `!!! warning`, `!!! note`, `!!! danger`.
- Use Mermaid for state machines and flow diagrams.
- Code blocks should always specify a language for syntax highlighting.
- Cross-link liberally — `[deployment](../clanguard-bot/deployment.md)`.
