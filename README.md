# GitManagementHelpBook

# 🔐 Managing Multiple SSH Keys for Git (Work & Personal)

This guide provides a **complete, industry-standard configuration** for managing **multiple Git SSH keys** (e.g., work and personal GitLab or GitHub accounts).  
It ensures you never accidentally push to the wrong repository or use the wrong credentials.

---

## 🧩 Overview

When you work with multiple Git accounts (like **personal** and **work**), using a single SSH key can cause conflicts or permission issues.

This guide helps you:

- ✅ Create separate SSH keys per account  
- ✅ Configure SSH aliases for each identity  
- ✅ Set up platform-specific configurations (Windows + Linux)  
- ✅ Use Git safely without cross-account mix-ups  

---

## 🧰 Prerequisites

Before starting, ensure you have:

- **Git** installed  
- **OpenSSH** available (default in modern Linux, Windows 10/11)  
- Access to your Git hosting accounts (GitLab, GitHub, etc.)

---

## ⚙️ Step 1: Generate Separate SSH Keys

Generate one SSH key for **each** account.

### 🧠 Naming convention:
Use clear filenames like:
- `id_ed25519_gitlab_work`
- `id_ed25519_gitlab_personal`

### 🪟 Windows (PowerShell)
```powershell
ssh-keygen -t ed25519 -C "your_work_email@company.com" -f "$env:USERPROFILE\.ssh\id_ed25519_gitlab_work"
ssh-keygen -t ed25519 -C "your_personal_email@example.com" -f "$env:USERPROFILE\.ssh\id_ed25519_gitlab_personal"
```

### 🐧 Linux / macOS
```bash
ssh-keygen -t ed25519 -C "your_work_email@company.com" -f ~/.ssh/id_ed25519_gitlab_work
ssh-keygen -t ed25519 -C "your_personal_email@example.com" -f ~/.ssh/id_ed25519_gitlab_personal
```

💡 **Tip:**  
Use **different passphrases** for each key for better security.

---

## 🔑 Step 2: Add SSH Keys to Your Git Accounts

Copy the contents of each `.pub` file and add them to the respective accounts.

### Example (Windows PowerShell)
```powershell
Get-Content "$env:USERPROFILE\.ssh\id_ed25519_gitlab_work.pub"
```

### Example (Linux)
```bash
cat ~/.ssh/id_ed25519_gitlab_work.pub
```

Then paste it into:

- **GitLab:** `User → Preferences → SSH Keys`
- **GitHub:** `Settings → SSH and GPG Keys`

---

## 🧭 Step 3: Configure the SSH Config File

The SSH configuration file defines how SSH connects to different hosts and which key to use for each.

### File Location

| OS | Path |
|----|------|
| **Windows** | `C:\Users\<YourUsername>\.ssh\config` |
| **Linux/macOS** | `~/.ssh/config` |

### Create (if missing)

- On **Windows**, create manually:  
  ```powershell
  notepad $env:USERPROFILE\.ssh\config
  ```
- On **Linux/macOS**:  
  ```bash
  nano ~/.ssh/config
  ```

---

### Example Configuration

```bash
# ==========================
#  Personal GitLab Account
# ==========================
Host gitlab-personal
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/id_ed25519_gitlab_personal
    IdentitiesOnly yes

# ==========================
#  Work GitLab Account
# ==========================
Host gitlab-work
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/id_ed25519_gitlab_work
    IdentitiesOnly yes
```

### Explanation:
| Field | Description |
|--------|-------------|
| **Host** | Alias used in SSH/Git commands (e.g., `gitlab-work`). |
| **HostName** | Actual domain of your Git server (`gitlab.com`, `github.com`, or self-hosted). |
| **User** | Always `git` for GitLab/GitHub. |
| **IdentityFile** | Path to the private key file for that account. |
| **IdentitiesOnly yes** | Ensures only the specified key is used (prevents using wrong keys). |

---

## ⚙️ Step 4: Start SSH Agent and Add Keys

