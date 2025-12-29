# 🔧 **GitHub Push Recovery Process (Deterministic, Reinstall‑Safe)**

A clean, reproducible workflow for restoring GitHub push access on **any machine**, after **any reinstall**, without relying on cached passwords, tokens, or GNOME keyring state.  
This process eliminates every common failure mode and ensures Git always uses SSH.

---

# 1️⃣ **Purge all stale GitHub authentication**

Fresh installs often inherit:

- cached or missing passwords  
- expired or missing tokens  
- GNOME keyring entries  
- credential helper junk  
- HTTPS fallback behavior  

All of this must be removed before switching to SSH.

### **A. Clear Git’s credential cache**
```bash
git credential-cache exit
```

### **B. Remove any stored GitHub credentials**
```bash
git credential reject <<EOF
protocol=https
host=github.com
EOF
```

### **C. Clear GNOME Keyring entries (Fedora)**
```bash
secret-tool clear service git host github.com
```

### **D. Disable credential helpers globally**
Prevents Git from silently re‑enabling HTTPS authentication.

```bash
git config --global --unset credential.helper
```

---

# 2️⃣ **Generate a fresh SSH keypair**

SSH avoids passwords, tokens, and keyring issues entirely.

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

# 🧩 **Why this process works**

| Failure Source | Eliminated By |
|----------------|---------------|
| Cached password | Clearing credential cache |
| GNOME keyring storing old credentials | `secret-tool clear` |
| Expired GitHub PAT | SSH keys don’t expire |
| HTTPS authentication fallback | Switching to SSH |
| Credential helper interference | `git config --global --unset credential.helper` |
| Machine reinstall | New keypair each time |
| Drift between machines | Deterministic steps |

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
