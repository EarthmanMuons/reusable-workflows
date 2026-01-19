# Zig Workflows

Reusable workflows for building and deploying [Zig](https://ziglang.org/)
documentation to GitHub Pages.

## Usage Example

```yml
jobs:
  zig_docs:
    uses: EarthmanMuons/reusable-workflows/.github/workflows/zig/deploy-github-pages-zig.yml@main
```

---

## deploy-github-pages-zig.yml

Builds Zig documentation using `zig test -femit-docs` and deploys the generated
docs to GitHub Pages.

**Required permissions**

- `pages: write`
- `id-token: write`
