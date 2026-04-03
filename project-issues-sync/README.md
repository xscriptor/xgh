# Project Issues Sync

<details open>
  <summary><b>Table of Contents</b></summary>
  <ul>
    <li><a href="#about">About</a></li>
    <li><a href="#how-it-works">How It Works</a></li>
    <li><a href="#installation-in-other-repositories">Installation</a></li>
    <li><a href="#permissions-setup-important">Permissions Setup</a></li>
    <li><a href="#customization">Customization</a></li>
  </ul>
</details>

<hr>

<h2 id="about">About</h2>

<details open>
  <summary>Click to expand</summary>

Automation to automatically synchronize the status of GitHub Issues (Opened, Closed, or Labeled) with the columns of a Kanban board (GitHub Projects V2) without requiring manual intervention.
</details>

<h2 id="how-it-works">How It Works</h2>

<details>
  <summary>Click to expand</summary>

This GitHub Actions workflow moves cards (Issues) between columns in your global GitHub Project (v2) based on issue events:

1. **New Issue (or Reopened):** Automatically places it in the `Todo` column.
2. **`in-progress` label added:** Moves it to the `In Progress` column. *(Pairs perfectly with github-roadmap-sync!)*
3. **Closed Issue:** Automatically moves it to the `Done` column.
</details>

<h2 id="installation-in-other-repositories">Installation in other repositories</h2>

<details>
  <summary>Click to expand</summary>

1. Copy the `.github/workflows/project-sync.yml` file into your destination repository.
2. Open the `.github/workflows/project-sync.yml` file and change `project_id: 1` to your actual project number.
   > **Note:** You can find this number in the URL of your Kanban board. If the URL is `github.com/users/xscriptor/projects/2`, your ID is `2`.

### Using alongside other workflows
If you already have other Workflows in your repository, **you can simply leave this file as a separate file**. 
Just place `project-sync.yml` inside the `.github/workflows/` directory next to your existing `.yml` files. GitHub Actions runs them completely independently, so they will not interfere with each other!
</details>

<h2 id="permissions-setup-important">Permissions Setup (Important)</h2>

<details>
  <summary>Click to expand</summary>

The new GitHub Projects V2 API (being tied to your global `xscriptor` account rather than a single project) does not work with the default generic Actions token. It requires extended permissions:

1. Create a **[Personal Access Token (Classic)](https://github.com/settings/tokens)**.
2. Grant it the following permissions: `repo`, `project` and `read:user`.
3. Go to your destination repository **Settings > Secrets and variables > Actions > New repository secret**.
4. Name it **`PROJECT_PAT`** and paste your token.
</details>

<h2 id="customization">Customization</h2>

<details>
  <summary>Click to expand</summary>

You can change the names of the columns where the bot will push the cards. If your board has columns like "To Do" or "Completed", edit the `project-sync.yml` file by modifying the `status_value` variable:

```yaml
status_value: "To Do"
```

</details>
