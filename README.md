# Github Actions

This GitHub Actions workflow file automates the building and deployment of a Hugo site to GitHub Pages. Here’s a breakdown of each section:

### Workflow Metadata
```yaml
name: Deploy Hugo site to Pages
on:
  push:
    branches:
      - main
  workflow_dispatch:
```
- **`name`**: The workflow’s name, visible in the Actions tab.
- **`on`**: Specifies triggers:
  - **`push`**: Runs when changes are pushed to the `main` branch.
  - **`workflow_dispatch`**: Allows manually running this workflow from the Actions tab.

### Permissions
```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```
- **`permissions`**: Grants the `GITHUB_TOKEN` access to manage GitHub Pages (`pages: write`) and read repository contents (`contents: read`).

### Concurrency
```yaml
concurrency:
  group: "pages"
  cancel-in-progress: false
```
- **`concurrency`**: Ensures only one deployment can run at a time, grouping by `"pages"` and not canceling in-progress runs.

### Default Shell
```yaml
defaults:
  run:
    shell: bash
```
- **`defaults`**: Sets the default shell for running commands to `bash`.

### Jobs

#### 1. Build Job
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.134.2
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
```
- **`runs-on: ubuntu-latest`**: Specifies the environment to run the job.
- **`env`**: Sets environment variables like `HUGO_VERSION` for the Hugo version to install.
- **`steps`**:
  - **Install Hugo CLI**: Downloads and installs Hugo Extended for building the site.

```yaml
      - name: Install Dart Sass
        run: sudo snap install dart-sass
```
- Installs Dart Sass, a CSS preprocessor, if the Hugo site uses SCSS/Sass.

```yaml
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
```
- **Checkout**: Clones the repository with submodules.

```yaml
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
```
- **Setup Pages**: Configures GitHub Pages settings for deployment.

```yaml
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
```
- **Install Node.js dependencies**: Runs `npm ci` if `package-lock.json` or `npm-shrinkwrap.json` exists (common for JS-based Hugo themes).

```yaml
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
```
- **Build with Hugo**: Builds the site using:
  - `--gc`: Garbage collects unused files.
  - `--minify`: Minimizes output.
  - `--baseURL`: Sets the base URL for GitHub Pages.

```yaml
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
```
- **Upload artifact**: Uploads the generated files in `./public` for the next deployment step.

#### 2. Deploy Job
```yaml
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
- **environment**: Defines the deployment environment as `github-pages`.
- **`needs: build`**: Ensures this job runs after the `build` job completes.
- **Deploy to GitHub Pages**: Deploys the uploaded artifact to GitHub Pages, making the site available publicly.
