# 🔧 **GitHub Push Recovery Process (Deterministic, Reinstall‑Safe)**

# 1️⃣ **Clear all broken or stale GitHub credentials**

Fresh installs often inherit:

- cached or no passwords  
- expired tokens  or no tokens
- GNOME keyring entries  
- credential helper junk  

These must be wiped.

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

This removes the old, invalid GitHub password that caused the push failure.

---

# 2️⃣ **Generate a fresh SSH keypair**

This avoids passwords, tokens, and keyring issues entirely.

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

Copy → GitHub → Settings → SSH and GPG keys → **New SSH key**

Paste it.

---

# 5️⃣ **Switch your repo from HTTPS → SSH**

This is critical.  
HTTPS uses passwords/tokens.  
SSH uses your key.

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

# 6️⃣ **Push normally**

```bash
git push
```

No password.  
No token.  
No keyring.  
No drift.

---

# 🧩 **Why this process works every time**

Because it eliminates every failure point:

| Failure Source | Eliminated By |
|----------------|---------------|
| Cached password | Clearing credential cache |
| GNOME keyring storing old credentials | `secret-tool clear` |
| Expired GitHub PAT | SSH keys don’t expire |
| HTTPS authentication | Switching to SSH |
| Machine reinstall | New keypair each time |
| Drift between machines | Deterministic steps |

This is the **lowest‑friction**, **lowest‑risk**, **most reproducible** workflow for someone who formats machines regularly.

---

# 🧱 **The Minimal Reinstall Checklist (copy/paste)**

```
# 1. Clear old credentials
git credential-cache exit
git credential reject <<EOF
protocol=https
host=github.com
EOF
secret-tool clear service git host github.com

# 2. Generate new SSH key
ssh-keygen -t ed25519 -C "email"

# 3. Load key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 4. Add ~/.ssh/id_ed25519.pub to GitHub

# 5. Switch repo to SSH
git remote set-url origin git@github.com:USERNAME/REPO.git

# 6. Push
git push
```

---