### 🪟 **Windows 10/11 (Recommended Commands)**
```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent

ssh-add $env:USERPROFILE\.ssh\id_ed25519_gitlab_personal
ssh-add $env:USERPROFILE\.ssh\id_ed25519_gitlab_work
```

✅ **Verification**
```powershell
ssh-add -l
```
You should see fingerprints for both keys.

---

### 🐧 **Linux/macOS**
Start the SSH agent:
```bash
eval "$(ssh-agent -s)"
```

Add your keys:
```bash
ssh-add ~/.ssh/id_ed25519_gitlab_personal
ssh-add ~/.ssh/id_ed25519_gitlab_work
```

✅ **Verify:**
```bash
ssh-add -l
```

---

## 🧪 Step 5: Test the SSH Connection

### Personal account
```bash
ssh -T git@gitlab-personal
```

### Work account
```bash
ssh -T git@gitlab-work
```

Expected output:
```
Welcome to GitLab, @yourusername!
```

---

## 🚀 Step 6: Clone Repositories Using Aliases

Use the `Host` alias instead of the default domain.

### Examples
```bash
# Personal repo
git clone git@gitlab-personal:username/personal-repo.git

# Work repo
git clone git@gitlab-work:company/work-repo.git
```

This ensures Git always uses the correct key.

---

## 💡 Best Practices & Professional Tips

### ✅ General
- Always use **separate keys** for each Git account.
- Use **clear key filenames** (`id_ed25519_gitlab_work`, `id_ed25519_github_personal`).
- Keep private keys **read-only**:
  ```bash
  chmod 600 ~/.ssh/id_ed25519_*
  ```

### ⚠️ Avoid Common Mistakes
| Mistake | Fix |
|----------|-----|
| Using default `git@gitlab.com` | Always use alias (`git@gitlab-work`, etc.) |
| Forgetting to start SSH agent | Ensure agent service is running |
| Wrong `.pub` key uploaded | Double-check each key’s `.pub` file |
| Config saved as `config.txt` (Windows) | Rename to `config` (no extension) |

### 🧠 Advanced
- Use `IdentitiesOnly yes` to prevent wrong key usage.
- For multiple Git providers (GitHub, GitLab, Bitbucket), add one block per host.
- You can include environment-specific settings using:
  ```bash
  Include ~/.ssh/config.work
  Include ~/.ssh/config.personal
  ```

### 🔒 Security Recommendations
- Use `ed25519` keys — more secure and faster than RSA.
- Protect your `.ssh` folder:
  ```bash
  chmod 700 ~/.ssh
  ```
- Never commit private keys or `.ssh/config` to repositories.

---

## 🧠 Troubleshooting

### Error: `Error connecting to agent: No such file or directory`
👉 SSH agent not running.

**Windows fix:**
```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

**Linux fix:**
```bash
eval "$(ssh-agent -s)"
```

### Error: `Permission denied (publickey)`
- The wrong key is being used or not added to agent.
- Check with `ssh -v git@gitlab-work` to see which key is being offered.

---

## 🧩 Quick Recap

| Step | Description |
|------|--------------|
| 1 | Generate separate SSH keys |
| 2 | Add public keys to respective Git accounts |
| 3 | Configure `~/.ssh/config` with aliases |
| 4 | Start SSH agent and add keys |
| 5 | Test connections |
| 6 | Clone using SSH aliases |

---

## 🧰 Example Directory Structure

```text
.ssh/
├── config
├── id_ed25519_gitlab_personal
├── id_ed25519_gitlab_personal.pub
├── id_ed25519_gitlab_work
└── id_ed25519_gitlab_work.pub
```

---

## 🏁 Final Notes

✅ This setup is **safe, portable, and DevOps-approved**.  
It eliminates credential mix-ups and ensures you always push to the correct Git account.  

If you maintain automation or CI/CD systems, apply the same logic — but **use environment variables or vault-managed SSH keys**, not personal keys.

---

> **Author’s Note:**  
> This guide follows secure DevOps best practices for SSH key isolation, Git identity management, and platform compatibility.  
> Tested on Windows 11 (PowerShell + Git Bash) and Ubuntu 22.04 LTS.
