# Using GitHub & SSH for Boritrader Projects
**Boritrade, LLC**

Our company mandates the use of SSH for authenticating GitHub connections across all Boritrader projects. This decision ensures a robust security posture while simplifying credential management. Unlike personal access tokens (whether classic or fine-grained), SSH keys provide persistent, non-expiring access without the risk of accidental exposure in configuration files. By standardizing on SSH, we eliminate the need for periodic token rotation, reduce overhead in managing multiple access methods, and enforce stronger encryption mechanisms, making it the optimal solution for secure, long-term repository interaction.

All developer commits must be made using SSH keys. Fine-grained personal access tokens (PATs) will be reserved exclusively for applications requiring read-only repository access. The use of GitHub Classic Keys is strictly prohibited.
---

## Creating an SSH Key on Local Machine (Linux):

### 1. **Generate the SSH Key**

1. Open a terminal and run the following command:
   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/github_personal_encryption -C "Boritrader GitHub"
   ```

   - `-t ed25519`: Specifies the Ed25519 algorithm, which is modern, secure, and fast.
   - `-f ~/.ssh/github_personal_encryption`: Custom filename for the key, stored in `~/.ssh/`.
   - `-C "Boritrader GitHub"`: Adds a comment for easier tracking.

2. When prompted, choose a passphrase (optional but recommended).
   - Press **Enter** to skip, or enter a secure passphrase for extra security.

### 2. **Add the SSH Key to the SSH Agent**

1. Start the SSH agent:
   ```bash
   eval "$(ssh-agent -s)"
   ```

2. Add the SSH key to the agent:
   ```bash
   ssh-add ~/.ssh/github_personal_encryption
   ```

### 3. **Add the Public Key to GitHub**

1. Copy the contents of the public key:
   ```bash
   cat ~/.ssh/github_personal_encryption.pub
   ```

2. Log in to [GitHub](https://github.com) and navigate to:
   - **Settings** → **SSH and GPG keys** → **New SSH Key**.

3. Paste the public key and provide a meaningful title (e.g., "Boritrader GitHub Key").

4. Click **Add SSH key**.

### 4. **Configure the SSH Config File**

To ensure GitHub uses the correct SSH key, configure the SSH client:

1. Open the SSH config file:
   ```bash
   nano ~/.ssh/config
   ```

2. Add this configuration:
   ```bash
   Host github.com
       HostName github.com
       User git
       IdentityFile ~/.ssh/github_personal_encryption
       IdentitiesOnly yes
   ```

   - `Host github.com`: Configures SSH settings for GitHub.
   - `IdentityFile ~/.ssh/github_personal_encryption`: Points to your SSH key.

3. Save and close the file (`Ctrl + O` to save, `Ctrl + X` to exit).

### 5. **Test the SSH Connection**

Test the setup by running:
```bash
ssh -T git@github.com
```

If successful, you should see:
```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

### 6. **Cloning Repositories**

You can now clone repositories using the SSH URL format:

```bash
git clone git@github.com:username/repository.git
```

The SSH config file will handle the correct key automatically.

---

## Steps to Update the Remote URL

### 1. **Check the Current Remote URL**

Verify the current remote URL:
```bash
git remote -v
```

Example output:
```
origin  git@github.com:Boritrade/Documentation.git (fetch)
origin  git@github.com:Boritrade/Documentation.git (push)
```

### 2. **Update the Remote URL**

To update the remote URL for the new organization repository:
```bash
git remote set-url origin git@github.com:Boritrade/Documentation.git
```

### 3. **Test the New Remote**

Ensure the remote works by running:
```bash
git pull origin main
```
