# Multilingual GitHub Pages Deployment

This document explains how the multilingual documentation is deployed to GitHub Pages.

## Architecture

The project uses a two-repository setup:

1. **Main Repository** (`narrowin/ot-labs-isc-cph-2025`)
   - Development and content creation happens here
   - Contains all lab materials and documentation source

2. **Docs Repository** (`narrowin/ot-labs-docs`)
   - Public repository dedicated to documentation
   - Hosts GitHub Pages deployment
   - Receives synced content from main repo

## Deployment Workflow

### 1. Local Development

Build both language versions locally:

```bash
./scripts/build-docs.sh
```

This creates:

```text
mkdocs/site/
├── index.html          # Language selection page
├── en/                 # English documentation
│   ├── index.html
│   ├── assets/
│   └── ...
└── de/                 # German documentation
    ├── index.html
    ├── assets/
    └── ...
```


### 2. Sync to Docs Repository

Run the sync script to push changes to the public docs repository:

```bash
./scripts/sync-docs.sh
```

This script:

- Copies `mkdocs/` directory to `../ot-labs-docs`
- Excludes virtual environments and caches
- Ensures deployment workflow exists
- Commits and pushes changes


### 3. Automatic Deployment

When changes are pushed to the docs repository, GitHub Actions automatically:

1. Checks out the repository
2. Installs Python dependencies
3. Builds English documentation: `mkdocs build -f mkdocs.en.yml`
4. Builds German documentation: `mkdocs build -f mkdocs.de.yml`
5. Creates root `index.html` with language selection
6. Deploys entire `site/` directory to GitHub Pages

## URLs

After deployment, documentation is available at:

- **Root**: `https://narrowin.github.io/ot-labs-docs/` (language selection)
- **English**: `https://narrowin.github.io/ot-labs-docs/en/`
- **German**: `https://narrowin.github.io/ot-labs-docs/de/`

## Language Switcher

Each page includes a language switcher in the header that allows users to switch between English and German versions. The switcher uses absolute paths that work both locally and on GitHub Pages:

```yaml
extra:
  alternate:
    - name: English
      link: /ot-labs-docs/en/
      lang: en
    - name: Deutsch
      link: /ot-labs-docs/de/
      lang: de
```

## Configuration Files

### mkdocs.en.yml

- Builds English documentation to `site/en/`
- Sets `site_url` for proper canonical URLs
- Configures language switcher paths

### mkdocs.de.yml

- Builds German documentation to `site/de/`
- Sets `site_url` for proper canonical URLs
- Configures language switcher paths

### .github-docs-repo-workflow-deploy.yaml

- GitHub Actions workflow template
- Copied to docs repository by sync script
- Handles multilingual build and deployment


## Development Workflow

1. **Edit content** in main repository
2. **Test locally** with `./scripts/build-docs.sh` and serve on localhost
3. **Sync to docs repo** with `./scripts/sync-docs.sh`
4. **Automatic deployment** triggers on push to docs repo
5. **Verify deployment** at GitHub Pages URL

## Key Features

- **Static site**: No server-side processing required
- **SEO friendly**: Each language has proper canonical URLs
- **Fast switching**: Language switcher uses client-side navigation
- **Clean URLs**: `/en/` and `/de/` paths are intuitive
- **Offline support**: Both versions include offline plugin
- **Automatic sync**: Single command deploys both languages

