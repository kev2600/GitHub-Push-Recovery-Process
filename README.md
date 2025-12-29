# 🔧 **GitHub Push Recovery Process (Deterministic, Reinstall‑Safe)**

This workflow restores GitHub push access on **any machine**, after **any reinstall**, with **zero reliance on cached credentials, tokens, or keyrings**.  
It is designed for users who regularly wipe their systems and require a **clean, reproducible, failure‑proof** authentication setup.

---

# 1️⃣ **Clear all broken or stale GitHub credentials**

Fresh installs often inherit:

- cached or missing passwords  
- expired or missing tokens  
- GNOME keyring entries  
- credential helper junk  
- HTTPS authentication fallback  

All of this must be wiped before switching to SSH.

### **A. Clear Git’s credential cache**
```bash
git credential-cache exit
```

### **B. Force Git to forget any stored GitHub credentials**
```bash
git credential reject <<EOF
protocol=https
host=github.com
EOF
```

### **C. Clear GNOME Keyring entries (Fedora uses this)**
```bash
secret-tool clear service git host github.com
```

### **D. Disable credential helpers globally**
Prevents Git from silently re‑enabling HTTPS auth.

```bash
git config --global --unset credential.helper
```

This ensures Git never tries to use stale passwords or tokens again.

---

# 2️⃣ **Generate a fresh SSH keypair**

SSH keys avoid all HTTPS authentication issues.

```bash
ssh-keygen -t ed25519 -C "your_email_here"
```

Press Enter for all prompts.

This creates:

- `~/.ssh/id_ed25519`  
- `~/.ssh/id_ed25519.pub`

---

# 3️⃣ **Load the key into the SSH agent**

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

---

# 4️⃣ **Add the public key to GitHub**

Show the key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy → GitHub → Settings → **SSH and GPG keys** → New SSH key  
Paste it.

---

# 5️⃣ **Verify SSH authentication**

This confirms the key works and GitHub recognizes it.

```bash
ssh -T git@github.com
```

Expected output:

```
Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

If you see this, SSH is fully functional.

---

# 6️⃣ **Switch your repo from HTTPS → SSH**

HTTPS uses passwords/tokens.  
SSH uses your key.  
SSH is deterministic and survives reinstalls.

Inside your repo:

```bash
git remote set-url origin git@github.com:USERNAME/REPO.git
```

Verify:

```bash
git remote -v
```

You should see:

```
origin  git@github.com:USERNAME/REPO.git (push)
```

---

# 7️⃣ **Push normally**

```bash
git push
```

No password.  
No token.  
No keyring.  
No drift.  
No failure points.

---

# 🧩 **Why this process works every time**

| Failure Source | Eliminated By |
|----------------|---------------|
| Cached password | Clearing credential cache |
| GNOME keyring storing old credentials | `secret-tool clear` |
| Expired GitHub PAT | SSH keys don’t expire |
| HTTPS authentication fallback | Switching to SSH |
| Machine reinstall | New keypair each time |
| Drift between machines | Deterministic steps |
| Credential helper interference | `git config --global --unset credential.helper` |

This workflow removes every variable that causes GitHub push failures.

---

# 🧱 **Minimal Reinstall Checklist (copy/paste)**

```
# 1. Clear old credentials
git credential-cache exit
git credential reject <<EOF
protocol=https
host=github.com
EOF
secret-tool clear service git host github.com
git config --global --unset credential.helper

# 2. Generate new SSH key
ssh-keygen -t ed25519 -C "email"

# 3. Load key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 4. Add ~/.ssh/id_ed25519.pub to GitHub

# 5. Verify SSH works
ssh -T git@github.com

# 6. Switch repo to SSH
git remote set-url origin git@github.com:USERNAME/REPO.git

# 7. Push
git push
```

---
