# IPLAB Documentation Repository

This repository contains the documentation for the IPLAB services and infrastructure.

## 📂 Organization

All documentation files, including Markdown pages and static assets, have been structured to live entirely within the `docs/` directory.

- `docs/`: The root of the documentation site (powered by Docsify).
- `docs/index.html`: The Docsify entry point.
- `docs/_sidebar.md`: Defines the navigation sidebar for the documentation.
- `docs/*.md`: The actual content files (e.g., `user-guide.md`, `admin-guide.md`, `migration.md`).

## ✏️ How to Edit

1. **Modify Existing Pages**: Simply edit the corresponding Markdown (`.md`) files within the `docs/` directory. 
2. **Add New Pages**: Create a new `.md` file inside the `docs/` directory.
3. **Update Navigation**: If you add a new page or change a title, make sure to update `docs/_sidebar.md` so the new content is discoverable through the documentation's navigation menu.
4. **Local Testing**: If you want to preview your changes locally before pushing, you can serve the `docs/` directory using any local web server (e.g., `python3 -m http.server -d docs`).

## 🚀 Deployment

The documentation site is **deployed automatically**. Once your changes are committed and pushed to the remote repository, they will be synchronized and published automatically without any additional manual steps required.
