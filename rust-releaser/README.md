<h1>rust-releaser</h1>

<p>
  A reusable GitHub Actions workflow that builds a Rust binary for every major platform
  (Ubuntu, Fedora, Arch Linux, Windows, macOS), signs every artifact with
  <a href="https://docs.sigstore.dev/cosign/overview/">Sigstore Cosign</a> (keyless),
  and publishes a GitHub Release — all triggered by a single git tag push.
</p>

<hr />

<h2>Table of Contents</h2>

<ol>
  <li><a href="#directory-layout">Directory Layout</a></li>
  <li><a href="#how-it-works">How It Works</a></li>
  <li><a href="#customization">Customization — What to Change for Your Application</a></li>
  <li><a href="#cargo-metadata">Required Cargo.toml Metadata</a></li>
  <li><a href="#secrets">Repository Secrets — How to Configure and Add Them</a></li>
  <li><a href="#triggering">Triggering a Release — Versioning Commands</a></li>
  <li><a href="#signing">Signing vs. Unsigned Publishing</a></li>
  <li><a href="#security-workflow">The Security Workflow</a></li>
  <li><a href="#troubleshooting">Troubleshooting</a></li>
  <li><a href="#about-x">About X</a></li>
</ol>

<hr />

<h2 id="directory-layout">1. Directory Layout</h2>

<pre>
rust-releaser/
├── .github/
│   └── workflows/
│       ├── release.yml    &lt;-- Main cross-platform release pipeline
│       └── security.yml   &lt;-- Format, lint, test, and audit checks
└── README.md              &lt;-- This document
</pre>

<p>
  Copy the <code>.github/</code> folder into the root of your own Rust project repository.
  No other files are required.
</p>

<hr />

<h2 id="how-it-works">2. How It Works</h2>

<p>The <code>release.yml</code> workflow runs the following jobs in parallel:</p>

<table>
  <thead>
    <tr>
      <th>Job</th>
      <th>Runner</th>
      <th>Output Format</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>build-linux-ubuntu</td>
      <td>ubuntu-latest</td>
      <td>.deb (via cargo-deb)</td>
    </tr>
    <tr>
      <td>build-linux-fedora</td>
      <td>ubuntu-latest + fedora:latest container</td>
      <td>.rpm (via cargo-generate-rpm)</td>
    </tr>
    <tr>
      <td>build-linux-arch</td>
      <td>ubuntu-latest + archlinux:latest container</td>
      <td>.pkg.tar.zst (manual bsdtar)</td>
    </tr>
    <tr>
      <td>build-windows</td>
      <td>windows-latest</td>
      <td>.exe (raw binary copy)</td>
    </tr>
    <tr>
      <td>build-macos</td>
      <td>macos-latest</td>
      <td>.dmg (via hdiutil)</td>
    </tr>
  </tbody>
</table>

<p>
  After all build jobs succeed, the <code>publish-release</code> job collects all artifacts,
  signs each one with Cosign (keyless — no private key needed), and uploads them to a
  GitHub Release. If a GitHub App token is configured it is used preferentially; otherwise
  the standard workflow <code>GITHUB_TOKEN</code> is used as a fallback.
</p>

<hr />

<h2 id="customization">3. Customization — What to Change for Your Application</h2>

<p>
  All application-specific values are controlled by a single environment variable defined
  at the top of <code>release.yml</code>:
</p>

<pre><code>env:
  CARGO_TERM_COLOR: always
  APP_NAME: "my-app"            # &lt;-- Change this to your binary name
  RELEASE_TAG: ...
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: "true"
</code></pre>

<h3>3.1 APP_NAME</h3>

<p>
  Set <code>APP_NAME</code> to the exact binary name produced by <code>cargo build --release</code>.
  This is the value of <code>name</code> under <code>[[bin]]</code> in your
  <code>Cargo.toml</code>, or the <code>[package] name</code> if you have no explicit
  <code>[[bin]]</code> section. For example:
</p>

<pre><code># Cargo.toml
[package]
name = "my-app"
</code></pre>

<pre><code># release.yml
env:
  APP_NAME: "my-app"
</code></pre>

<p>
  If your package name contains hyphens (<code>my-app</code>), Cargo keeps them as-is in
  the binary name. If it contains underscores (<code>my_app</code>), Cargo also keeps them.
  Use the exact name that appears in <code>target/release/</code> after a local build.
</p>

<h3>3.2 Removing Platforms You Do Not Need</h3>

<p>
  Delete the corresponding job block from <code>release.yml</code> and remove the job name
  from the <code>needs:</code> list in the <code>publish-release</code> job. For example,
  to remove the Fedora build:
</p>

<pre><code>  # Delete the entire build-linux-fedora job block, then update:
  publish-release:
    needs:
      - build-linux-ubuntu
      # - build-linux-fedora   &lt;-- remove this line
      - build-linux-arch
      - build-windows
      - build-macos
