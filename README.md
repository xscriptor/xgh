<h1 align="center">X GitHub Toolkit</h1>

<p align="center">
  <img src="https://xscriptor.github.io/badges/social/github.svg" alt="GitHub" />
  <img src="https://xscriptor.github.io/badges/ci/github-actions.svg" alt="GitHub Actions" />
</p>

<p align="center">
  A collection of reusable GitHub configurations, scripts, and workflows<br/>
  designed to streamline repository management.
</p>

---

> **Status:** Under active development

## What's Inside

| Module | Description |
|--------|-------------|
| [github-roadmap-sync](./github-roadmap-sync/) | Keep your `ROADMAP.md` checkboxes in sync with GitHub Issues automatically — auto-creates issues, manages labels, and closes tasks on push. |

## Quick Start

Each module is self-contained. Pick the one you need, copy its files into your repository, and follow the setup instructions in its own README:

```
xgh/
└── .github/
    ├── scripts/       ← automation scripts
    └── workflows/     ← GitHub Actions triggers
```

No external dependencies beyond the GitHub CLI (`gh`) and Python.

## Contributing

1. Fork the repository.
2. Create a feature branch (`git checkout -b feat/my-feature`).
3. Commit your changes and push.
4. Open a Pull Request.

## License

This project is licensed under the [MIT License](./LICENSE).
