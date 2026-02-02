# Verified Commits

Git lets anyone set any name and email in their commits. Without signing, there's no proof a commit actually came from the person it claims to. Signed commits get a "Verified" badge on GitHub and can be required by branch protection rules.

You have two options: SSH signing (simpler) or GPG signing (traditional).

## Option A: SSH signing (recommended)

SSH signing reuses your existing SSH key. No extra software needed.

### 1. Tell git to use SSH for signing

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

Replace `~/.ssh/id_ed25519.pub` with the path to your actual public key if it differs.

### 2. Add your SSH key to GitHub as a signing key

1. Copy your public key:

   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

2. Go to **GitHub > Settings > SSH and GPG keys > New SSH key**.
3. Set **Key type** to **Signing Key**.
4. Paste the key and save.

### 3. Verify it works

```bash
git commit --allow-empty -m "test signed commit"
git log --show-signature -1
```

You should see `Good "git" signature` in the output.

## Option B: GPG signing

### 1. Generate a GPG key

```bash
gpg --full-generate-key
```

Choose **RSA and RSA**, **4096 bits**, and use the same email address you use on GitHub.

### 2. Get your key ID

```bash
gpg --list-secret-keys --keyid-format=long
```

Look for the line starting with `sec`. The key ID is the part after the `/`:

```text
sec   rsa4096/ABCDEF1234567890 2024-01-01 [SC]
```

In this example, the key ID is `ABCDEF1234567890`.

### 3. Tell git to use your GPG key

```bash
git config --global user.signingkey ABCDEF1234567890
git config --global commit.gpgsign true
```

### 4. Add your GPG key to GitHub

Export your public key:

```bash
gpg --armor --export ABCDEF1234567890
```

Copy the entire output (including the `-----BEGIN PGP PUBLIC KEY BLOCK-----` lines), then go to **GitHub > Settings > SSH and GPG keys > New GPG key** and paste it.

### 5. Verify it works

```bash
git commit --allow-empty -m "test signed commit"
git log --show-signature -1
```

You should see `Good signature` in the output.

## Commits made on GitHub

Commits created through the GitHub web interface (editing files, merging PRs, squash-merging) are automatically signed by GitHub. No setup needed for those.

## Troubleshooting

**"error: cannot run gpg: No such file or directory"**

Install GPG. On macOS: `brew install gnupg`. On Ubuntu: `sudo apt install gnupg`.

**GPG signing hangs or asks for passphrase in a weird way**

Tell GPG to use the terminal for passphrase input:

```bash
export GPG_TTY=$(tty)
```

Add that line to your `~/.bashrc` or `~/.zshrc` to make it permanent.

**"error: Load key ... invalid format" (SSH signing)**

Make sure `user.signingkey` points to the `.pub` (public) key file, not the private key.

**Commits show "Unverified" on GitHub**

Check that the email in `git config user.email` matches a verified email on your GitHub account.