</code></pre>

<h3>3.3 Adding Extra Build Features or Targets</h3>

<p>
  Each build job runs <code>cargo build --release --locked</code>. If your application
  requires specific Cargo features, replace the build step with:
</p>

<pre><code>      - name: Build
        run: cargo build --release --locked --features "feature-a,feature-b"
</code></pre>

<p>
  For cross-compilation to a different architecture (for example, ARM64 on the Ubuntu runner),
  add the Rust target and use <code>cross</code> or a custom toolchain step before the build.
</p>

<h3>3.4 Changing the Tag Pattern</h3>

<p>
  The workflow triggers on any tag matching <code>v*</code>. To use a different pattern
  (for example, only tags like <code>release-1.0.0</code>), change the trigger section:
</p>

<pre><code>on:
  push:
    tags:
      - "release-*"
</code></pre>

<p>
  Also update the <code>RELEASE_TAG</code> stripping logic in any packaging step
  that strips the leading <code>v</code> prefix with <code>${RELEASE_TAG#v}</code>,
  since a different prefix would need a different expression.
</p>

<h3>3.5 Adjusting the macOS DMG</h3>

<p>
  The macOS build copies the raw binary into a DMG. If you have an <code>.app</code> bundle
  or additional resources, replace the <code>Package (.dmg)</code> step body with your own
  staging logic before the <code>hdiutil create</code> call.
</p>

<h3>3.6 Adjusting the Windows Package</h3>

<p>
  The Windows build copies the raw <code>.exe</code>. If you need an installer (for example,
  built with NSIS or WiX), add those steps after the <code>cargo build</code> step and adjust
  the path in the <code>Copy-Item</code> command to point to the generated installer file.
</p>

<hr />

<h2 id="cargo-metadata">4. Required Cargo.toml Metadata</h2>

<p>
  The packaging tools (<code>cargo-deb</code> and <code>cargo-generate-rpm</code>) read
  metadata directly from <code>Cargo.toml</code>. At minimum you need:
</p>

<pre><code>[package]
name        = "my-app"
version     = "0.1.0"
description = "Short description of your application"
license     = "MIT"
homepage    = "https://github.com/your-org/your-repo"
repository  = "https://github.com/your-org/your-repo"
authors     = ["Your Name &lt;you@example.com&gt;"]
</code></pre>

<p>For the .deb package, add a <code>[package.metadata.deb]</code> section:</p>

<pre><code>[package.metadata.deb]
maintainer         = "Your Name &lt;you@example.com&gt;"
copyright          = "2025 Your Name"
license-file       = ["LICENSE", "0"]
extended-description = "Longer description for the package manager."
depends            = "$auto"
section            = "utility"
priority           = "optional"
assets = [
  ["target/release/my-app", "usr/bin/", "755"],
]
</code></pre>

<p>For the .rpm package, add a <code>[package.metadata.generate-rpm]</code> section:</p>

<pre><code>[package.metadata.generate-rpm]
assets = [
  { source = "target/release/my-app", dest = "/usr/bin/my-app", mode = "755" },
]
</code></pre>

<hr />

<h2 id="secrets">5. Repository Secrets — How to Configure and Add Them</h2>

<h3>5.1 Overview of Required Secrets</h3>

<table>
  <thead>
    <tr>
      <th>Secret Name</th>
      <th>Required</th>
      <th>Purpose</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>RELEASE_GH_APP_ID</td>
      <td>Optional (recommended)</td>
      <td>GitHub App ID used to generate a scoped token for publishing releases</td>
    </tr>
    <tr>
      <td>RELEASE_GH_APP_PRIVATE_KEY</td>
      <td>Optional (recommended)</td>
      <td>PEM private key of the GitHub App above</td>
    </tr>
  </tbody>
</table>

<p>
  If neither secret is configured the workflow falls back to the built-in
  <code>GITHUB_TOKEN</code>, which is always available automatically. The only limitation
  of the fallback is that releases created with <code>GITHUB_TOKEN</code> cannot trigger
  other workflows downstream.
</p>

<h3>5.2 How to Add Secrets to a Repository</h3>

<ol>
  <li>Go to your repository on GitHub.</li>
  <li>Click <strong>Settings</strong> in the top navigation bar.</li>
  <li>In the left sidebar click <strong>Secrets and variables</strong>, then <strong>Actions</strong>.</li>
  <li>Click <strong>New repository secret</strong>.</li>
  <li>Enter the secret name exactly as shown in the table above.</li>
  <li>Paste the secret value and click <strong>Add secret</strong>.</li>
</ol>

<h3>5.3 Creating a GitHub App for Release Publishing (Optional but Recommended)</h3>

<p>Using a GitHub App gives the workflow a token with fine-grained permissions limited to
releasing on this specific repository, without sharing any personal access token.</p>

<ol>
  <li>
    Go to <strong>Settings &gt; Developer settings &gt; GitHub Apps</strong> and click
    <strong>New GitHub App</strong>.
  </li>
  <li>
    Give it a name (for example <code>my-app-releaser</code>), set the Homepage URL to your
    repository, and disable the Webhook option.
  </li>
  <li>
    Under <strong>Repository permissions</strong> grant:
    <ul>
      <li>Contents: <strong>Read and write</strong></li>
    </ul>
    All other permissions should remain as <strong>No access</strong>.
  </li>
  <li>Click <strong>Create GitHub App</strong>.</li>
  <li>
    On the App settings page note the <strong>App ID</strong> — this is the value for
    <code>RELEASE_GH_APP_ID</code>.
  </li>
  <li>
    Scroll to <strong>Private keys</strong> and click <strong>Generate a private key</strong>.
    A <code>.pem</code> file is downloaded. Open it and copy the entire contents — this is
    the value for <code>RELEASE_GH_APP_PRIVATE_KEY</code>.
  </li>
  <li>
    Go to the App's <strong>Install App</strong> tab, click <strong>Install</strong>, and
    choose <strong>Only select repositories</strong>, selecting your target repository.
  </li>
</ol>

<hr />

<h2 id="triggering">6. Triggering a Release — Versioning Commands</h2>

<p>
  The workflow starts automatically when a tag matching <code>v*</code> is pushed. Use
  semantic versioning (<code>vMAJOR.MINOR.PATCH</code>). Below are the exact commands
  to create and push a tag:
</p>

<h3>6.1 Standard Tag Push (Most Common)</h3>

<pre><code># Make sure your local main branch is up to date
git checkout main
git pull origin main

# Create an annotated tag (recommended: annotated tags carry a message)
git tag -a v1.0.0 -m "Release v1.0.0"

# Push the tag to GitHub — this starts the workflow
git push origin v1.0.0
</code></pre>

<h3>6.2 Lightweight Tag (No Message)</h3>

<pre><code>git tag v1.0.0
git push origin v1.0.0
</code></pre>

<h3>6.3 Push a Tag on a Specific Commit</h3>

<pre><code># Tag a specific commit hash instead of HEAD
git tag -a v1.0.0 &lt;commit-sha&gt; -m "Release v1.0.0"
git push origin v1.0.0
</code></pre>

<h3>6.4 Delete and Re-push a Tag (Force a Re-run)</h3>

<pre><code># Delete tag locally and remotely, then re-create it
git tag -d v1.0.0
git push origin --delete v1.0.0

git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
</code></pre>

<h3>6.5 Manual Trigger via GitHub UI (workflow_dispatch)</h3>

<p>
  You can also run the workflow manually from GitHub Actions without pushing a new tag.
  This is useful to re-publish an existing release:
</p>

<ol>
  <li>Go to your repository on GitHub.</li>
  <li>Click the <strong>Actions</strong> tab.</li>
  <li>Select the <strong>Release</strong> workflow in the left sidebar.</li>
  <li>Click <strong>Run workflow</strong>.</li>
  <li>Enter the existing tag (for example <code>v1.0.0</code>) in the input field.</li>
  <li>Click <strong>Run workflow</strong> to start it.</li>
</ol>

<hr />

<h2 id="signing">7. Signing vs. Unsigned Publishing</h2>

<h3>7.1 Keyless Signing with Cosign (Default)</h3>

<p>
  The workflow uses <a href="https://docs.sigstore.dev/cosign/overview/">Sigstore Cosign</a>
  in keyless mode. No private key is stored anywhere. Instead, Cosign uses the GitHub
  OIDC token (<code>id-token: write</code> permission) to obtain a short-lived certificate
  from Sigstore's Fulcio CA. The certificate and signature are recorded in Sigstore's
  immutable Rekor transparency log.
</p>

<p>For each artifact the workflow produces two additional files:</p>

<ul>
  <li><code>artifact.sig</code> — the detached signature</li>
  <li><code>artifact.pem</code> — the short-lived signing certificate</li>
</ul>

<p>Users can verify a downloaded artifact with:</p>

<pre><code>cosign verify-blob \
  --signature my-app-v1.0.0-linux-ubuntu-amd64.deb.sig \
  --certificate my-app-v1.0.0-linux-ubuntu-amd64.deb.pem \
  --certificate-identity-regexp "https://github.com/your-org/your-repo" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  my-app-v1.0.0-linux-ubuntu-amd64.deb
</code></pre>

<h3>7.2 Disabling Signing (Unsigned Publishing)</h3>

<p>
  If you do not want to sign artifacts, remove the two signing steps from the
  <code>publish-release</code> job and remove the <code>id-token: write</code> permission:
</p>

<pre><code>    permissions:
      contents: write
      # id-token: write   &lt;-- remove this line

    steps:
      # ... other steps ...

      # Delete or comment out these two steps:
      # - name: Install cosign
      #   uses: sigstore/cosign-installer@v3
      # - name: Sign assets (keyless)
      #   run: |
      #     for file in dist/*; do ...
</code></pre>

<h3>7.3 Signing with a Custom Key Pair (Advanced)</h3>

<p>
  If you prefer to manage your own key pair instead of using keyless signing:
</p>

<ol>
  <li>Generate a key pair: <code>cosign generate-key-pair</code></li>
  <li>
    Store the private key passphrase as a secret named <code>COSIGN_PASSWORD</code> and the
    private key contents as <code>COSIGN_PRIVATE_KEY</code>.
  </li>
  <li>Replace the keyless signing step with:</li>
</ol>

<pre><code>      - name: Sign assets (key pair)
        env:
          COSIGN_PASSWORD: ${{ secrets.COSIGN_PASSWORD }}
          COSIGN_PRIVATE_KEY: ${{ secrets.COSIGN_PRIVATE_KEY }}
        run: |
          echo "${COSIGN_PRIVATE_KEY}" > cosign.key
          for file in dist/*; do
            cosign sign-blob --key cosign.key "$file" \
              --output-signature "${file}.sig"
          done
          rm -f cosign.key
</code></pre>

<hr />

<h2 id="security-workflow">8. The Security Workflow</h2>

<p>
  The <code>security.yml</code> workflow runs on every push and pull request to
  <code>main</code>. It performs the following checks:
</p>

<table>
  <thead>
    <tr>
      <th>Step</th>
      <th>Command</th>
      <th>Purpose</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Format check</td>
      <td>cargo fmt --all -- --check</td>
      <td>Ensures code is formatted with rustfmt</td>
    </tr>
    <tr>
      <td>Lints</td>
      <td>cargo clippy --all-targets --all-features -- -D warnings</td>
      <td>Fails on any Clippy warning</td>
    </tr>
    <tr>
      <td>Tests</td>
      <td>cargo test --all-targets --all-features</td>
      <td>Runs the full test suite</td>
    </tr>
    <tr>
      <td>Vulnerability audit</td>
      <td>cargo audit</td>
      <td>Checks dependencies against the RustSec advisory database</td>
    </tr>
  </tbody>
</table>

<h3>8.1 Ignoring a Specific Advisory</h3>

<p>
  The default configuration ignores <code>RUSTSEC-2023-0071</code> (a known advisory in
  the <code>rsa</code> crate that many projects carry transitively). To ignore a different
  advisory, or to stop ignoring this one, edit the audit step in <code>security.yml</code>:
</p>

<pre><code>      - name: Dependency vulnerability audit
        run: cargo audit
        # To ignore a specific advisory:
        # run: cargo audit --ignore RUSTSEC-YYYY-NNNN
</code></pre>

<hr />

<h2 id="troubleshooting">9. Troubleshooting</h2>

<table>
  <thead>
    <tr>
      <th>Symptom</th>
      <th>Likely Cause</th>
      <th>Fix</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Build fails: binary not found in target/debian/</td>
      <td>APP_NAME does not match the package name in Cargo.toml</td>
      <td>Set APP_NAME to the exact value of <code>[package] name</code></td>
    </tr>
    <tr>
      <td>cargo deb fails with missing metadata</td>
      <td>Cargo.toml is missing required fields</td>
      <td>Add description, license, and [package.metadata.deb] section</td>
    </tr>
    <tr>
      <td>cargo generate-rpm fails with missing assets</td>
      <td>Missing [package.metadata.generate-rpm] section</td>
      <td>Add the assets array to Cargo.toml as shown in section 4</td>
    </tr>
    <tr>
      <td>publish-release fails: token has no permission</td>
      <td>Missing contents: write on the publish job</td>
      <td>Verify the permissions block under publish-release</td>
    </tr>
    <tr>
      <td>GitHub App token generation fails</td>
      <td>Secrets not set or App not installed on the repository</td>
      <td>The workflow continues with GITHUB_TOKEN automatically; check section 5</td>
    </tr>
    <tr>
      <td>Cosign sign-blob fails with OIDC error</td>
      <td>id-token: write permission is missing</td>
      <td>Add id-token: write under the publish-release permissions block</td>
    </tr>
    <tr>
      <td>Workflow does not start on tag push</td>
      <td>Tag does not match the v* pattern</td>
      <td>Use a tag starting with v, for example v1.0.0</td>
    </tr>
    <tr>
      <td>cargo build fails with Cargo.lock mismatch</td>
      <td>Cargo.lock is not committed to the repository</td>
      <td>Commit Cargo.lock; the --locked flag requires it to be present</td>
    </tr>
  </tbody>
</table>

<hr />


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