<h1 align="center">X GitHub Toolkit</h1>

<div align="center">
  <img src="https://xscriptor.github.io/badges/social/github.svg" alt="GitHub" />
  <img src="https://xscriptor.github.io/badges/ci/github-actions.svg" alt="GitHub Actions" />
</div>

<p align="center"><em>
  A collection of reusable GitHub configurations, scripts, and workflows<br/>
  designed to streamline repository management.</em>
</p>

<details open>
  <summary><b>Table of Contents</b></summary>
  <ul>
    <li><a href="#whats-inside">What's Inside</a></li>
    <li><a href="#quick-start">Quick Start</a></li>
    <li><a href="#related-documents">Related Documents</a></li>
    <li><a href="#about-x">About X</a></li>
  </ul>
</details>

<hr>

<h2 id="whats-inside">What's Inside</h2>

<details open>
  <summary>Click to expand</summary>

| Module | Description |
|--------|-------------|
| [github-roadmap-sync](./github-roadmap-sync/) | Keep your `ROADMAP.md` checkboxes in sync with GitHub Issues automatically — auto-creates issues, manages labels, and closes tasks on push. |
| [project-issues-sync](./project-issues-sync/) | Automation to automatically synchronize the status of GitHub Issues (Opened, Closed, or Labeled) with the columns of a Kanban board. |
| [rust-releaser](./rust-releaser/) | Reusable GitHub Actions workflow that builds a Rust binary for every major platform (Ubuntu, Fedora, Arch Linux, Windows, macOS), signs every artifact with Sigstore Cosign (keyless), and publishes a GitHub Release — all triggered by a single git tag push. |
</details>


<h2 id="quick-start">Quick Start</h2>

<details>
  <summary>Click to expand</summary>

Each module is self-contained. Pick the one you need, copy its files into your repository, and follow the setup instructions in its own README. 

Example structure:
```
xgh/
└── .github/
    ├── scripts/       ← automation scripts
    └── workflows/     ← GitHub Actions triggers
```

No external dependencies beyond the GitHub CLI (`gh`) and Python.
</details>

<h2 id="related-documents">Related Documents</h2>

<details>
  <summary>Click to expand</summary>

For more information on how to participate, get help, or understand the repository policies, please refer to the following documents:

- [Contributing Guidelines](./CONTRIBUTING.md)
- [Code of Conduct](./CODE_OF_CONDUCT.md)
- [Security Policy](./SECURITY.md)
- [Support](./SUPPORT.md)
- [License](./LICENSE)

</details>


<div id="about-x" align="center">
<h2>X</h2>
<a href="https://dev.xscriptor.com">
  <img src="https://xscriptor.github.io/icons/icons/code/product-design/xsvg/verified-filled.svg" width="24" alt="X Web" />
</a>
 & 
<a href="https://github.com/xscriptor">
  <img src="https://xscriptor.github.io/icons/icons/code/product-design/xsvg/github.svg" width="24" alt="X Github Profile" />
</a>
 & 
<a href="https://www.xscriptor.com">
  <img src="https://xscriptor.github.io/icons/icons/code/product-design/xsvg/quotes.svg" width="24" alt="Xscriptor web" />
</a>
</div>