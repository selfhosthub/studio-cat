# Studio Catalog (Public)

Public catalog repository for SelfHostHub Studio marketplace.

## Structure

```
studio-cat/
├── marketplace-catalog.json   # Package catalog (lists ALL packages)
├── templates-catalog.json     # Template catalog (lists ALL templates)
├── packages/                  # Basic tier packages only
│   └── core-1.0.0.zip
└── templates/                 # Basic tier templates only
    ├── welcome-webhook.json
    └── httpbin-echo-flow.json
```

## Catalogs

The catalog files list **all** packages and templates (both basic and advanced tiers).

- **Basic tier** content is hosted in this public repo
- **Advanced tier** content URLs point to the private `studio-mark` repo

## Environment Variables

Configure Studio to use this catalog:

```
MARKETPLACE_CATALOG_URL=https://raw.githubusercontent.com/kickin6/studio-cat/main/marketplace-catalog.json
TEMPLATES_CATALOG_URL=https://raw.githubusercontent.com/kickin6/studio-cat/main/templates-catalog.json
```

## Access

- Basic packages/templates: No authentication required
- Advanced packages/templates: Requires `MARKETPLACE_TOKEN` (GitHub PAT with repo access to studio-mark)
# studio-cat
