---
title: "Appendix C — Signed Commits & GPG"
description: "Appendix C — Signed Commits & GPG."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
status: published
---

# Appendix C — Signed Commits & GPG

Git records the author and committer name and email for every commit, but it does not verify them. Anyone can configure `user.name` and `user.email` to any value and push commits that appear to come from someone else. **Commit signing** closes this gap: a cryptographic signature attached to a commit proves that it was created by the holder of a specific private key, and that it has not been tampered with since signing.

GitHub displays a **Verified** badge next to signed commits, giving reviewers confidence that the commit genuinely came from the stated author.

---

## Signing Methods

Git supports three signing mechanisms:

| Method | Key type | Setup complexity | Best for |
|---|---|---|---|
| **GPG** | OpenPGP (RSA, Ed25519) | Moderate | Traditional; widely supported |
| **SSH** | SSH key pair | Low | Reuses existing SSH keys |
| **S/MIME** | X.509 certificate | High | Enterprise / corporate PKI |

GPG is the oldest and most widely documented method. SSH signing (introduced in Git 2.34, 2021) is simpler to set up if you already have an SSH key configured for GitHub authentication. This appendix covers GPG in depth and SSH signing as a streamlined alternative.

---

## GPG Signing

### Install GPG

**macOS:**

```bash
brew install gnupg
```

**Linux (Debian/Ubuntu):**

```bash
sudo apt install gnupg
```

**Windows:**

