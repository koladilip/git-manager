# Git Management Tools

This folder contains a unified, context-aware CLI tool for managing multiple Git repositories with intelligent behavior.

## 🛠️ Tool Overview

### `git-manage` - Unified Git Repository Manager
A comprehensive, context-aware CLI tool that adapts its behavior based on whether you're inside or outside a workspace. It combines repository discovery, workspace management, and interactive selection into a single powerful tool.

**Key Features:**
- 🎯 **Context-aware**: Different commands and behavior inside vs outside workspaces
- 🔍 **Smart Discovery**: Global organization cache with auto-refresh
- 📦 **Workspace Management**: Create and manage multi-repository workspaces
- ⚡ **Interactive Selection**: Fuzzy search with multi-select using `fzf`
- 🔄 **Auto-updates**: Laptop-friendly cron scheduling
- 🏢 **Multi-org Support**: Manage repositories from multiple organizations
- 🛠️ **Auto-install**: Automatic dependency installation with OS detection
- **Default branch detection** (uses cache, GitHub API, or local fallback)
- Autocomplete for commands and repo names (via completion script)

## 📋 Prerequisites

This tool requires the following dependencies:

- **git** - For repository operations
- **gh** (GitHub CLI) - For fetching repository information
- **fzf** - For interactive fuzzy selection

### 🚀 Automatic Installation
The `install` command can automatically install missing dependencies and the tool itself:

```bash
./git-manage install
```

This will:
- ✅ Detect your operating system (macOS/Linux)
- ✅ Check for missing dependencies
- ✅ Offer to install them using your system's package manager
- ✅ Install git-manage to `~/.local/bin`
- ✅ Add `~/.local/bin` to your PATH

### 🛠️ Manual Installation (if needed)

#### macOS (using Homebrew)
```bash
brew install git gh fzf
```

#### Ubuntu/Debian
```bash
# GitHub CLI
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install git gh fzf
```

#### Authentication
After installing GitHub CLI, authenticate with:
```bash
gh auth login
```

## 🚀 Quick Start

### Installation Benefits
- **System-wide access**: Available from any directory after installation
- **Standard location**: Installs to `~/.local/bin` (Unix standard)
- **No repository dependency**: Works even if the original repo is moved/deleted
- **Easy updates**: Run `git-manage install` again to update

### 1. One-command installation
```bash
./git-manage install
```
This will:
- Check and install missing dependencies (git, gh, fzf)
- Install git-manage to `~/.local/bin`
- Add `~/.local/bin` to your PATH in `~/.zshrc`
- Check GitHub CLI authentication status

### 2. Create a workspace
```bash
git-manage create my-projects
cd my-projects
```

### 3. Start adding repositories
```bash
# Interactive selection from your default orgs
git-manage clone

# Or clone a specific repository
git-manage clone awesome-repo   # Uses your default orgs
git-manage clone github/cli     # org/repo format
git-manage clone https://github.com/user/repo.git
```

## 📖 Commands Reference

### Global Commands (Available Everywhere)

#### `install`
Install git-manage to ~/.local/bin and add to PATH
```bash
git-manage install
```

#### `create <workspace>`
Initialize a new workspace with management scripts
```bash
git-manage create my-projects
```

#### `clone [repo/org/url]`
**Outside workspace**: Interactive picker or direct clone to current directory
**Inside workspace**: Add to workspace and clone

```bash
# Interactive selection (outside workspace)
git-manage clone

# Clone specific repo (outside workspace)  
git-manage clone cli/cli

# Inside workspace - adds to workspace
cd my-projects
git-manage clone awesome-repo
```

#### `import-org [org]`
Cache all repositories from an organization globally
```bash
git-manage import-org microsoft
git-manage import-org  # uses your first default org
```

#### `list-orgs`
Show all imported organizations
```bash
git-manage list-orgs
```

#### `refresh-orgs`
Refresh all cached organization data
```bash
git-manage refresh-orgs
```

#### `setup-org-cron`
Setup automatic refresh of organization data
```bash
git-manage setup-org-cron
```

### Workspace Commands (Inside Workspace Only)

#### `update`
Update all repositories in the workspace
```bash
git-manage update
```

#### `setup-cron`
Setup automatic updates for the workspace
```bash
git-manage setup-cron
```

#### `select-repos [org]`
Interactive multi-select for adding repositories to workspace
```bash
git-manage select-repos rudderlabs
git-manage select-repos  # uses your first default org
```

#### `sync`
Pull latest only if no local changes (uses correct default branch)
```bash
git-manage sync
```

#### `reset`
Hard reset all repos to their default branch (shows which branch)
```bash
git-manage reset
```

#### `reset-and-sync`
Reset all repos, then sync (pull latest)
```bash
git-manage reset-and-sync
```

#### `upgrade`
Add missing scripts to workspace
```bash
git-manage upgrade
```

