---
title: "Chapter 04 — GitHub Authentication"
description: "Chapter 04 — GitHub Authentication."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - getting-started
status: published
---

# Chapter 04 — GitHub Authentication

Before you can push code to GitHub or clone private repositories, Git needs to prove your identity to GitHub's servers. This chapter explains how that authentication works, covers the three main approaches — HTTPS with Personal Access Tokens, SSH keys, and the GitHub CLI — and helps you choose the right method for your situation.

---

## How Git Authenticates to GitHub

When you run `git push` or `git clone` against a GitHub URL, Git uses one of two **transport protocols**:

- **HTTPS** — URLs look like `https://github.com/user/repo.git`. Git sends credentials (a token or password) over an encrypted connection. This is the default when you clone without specifying a protocol.
- **SSH** — URLs look like `git@github.com:user/repo.git`. Git uses a cryptographic key pair instead of a password. No credentials are transmitted; the server verifies your identity by checking that you hold the private key matching a public key you have registered on GitHub.

Both protocols encrypt the data in transit. The difference is in *how identity is proved*: a secret token for HTTPS, or a key pair for SSH.

> **Further reading:** [About remote repositories — GitHub Docs](https://docs.github.com/en/get-started/getting-started-with-git/about-remote-repositories)

---

## HTTPS Authentication with Personal Access Tokens

GitHub removed support for password authentication in August 2021. If you use HTTPS, you must authenticate with a **Personal Access Token (PAT)** — a long, randomly generated string that acts like a password but can be scoped and revoked independently.

### Creating a Classic PAT

1. Sign in to GitHub and go to **Settings → Developer settings → Personal access tokens → Tokens (classic)**.
2. Click **Generate new token (classic)**.
3. Give the token a descriptive name (e.g., `laptop-dev`), set an expiry, and select scopes:
   - `repo` — full access to public and private repositories (required for pushing)
   - `workflow` — allows updating GitHub Actions workflow files
   - `read:org` — needed if you work with organisation repositories
4. Click **Generate token** and copy it immediately. GitHub will not show it again.

```bash
# When Git prompts for a password, paste your PAT instead
Username: your-github-username
Password: ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

> **Tip:** Classic PATs grant access to every repository you can reach with the selected scopes. For tighter control, see fine-grained PATs below.

### Using a PAT in a Clone URL

You can embed the token directly in a remote URL, which is useful in CI environments:

```bash
git clone https://YOUR_TOKEN@github.com/user/repo.git
```

Avoid this approach on shared machines — the token is visible in `git remote -v` output and in shell history.

### Fine-Grained PATs

GitHub also offers **fine-grained personal access tokens**, which restrict access to specific repositories and use a more granular permission model. They are found at **Settings → Developer settings → Personal access tokens → Fine-grained tokens**. Fine-grained PATs are the recommended choice for automation and third-party integrations.

> **Further reading:** [Managing your personal access tokens — GitHub Docs](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

---

## SSH Authentication

SSH authentication uses **asymmetric cryptography**: you generate a key pair consisting of a **private key** (kept secret on your machine) and a **public key** (shared with GitHub). When you connect, GitHub challenges your SSH client to prove it holds the private key — without ever transmitting the key itself. Once your public key is registered on GitHub, you can authenticate without entering a token or password on every push.

### Generating a Key Pair

The modern recommended algorithm is **Ed25519**, which produces short, fast keys with strong security:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

The `-C` flag adds a comment (usually your email) to help identify the key later. When prompted:

- **File location** — press Enter to accept the default (`~/.ssh/id_ed25519`).
- **Passphrase** — enter a strong passphrase. This encrypts the private key on disk so it cannot be used even if someone copies the file.

Two files are created:

| File | Contents |
|---|---|
| `~/.ssh/id_ed25519` | Your private key — **never share this** |
| `~/.ssh/id_ed25519.pub` | Your public key — safe to share |

If your system is too old to support Ed25519, use RSA with a 4096-bit key:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### Adding the Public Key to GitHub

1. Copy the contents of your public key:

```bash
cat ~/.ssh/id_ed25519.pub
# ssh-ed25519 AAAAC3Nza... your_email@example.com
```

2. Go to **GitHub → Settings → SSH and GPG keys → New SSH key**.
3. Give it a title (e.g., `MacBook Pro 2024`), leave the key type as **Authentication Key**, paste the public key, and click **Add SSH key**.

### Testing the Connection

```bash
ssh -T git@github.com
# Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

If you see this message, SSH authentication is working correctly.

### The SSH Agent

Typing your passphrase on every push is inconvenient. The **SSH agent** holds your decrypted key in memory for the duration of a session:

```bash
# Start the agent (if not already running)
eval "$(ssh-agent -s)"

# Add your key — you will be prompted for the passphrase once
ssh-add ~/.ssh/id_ed25519
```

On macOS, Keychain integration means the agent starts automatically at login and can persist your passphrase across reboots. Add the following to `~/.ssh/config`:

```
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

### Managing Multiple GitHub Accounts

If you have separate work and personal GitHub accounts, you can configure SSH to use different keys per host alias:

```
# ~/.ssh/config

Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal

Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work
```

Then use the alias in place of `github.com` when cloning:

```bash
git clone git@github-work:my-org/repo.git
```

> **Further reading:** [Connecting to GitHub with SSH — GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)

---

## The GitHub CLI (`gh`)

The **GitHub CLI** is an official command-line tool that wraps the GitHub API and handles authentication for you. It is the simplest starting point for new users and the most convenient option on developer machines.

### Installing `gh`

**Windows** (via winget or the MSI installer from [cli.github.com](https://cli.github.com)):

```bash
winget install --id GitHub.cli
```

**macOS** (via Homebrew):

```bash
brew install gh
```

**Linux** (Debian/Ubuntu):

```bash
(type -p wget >/dev/null || (sudo apt update && sudo apt install wget -y)) \
  && sudo mkdir -p -m 755 /etc/apt/keyrings \
  && wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg \
     | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
  && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
     | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
  && sudo apt update \
  && sudo apt install gh -y
```

### Authenticating with `gh auth login`

Run the interactive setup wizard:

```bash
gh auth login
```

You will be asked:

1. Which account type — **GitHub.com** or GitHub Enterprise
2. Preferred protocol — **HTTPS** or **SSH**
3. If SSH: whether to generate a new key or upload an existing one
4. How to authenticate — browser (recommended) or paste a token

After completing the flow, `gh` stores your credentials securely and configures Git to use them automatically for HTTPS operations. You do not need to manage tokens manually.

### Common `gh` Commands

```bash
gh repo clone user/repo         # Clone a repository
gh repo create                  # Create a new repository interactively
gh pr list                      # List open pull requests
gh pr create                    # Open a pull request from the current branch
gh issue list                   # List open issues
gh issue create                 # Create a new issue
gh auth status                  # Show current authentication state
gh auth logout                  # Remove stored credentials
```

> **Further reading:** [GitHub CLI manual](https://cli.github.com/manual/)

---

## Choosing Your Method

| Method | Setup effort | Best for |
|---|---|---|
| **gh CLI** | Low — one interactive wizard | Beginners; developer machines; daily GitHub work |
| **SSH keys** | Medium — generate key, add to GitHub, configure agent | Long-lived developer machines; teams that prefer no tokens; multiple accounts |
| **HTTPS + PAT** | Low-medium — generate token, configure credential helper | CI/CD pipelines; environments where SSH is blocked; scripting |

For a personal development machine, **`gh auth login` is the recommended starting point** — it gets you authenticated in minutes and handles credential storage transparently. You can always add SSH keys later for more control.

---

## Credential Caching for HTTPS

When you use HTTPS without `gh`, Git prompts for credentials on every operation unless you configure a **credential helper** to cache them.

### macOS — Keychain

macOS users get seamless integration via the bundled helper:

```bash
git config --global credential.helper osxkeychain
```

Credentials are stored in the macOS Keychain and retrieved automatically.

### Windows — Credential Manager

Git for Windows ships with a helper that uses Windows Credential Manager:

```bash
git config --global credential.helper manager
```

### Linux — Cache or Store

The `cache` helper holds credentials in memory for a fixed timeout (default: 15 minutes):

```bash
git config --global credential.helper 'cache --timeout=3600'
```

The `store` helper writes credentials to a plain-text file (`~/.git-credentials`) — convenient but less secure:

```bash
git config --global credential.helper store
```

> **Tip:** On Linux developer machines, prefer the SSH method to avoid storing plaintext tokens on disk.

> **Further reading:** [Caching your GitHub credentials in Git — GitHub Docs](https://docs.github.com/en/get-started/getting-started-with-git/caching-your-github-credentials-in-git)

---

## Summary

- GitHub requires token-based or key-based authentication — passwords are no longer accepted over HTTPS.
- **`gh auth login`** is the fastest path to a working setup on a developer machine.
- **SSH keys** offer the most seamless day-to-day experience once configured, with no tokens to manage or rotate.
- **PATs** are the right choice for automation and CI/CD pipelines where SSH is impractical.
- Always protect your private keys and tokens: never commit them to a repository, never share them in chat, and set reasonable expiry dates on PATs.

---

*Previous: [Chapter 03 — Installing & Configuring Git](ch03-install-configure.md)* · *Next: [Chapter 05 — Creating a Repository & Basic Operations](../part2/ch05-basic-operations.md)*