Download [Gpg4win](https://gpg4win.org/) or install via `winget`:

```bash
winget install GnuPG.GnuPG
```

### Generate a GPG key

```bash
gpg --full-generate-key
```

When prompted:

- **Key type:** `ECC (sign and certify)` with `Curve 25519` is recommended for new keys (Ed25519). Alternatively, `RSA and RSA` with 4096 bits is widely compatible.
- **Expiry:** setting an expiry (e.g. 2 years) is good practice — an expiring key limits damage if the private key is later compromised.
- **User ID:** use the same name and email address as your Git `user.name` and `user.email`. GitHub matches the signing key's email to your verified GitHub email addresses.
- **Passphrase:** protect the private key with a strong passphrase.

### List your keys

```bash
gpg --list-secret-keys --keyid-format=long
```

Example output:

```
sec   ed25519/3AA5C34371567BD2 2024-01-15 [SC]
      1234567890ABCDEF1234567890ABCDEF12345678
uid   [ultimate] Alice Example <alice@example.com>
ssb   cv25519/4BB6D45482678CE3 2024-01-15 [E]
```

The long key ID is the part after the algorithm: `3AA5C34371567BD2`. The full 40-character fingerprint is the line below `sec`.

### Export the public key to GitHub

```bash
gpg --armor --export 3AA5C34371567BD2
```

Copy the output (the block starting `-----BEGIN PGP PUBLIC KEY BLOCK-----`) and add it to GitHub at **Settings → SSH and GPG keys → New GPG key**.

### Configure Git to use the key

```bash
git config --global user.signingkey 3AA5C34371567BD2
git config --global gpg.program gpg
```

On macOS with Homebrew GPG, you may need to specify the full path:

```bash
git config --global gpg.program $(which gpg)
```

### Sign commits

To sign a single commit, add `-S`:

```bash
git commit -S -m "Add feature"
```

To sign all commits automatically:

```bash
git config --global commit.gpgsign true
```

With `commit.gpgsign true`, every `git commit` is signed without needing `-S`. Git will prompt for your GPG passphrase (or use a GPG agent if one is running).

### Sign tags

Annotated tags can also be signed with `-s` (lowercase):

```bash
git tag -s v1.2.0 -m "Release 1.2.0"
```

Verify a signed tag:

```bash
git tag -v v1.2.0
```

### Verify commit signatures

```bash
git log --show-signature          # full GPG output for each commit
git verify-commit <sha>           # verify a specific commit
```

Output from `git verify-commit`:

```
gpg: Signature made Mon 15 Jan 2024 10:23:01 UTC
gpg:                using EDDSA key 3AA5C34371567BD2
gpg: Good signature from "Alice Example <alice@example.com>" [ultimate]
```

---

## GPG Agent — Avoiding Repeated Passphrase Prompts

Entering a passphrase on every commit is friction. `gpg-agent` caches the passphrase for a configurable duration.

On Linux/macOS, `gpg-agent` usually starts automatically. To ensure it is running and configure its cache timeout, edit (or create) `~/.gnupg/gpg-agent.conf`:

```
default-cache-ttl 3600
max-cache-ttl 86400
```

Then reload:

```bash
gpgconf --kill gpg-agent
gpgconf --launch gpg-agent
```

On macOS, the `pinentry-mac` program integrates with the macOS Keychain so the passphrase can be stored permanently:

```bash
brew install pinentry-mac
echo "pinentry-program $(which pinentry-mac)" >> ~/.gnupg/gpg-agent.conf
gpgconf --kill gpg-agent
```

---

## SSH Signing (Simpler Alternative)

Since Git 2.34, you can use an SSH key — the same key you already use for GitHub authentication — to sign commits. No GPG installation required.

### Configure SSH signing

Tell Git to use SSH for signing and specify your public key:

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

Replace `~/.ssh/id_ed25519.pub` with the path to your actual public key file.

### Add the signing key to GitHub

Go to **Settings → SSH and GPG keys → New SSH key**, select **Signing Key** as the key type, and paste your public key. Note that GitHub requires a separate entry for signing keys — the same key used for authentication needs to be added again as a signing key.

### Verify SSH signatures locally

SSH signature verification requires an **allowed signers file** that maps email addresses to public keys:

```bash
# Create the file
echo "alice@example.com $(cat ~/.ssh/id_ed25519.pub)" > ~/.ssh/allowed_signers

# Tell Git where to find it
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
```

Then verify normally:

```bash
git log --show-signature
git verify-commit <sha>
```

---

## Vigilant Mode on GitHub

GitHub's **vigilant mode** marks commits as **Unverified** if they have no signature, rather than showing nothing. This makes the absence of a signature explicit and visible to reviewers.

Enable it at **Settings → SSH and GPG keys → Vigilant mode → Enable vigilant mode**.

With vigilant mode enabled:

| Commit state | Badge shown |
|---|---|
| Signed with a key verified by GitHub | **Verified** (green) |
| Signed but key not verified / expired | **Unverified** (grey) |
| Not signed | **Unverified** (grey) |

This is useful for teams that require signed commits via branch protection rules — reviewers can immediately see any commit that bypassed the requirement.

### Requiring signed commits via branch protection

In **Settings → Branches → Branch protection rules**, enable **Require signed commits**. This blocks any push to the protected branch that contains unsigned commits — including force-pushes and squash merges that re-create commits without signatures.

---

## Key Management

### Backing up your private key

```bash
gpg --export-secret-keys --armor 3AA5C34371567BD2 > alice-private-key.asc
```

Store the backup encrypted in a secure location (password manager, encrypted drive). If you lose the private key, you can no longer sign commits with that key and will need to generate a new one and re-verify with GitHub.

### Revoking a compromised key

If a private key is compromised, generate a **revocation certificate** immediately (ideally before it is needed):

```bash
gpg --gen-revoke 3AA5C34371567BD2 > revoke.asc
```

To revoke, import the certificate:

```bash
gpg --import revoke.asc
```

Then upload the revoked key to a public keyserver so others' installations see the revocation:

```bash
gpg --keyserver keyserver.ubuntu.com --send-keys 3AA5C34371567BD2
```

Remove the revoked key from GitHub under **Settings → SSH and GPG keys**.

### Key expiry

When a key expires, existing commits signed with it show as **Unverified** on GitHub (because the key can no longer be confirmed as valid at the time of verification). Extend the expiry before it lapses:

```bash
gpg --edit-key 3AA5C34371567BD2
# At the gpg> prompt:
expire     # follow prompts to set new expiry
save
```

Re-export and update the key on GitHub after extending the expiry.

---

## Common Pitfalls

**Email mismatch**

GitHub matches the signing key's UID email to your verified GitHub email addresses. If `user.email` in Git does not match any email in the GPG key's UID, GitHub cannot verify the signature. Run `gpg --list-keys` and confirm the email matches your `git config user.email`.

**GPG agent not running on commit**

If `gpg-agent` is not running, Git may fail with `error: gpg failed to sign the data`. Start the agent with `gpgconf --launch gpg-agent` or add it to your shell startup script.

**TTY errors in non-interactive environments**

Signing in CI or scripts can fail with `Inappropriate ioctl for device`. The passphrase prompt has no TTY. Solutions:

```bash
# Use a passphrase-free subkey (not recommended for personal keys)
# Or suppress the TTY requirement:
export GPG_TTY=$(tty)
```

In CI, use a passphrase-free GPG key stored in a secret, or skip signing on CI builds (CI commits are typically not human-authored).

**SSH key already added as authentication key**

On GitHub, a key added as an **authentication** key cannot also be used as a **signing** key from the same entry. You must add it a second time explicitly choosing **Signing Key** as the type.

---

## Quick Reference

```bash
# GPG — generate key
gpg --full-generate-key

# GPG — list keys
gpg --list-secret-keys --keyid-format=long

# GPG — export public key (to paste into GitHub)
gpg --armor --export <key-id>

# Configure Git to sign with GPG
git config --global user.signingkey <key-id>
git config --global commit.gpgsign true

# Configure Git to sign with SSH
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true

# Sign a single commit explicitly
git commit -S -m "message"

# Sign an annotated tag
git tag -s v1.0.0 -m "Release 1.0.0"

# Verify commits
git log --show-signature
git verify-commit <sha>

# Verify a signed tag
git tag -v v1.0.0

# Backup private key
gpg --export-secret-keys --armor <key-id> > backup.asc

# Revoke a key
gpg --gen-revoke <key-id> > revoke.asc
gpg --import revoke.asc
```

---

## Summary

- Commit signing cryptographically proves authorship; GitHub displays a **Verified** badge for commits signed with a key registered to your account.
- **GPG signing:** generate a key with `gpg --full-generate-key`, export the public key to GitHub, set `user.signingkey` and `commit.gpgsign true`. Use `gpg-agent` to cache the passphrase.
- **SSH signing:** simpler if you already have an SSH key; set `gpg.format ssh` and `user.signingkey`, add the key to GitHub as a **Signing Key** (separate from the authentication key entry).
- **Vigilant mode** marks unsigned commits explicitly as Unverified. **Branch protection** can enforce signing on protected branches.
- Manage keys responsibly: back up private keys, set expiry dates, and revoke compromised keys promptly.

> **Further reading:** [GitHub Docs — Managing commit signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification) · [GPG best practices (riseup.net)](https://riseup.net/en/security/message-security/openpgp/best-practices)

---

*Previous: [Appendix B — Git LFS (Large File Storage)](appB-git-lfs.md)*