## 🎯 Context-Aware Behavior

The tool automatically detects whether you're inside a workspace and adapts accordingly:

### Outside Workspace
- `clone` acts as a general-purpose repository picker
- Focus on discovery and one-off cloning
- Help shows global commands

### Inside Workspace  
- `clone` adds repositories to workspace state
- Additional workspace management commands available
- Help shows workspace-specific commands

## 🏢 Multi-Organization Support

### Default Organizations

You can set one or more default organizations. These are used for:
- Autocomplete in `clone` and `select-repos` commands
- The default org for interactive pickers
- The default org when cloning by repo name only

#### Setting/Changing Default Orgs

You will be prompted to set default orgs during installation. To change them later:

```bash
git-manage config-orgs
```
- Enter a comma-separated list (e.g. `rudderlabs,microsoft,facebook`)
- Press Enter to keep current settings
- Type `clear` to remove all default orgs

#### Example
```bash
git-manage config-orgs
# Default organizations: rudderlabs, microsoft
```

### Global Organization Cache
Organizations are cached globally in `~/.git-manage/`:
```
~/.git-manage/
├── rudderlabs/
│   └── repos.txt
├── microsoft/
│   └── repos.txt
└── github/
    └── repos.txt
```

- Autocomplete and interactive pickers work across all your configured default orgs.
- You can add, remove, or change orgs at any time with `git-manage config-orgs`.

## ⏰ Laptop-Friendly Scheduling

Both cron jobs use laptop-friendly times when your computer is likely to be on:

### Workspace Updates (`setup-cron`)
- 9:00 AM (weekdays) - Morning startup
- 2:00 PM (daily) - Afternoon check  
- 5:00 PM (weekdays) - Before end of day

### Organization Data Refresh (`setup-org-cron`)
- 9:00 AM (weekdays) - Morning startup
- 1:00 PM (daily) - Lunch break
- 6:00 PM (weekdays) - Evening wrap-up

## 📁 Workspace Structure

When you create a workspace, it creates:

```
workspace-name/
├── repos.txt          # List of cloned repositories
├── clone_repos.sh     # Clone all repositories
├── add_repo.sh        # Add and clone a repository
├── update_repos.sh    # Update all repositories
└── update.log         # Update logs (created after setup-cron)
```

## 🎮 Interactive Selection Controls

When using `fzf` for repository selection:

- **Arrow keys** or **typing**: Filter repositories
- **Tab**: Select/deselect repositories (multi-select)
- **Ctrl+A**: Select all filtered repositories
- **Ctrl+D**: Deselect all repositories  
- **Enter**: Confirm selection
- **Esc**: Cancel selection

### Display Format
```
repo-name [Language] ⭐123 🔒 - Repository description
```

**Indicators:**
- 🔒 - Private repository
- ⭐ - Star count
- [Language] - Primary programming language

## 🔧 Troubleshooting

### Common Issues

1. **"gh: command not found"**
   - Install GitHub CLI: https://cli.github.com/

2. **"fzf: command not found"**
   - Install fzf: https://github.com/junegunn/fzf#installation

3. **"Not authenticated with GitHub CLI"**
   - Run: `gh auth login`

4. **"Failed to fetch repositories"**
   - Check if the organization/user exists
   - Verify you have access to the repositories
   - Ensure GitHub CLI is authenticated

5. **SSH key issues**
   - Ensure your SSH key is added to GitHub
   - Test with: `ssh -T git@github.com`

### Debug Mode
Add `set -x` at the top of the script to see detailed execution.

## 📝 Examples

### Workflow 1: Setting up a new workspace
```bash
# Install tool globally
git-manage install

# Create workspace
git-manage create my-projects
cd my-projects

# Import organization data
git-manage import-org rudderlabs

# Interactively select repositories
git-manage select-repos

# Setup automatic updates
git-manage setup-cron
```

### Workflow 2: One-off repository cloning
```bash
# Clone a specific repository
git-manage clone cli/cli

# Interactive selection from organization
git-manage clone
```

### Workflow 3: Managing multiple organizations
```bash
# Import multiple organizations
git-manage import-org microsoft
git-manage import-org github
git-manage import-org rudderlabs

# Setup automatic refresh
git-manage setup-org-cron

# List all available organizations
git-manage list-orgs
```

## 🤝 Contributing

Feel free to submit issues and pull requests to improve this tool!

## 📄 License

This tool is provided as-is under the same license as the parent repository. 

## Shell Completion
- Completion logic is in `~/.git-manage/completion.sh`
- Your `.zshrc` or `.bashrc` only needs:
  ```sh
  source "$HOME/.git-manage/completion.sh"
  ```
- Autocomplete works for all commands and repo names (tab-completion)

## Upgrading Existing Workspaces
Run `git-manage upgrade` in any workspace to add missing scripts or update logic. 