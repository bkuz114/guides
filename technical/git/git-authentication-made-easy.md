# Git Authentication Made Easy

**Audience:** Developers or other technically savvy users comfortable with the command line and basic git concepts, but who have never needed to understand the Git authentication stack.

**Goal:** To explain what is happening under the hood so that when it breaks, you know exactly which part failed and how to fix it, without guessing.

---

## Introduction: What This Guide Explains

When you perform certain Git operations against a remote repository (push, pull, fetch, clone), the remote requires you to prove who you are. This guide explains how Git handles authentication, what can go wrong, and how to fix it.

Authentication depends on two independent things:

1. **The remote URL of the specific repo you are working in** (decides whether SSH or HTTPS is used as the authentication method).
2. **Your Git configuration** (decides how credentials are managed for whichever authentication method is set for that repo).

A misconfiguration in one can be masked by a correct configuration in the other. You might have broken HTTPS credentials but never notice because you are working in an SSH-configured repo. Or you might have a perfect HTTPS credential helper but still be prompted because you are in a repo with an SSH URL and a broken SSH key.

---

## The Core Concepts (Read this first)

### The Two Authentication Methods: SSH vs HTTPS

You are connecting to a remote server and must authenticate against it. There are two distinct approaches Git uses to complete that authentication.

- **SSH:** Uses key pairs (`~/.ssh/id_rsa`). Authentication happens at the start of the connection. Git never sees a password. Once your key is in the agent, it works globally. Credential helpers are irrelevant. If your remote URL starts with `git@github.com`, you are on SSH.
- **HTTPS:** Uses standard HTTP authentication. The server returns a `401 Unauthorized`, and Git must supply a username/password (or token) via an `Authorization` header. If your remote URL starts with `https://github.com`, you are on HTTPS.

### Critical nuance: Per repo vs Global settings

**Critical nuance:** Authentication method is a per-repo setting, **not** a global setting. It is determined by how you initially clone the repo, but it can be changed later (see Appendix D for commands). When you `git push` (or perform other actions), the behavior you observe (e.g., git prompting for credentials, or the push working) depends on how that specific repo is set up – **not** just global git settings on your machine.

There are some other factors at play as well. Ultimately, behavior will depend on:

1. The repo you are trying to perform the action on, and how it is configured on your machine (ssh vs https)
2. The repo's state (public vs private). For example, reading from a public repo may work anonymously without any authentication, while pushing to the same repo always requires it. This is why a broken credential setup can go unnoticed until you try to push.
3. What action you are trying to perform and your permissions (for example, you could have an auth token for a machine associated with your account, but it does not allow for changing github actions - only for pushing)
4. The state of your global configuration on that machine (ssh key setup, https credential manager, etc.)

---

## SSH authentication

SSH authentication is arguably simpler to manage and conceptualize.

This section provides a brief conceptual overview of how SSH authentication works with Git.

It then provides a walk through of setting up for SSH authentication on your machine **for a specific GitHub account**. This will be a **machine-wide** setup. Once SSH authentication is corectly set up, then you can push to any repo on your machine **with an SSH remote** (so long as you have permissions to do so).

### How SSH Authentication Works with Git

When Git connects to an SSH remote:

1. Git invokes the SSH client on the local machine.
2. The SSH client looks for a private key (typically `~/.ssh/id_rsa` or `~/.ssh/id_ed25519`).
3. If the key is loaded in an SSH agent, the agent handles authentication.
4. The server verifies the corresponding public key.
5. If it matches a key registered on your GitHub account, authentication succeeds.
6. Git proceeds with the operation. No credential helper is involved.

If the key is missing, passphrase-protected without an agent, or not registered on GitHub, authentication fails and you get an SSH error (often a permission denied or a prompt for the key's passphrase).

### Walkthrough: Setting up SSH authentication for a new machine

[source](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

#### Step 1: Generate the SSH key

- Open a terminal.

- Create an SSH key using `ed25519` and using you email associated with GitHub

```
$ ssh-keygen -t ed25519 -C "youremail@email.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/c/Users/Ivan/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Ivan/.ssh/id_ed25519
Your public key has been saved in /c/Users/Ivan/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:lPMfpkBsi664J+7Vnb3Wvd5H1mNZBDqhQwarLcp7dEk youremail@email.com
The key's randomart image is:
+--[ED25519 256]--+
|        ..o . .. |
|       . = . o  .|
|        O o o  . |
|       BE+ . .  .|
|      +.S.. o   +|
|   . +.oo+ + . ++|
|    +.o.o o....o.|
|  .o.o.   ... ...|
| o=+o.   ..  .o.o|
+----[SHA256]-----+

```

**I accept defaults and no passphrase**

#### Step 2. Add the new SSH key identity

```
$ ssh-add /c/Users/Ivan/.ssh/id_ed25519
Identity added: /c/Users/Ivan/.ssh/id_ed25519 (youremail@email.com)
```

**Note**: If you get `"Could not open a connection to your authentication agent"`, start the agent in the background:

```
eval "$(ssh-agent -s)"
```

Now run the add command again

#### Step 3. Add the SSH public key to your GitHub account.

1. Login into GitHub.
2. Account Settings → SSH and GPG keys → New SSH Key
3. Title the key (e.g. HP laptop)
4. Add in the **entire text** from the .pub file that was generated.

#### Step 4. Test it works

```
$ ssh -T git@github.com
Hi Ivan! You've successfully authenticated, but GitHub does not provide shell access.
```

Alternatively, git push from an SSH configured repo. **The push should work automatically, without any credential prompting.**

---

## HTTPS authentication

This section provides a walk through of the concepts related to HTTPs authentication, as well as how to configure git globally to work with HTTPs configured remotes.

### Core Concepts

#### Part 1: How Git Handles HTTPS Credentials

When Git needs credentials for an HTTPS remote:

1. Git attempts the connection.
2. Server returns `401 Unauthorized`.
3. Git looks at a config setting called `credential.helper`.
4. If set, Git executes that program and asks it for credentials.
5. If not set, Git prompts you directly in the terminal for your credentials.

**The program referenced in step 4 is called a credential helper.**

#### Part 2: The Credential Helpers

A credential helper does two things:

1. **Store** a secret somewhere (file, RAM, OS keychain).
2. **Retrieve** that secret when Git asks for it.

The helpers differ in **where they store**, **whether they can also obtain credentials themselves**, and **what platforms they work on**.

| Helper | Where it stores | How it obtains credentials | Availability | Limitations |
| :--- | :--- | :--- | :--- | :--- |
| `store` | Plaintext file on disk (`~/.git-credentials`) | Prompts you in terminal. You paste a token. It saves it. | All platforms | Insecure. Token readable by anyone with filesystem access. No encryption. |
| `cache` | RAM (a daemon process) | Prompts you in terminal. Saves to RAM. | All platforms | Ephemeral. Dies on reboot/sleep. Not for permanent use. |
| `osxkeychain` | Apple Keychain | Prompts you in terminal. You paste a token. It saves it. | macOS only | Requires GUI session. Keychain must be unlocked. |
| `wincred` | Windows Credential Manager | Prompts you in terminal. You paste a token. It saves it. | Windows only | Legacy helper. Works for basic PAT storage but does not handle OAuth flows. GCM (manager) is preferred on modern Windows. |
| `libsecret` | Linux Secret Service (GNOME Keyring / KWallet) | Prompts you in terminal. You paste a token. It saves it. | Linux only | Requires desktop environment with Secret Service running. Headless servers often lack this. |
| `manager` / `manager-core` (GCM) | OS keychain (whatever is available) | Opens a browser. Logs into GitHub. Gets a token automatically. Saves it. | macOS, Windows, Linux | Requires GUI for browser flow on first auth. Heavier install than dumb helpers. |

**All of these are credential helpers.** They all plug into Git the same way. Git does not care how they work internally.

#### Part 3: The Two Behaviors of Credential Helpers

Credential helpers fall into two categories based on how they obtain credentials.

##### Behavior A: Dumb Helpers

`store`, `cache`, `osxkeychain`, `wincred`, and `libsecret` are dumb.

They do not know what GitHub is. They do not know what OAuth is. They only know:
- "Save this string under this key."
- "Give me the string for this key."

When they don't have a secret, they ask **you** for it via terminal prompt. You paste in a Personal Access Token. They save it.

##### Behavior B: Smart Helpers

`manager` / `manager-core` (GCM) is smart.

When GCM doesn't have a secret, it does not ask you to paste anything. Instead:
1. It opens a browser.
2. You log into GitHub (including 2FA if enabled).
3. GitHub gives GCM a token.
4. GCM saves that token to the OS keychain.
5. GCM returns the token to Git.

GCM automates the token acquisition process. That is the only difference between it and the dumb helpers. In every other respect, it is the same: Git calls it, it returns a secret.

#### Part 4: How External Programs Interact with Credential Helpers

A credential helper is just a program that stores and retrieves secrets.

**Any other program can also write to that same credential helper.**

This is a general pattern. Many CLI tools authenticate you to a service, obtain a token, and then write that token to whatever credential helper you have configured. GitHub CLI (`gh`) is one example. There are equivalents for GitLab, cloud providers, and other services.

These tools are **not** credential helpers themselves. They are **writers**. They obtain a secret and store it in your configured credential helper.

**The critical implication:**

- If you have a credential helper configured (e.g., `osxkeychain`), these tools write the token there. Git benefits because Git reads from the same place.
- If you have **no** credential helper configured, these tools have nowhere to write. They typically store the token in their own config file. Git remains unaware and will still prompt you for credentials when you push.

**The practical rule:** Configure a credential helper for Git **first**. Then run any external auth tool. That way, the tool writes to the same place Git reads from, and everything works.

### Setting up HTTPs authentication up a new machine

#### Step 1: Check if a helper is already set

```bash
git config --global credential.helper
```

If it returns something (e.g., `manager`, `manager-core`, `osxkeychain`, `wincred`), an installer or previous setup already configured one. If it returns empty, you need to set one.

#### Step 2: Choose your helper

| Situation | Recommendation |
| :--- | :--- |
| Modern macOS / Windows / Linux with GUI | `manager` / `manager-core` (GCM) |
| macOS without GCM | `osxkeychain` |
| Windows without GCM (legacy setups) | `wincred` |
| Linux with GNOME Keyring or KWallet | `libsecret` |
| Headless Linux, CI, VMs, no keyring | `store` |

#### Step 3: Configure it

**Option A: GCM (smart helper, handles browser flow automatically)**

```bash
git config --global credential.helper manager
```

**Note:** `manager` and `manager-core` refer to the same helper (Git Credential Manager). Older installs and documentation may use `manager-core`. Both work. Newer installs default to `manager`. If your system already has `manager-core` configured, there is no need to change it.

Then push. A browser opens. Log in. Done.

**Option B: OS keychain (dumb helper, requires a PAT)**

macOS:
```bash
git config --global credential.helper osxkeychain
```

Windows:
```bash
git config --global credential.helper wincred
```

Linux (libsecret):
```bash
sudo apt-get install libsecret-1-0 libsecret-1-dev
cd /usr/share/doc/git/contrib/credential/libsecret
sudo make
git config --global credential.helper /usr/share/doc/git/contrib/credential/libsecret/git-credential-libsecret
```

**Note:** The contrib directory path varies by distribution. On Debian/Ubuntu it is typically `/usr/share/doc/git/contrib/credential/libsecret`. On Fedora, Arch, and others, the contrib files may not be installed by default. If you cannot find this path, check your distribution's documentation for where git credential helpers are located, or use `store` as a fallback.

Then push. Git prompts you. Enter username and PAT.

**Option C: Plaintext (works everywhere, insecure)**

```bash
git config --global credential.helper store
```

Then push. Git prompts you. Enter username and PAT.

#### Step 4: If using a dumb helper, create a PAT

1. Go to GitHub → Settings → Developer Settings → Personal Access Tokens.
2. Generate a new token (Classic or Fine-grained).
3. Grant it `repo` scope.
4. Copy the token. Treat it like a password.

#### Step 5: Testing if everything is working

1. Navigate to a repo on your machine with **an HTTPS remote**.
2. Execute the following command:

```bash
git push --dry-run
```

**This will NOT push to the remote. This only simulates a push.**


You should see whatever you would normally see from a `git push` in your repo's current state—for example: "Everything up-to-date" if there are no unpushed commits, or a list of branches that would be pushed. Either way, if it completes without an authentication error, your credentials are working.

---

## Troubleshooting

### "Git asks for a password when I push/pull/fetch"

First, determine which authentication method this repo uses (HTTPS or SSH):

```bash
git remote -v
```

- If it starts with `https://github.com/`: you are on HTTPS. This is a credential helper issue. See the HTTPS scenarios below.
- If it starts with `git@github.com:`: you are on SSH. This is an SSH key issue, not a credential helper issue. See the SSH scenarios below.

### "I'm on SSH and getting a password prompt"

This is not a GitHub credential prompt. It is one of the following:

1. **Your SSH key has a passphrase and is not loaded in an agent.**

   You are being asked for the key's passphrase, not your GitHub password.

   **Fix:** Load the key into the agent:

   ```bash
   ssh-add ~/.ssh/id_ed25519
   ```

   (Replace with your actual key filename.)

2. **Your local system password is being requested.**

   This can happen on macOS when the SSH agent tries to access the keychain.

   **Fix:** Unlock the keychain or re-add the key.

3. **You have no SSH key, or it is not registered on GitHub.**

   SSH first tries key-based auth. If no key works, it falls back to asking for a password. GitHub does not support SSH password auth, so this will always fail. The prompt is not coming from Git. It's SSH itself.

   **Fix:**

   - Check for existing keys: `ls ~/.ssh/`
   - Check what's registered: compare `cat ~/.ssh/id_*.pub` against GitHub → Settings → SSH and GPG keys.
   - If no key: generate one (`ssh-keygen -t ed25519 -C "your_email@example.com"`).
   - If key exists but not registered: add the public key to GitHub.

### "I'm on SSH and getting 'permission denied (publickey)'"

Your SSH key exists but GitHub rejected it.

**Fix:**

- Check which keys you have: `ls ~/.ssh/`
- Check what's registered on GitHub: Settings → SSH and GPG keys.
- Ensure the public key (`~/.ssh/id_*.pub`) matches what's registered.
- If you have multiple keys, force SSH to use the correct one by adding to `~/.ssh/config`:
  ```
  Host github.com
    IdentityFile ~/.ssh/id_ed25519
  ```

### "I'm on HTTPS and Git asks for credentials every time even though I set a helper"

**Fix:**

Check what helper is actually set:

```bash
git config --show-origin --get credential.helper
```

This tells you which file set it. It may be overridden locally or system-wide.

Possible causes:

- **Helper set to `cache`:** It stores in RAM and dies on reboot. You will be prompted again after every reboot. Use a persistent helper instead.
- **Helper set in one file but overridden in another:** The `--show-origin` output will show this. Unset the override or consolidate.
- **Helper set to `store` but file deleted or permissions changed:** Check `~/.git-credentials` exists and is readable.
- **Helper set but stored credential is invalid:** The helper returns an invalid token. GitHub rejects it. Git clears it and prompts again. Generate a new PAT.

### "I'm on HTTPS and Git prompts every time, and I have no helper set"

You have no credential helper configured. Git falls back to prompting you directly every time.

**Fix:**

Set one:

```bash
git config --global credential.helper manager        # GCM, modern, handles browser flow
git config --global credential.helper manager-core   # GCM, older alias, same helper
git config --global credential.helper osxkeychain   # macOS keychain
git config --global credential.helper wincred       # Windows Credential Manager
git config --global credential.helper libsecret     # Linux keyring
git config --global credential.helper store         # plaintext, works everywhere, insecure
```

Then push. Git will prompt once, store the credential, and stop prompting.

### "I'm on HTTPS and I ran an external auth tool but Git still prompts"

You have no credential helper configured. The tool stored the token in its own config file, not where Git reads from.

**Fix:**

Configure a helper:

```bash
git config --global credential.helper manager        # or manager-core / osxkeychain / wincred / libsecret / store
```

Then either re-run the external auth tool, or push and enter a PAT when Git prompts.

### "I'm on HTTPS and it asked once, worked for a while, now asks again"

The token expired or was revoked.

**Fix:**

- **Dumb helper (`store`, `osxkeychain`, `wincred`, `libsecret`):** Generate a new PAT on GitHub. Next prompt, paste the new PAT.
- **GCM (`manager` / `manager-core`):** Push again to trigger the browser flow and obtain a fresh token.
- **If you changed your GitHub password:** All existing tokens were revoked. Generate new ones and re-authenticate.

### "I'm on HTTPS and `git push --dry-run` succeeded, but `git push` still asks for credentials"

This can happen when the dry-run succeeds because it's testing a different remote or because the helper caches credentials temporarily. But more likely, the user is in a different repo or the remote URL changed.

**Fix:**

- Check `git remote -v` in the repo you're actually pushing from.
- Confirm you're in the same directory where the dry-run succeeded.

---

## Appendix A: Why HTTPs Authentication is so Obtuse (The History)

Before 2021, GitHub allowed password authentication over HTTPS.

You typed your GitHub password into the terminal. Git handed it to GitHub. Done.

The only credential helper you needed was your own memory.

In 2021, GitHub killed password auth and forced everyone onto Personal Access Tokens (PATs).

PATs are long, random, unmemorable strings. You cannot type them from memory. You need somewhere to store them securely. And you need a way to obtain them in the first place (browser flow with 2FA).

This created the current landscape:
- Dumb helpers store tokens you manually obtain.
- Smart helpers (GCM) automate token acquisition and storage.
- External tools can also obtain tokens and write them to your helper.

The ecosystem fragmented. Older guides reference older approaches. Newer guides assume newer tooling. The underlying concepts never changed, but the documentation obscures them.

---

## Appendix B: Nuance: Git Has No Default Credential Helper

Git itself does **not** set a default credential helper.

When you install Git, `credential.helper` is unset unless:

1. The Git installer sets it for you (common with official installers on macOS and Windows).
2. You or a system administrator set it manually.
3. A Linux distribution package ships with a default config that sets it.

So if it *feels* like there is a default, it is because the installer made a choice on your behalf. Not because Git has one built in.

If no helper is set, Git prompts you in the terminal every time and saves nothing.

**This is the source of much confusion.** Two identical-looking machines may behave differently because one installer set a helper and the other did not.

---

## Appendix C: The `cache` Helper and Why the "Absurd Integer" Trick Fails

Many old guides will explain to use cache helper with a huge interger timeout amount. It will work for a while. Then it will fail.

**Reason**: The `cache` helper stores credentials in RAM via a daemon process. It is the only helper that is fundamentally ephemeral.

When you set:

```bash
git config --global credential.helper 'cache --timeout=999999999'
```

You are telling Git: "Store credentials in RAM for 999999999 seconds."

The problem is that the daemon itself is not persistent. It dies when:
- The machine reboots.
- The machine sleeps.
- The OS garbage collects stale processes.

When the daemon dies, the RAM is cleared. The credential is gone. The timer only runs as long as the daemon lives.

This is why you can set an absurd timeout and still be prompted a week later. The timer is not the issue. The daemon's lifetime is.

`cache` is only appropriate for temporary convenience on shared machines where you do not want credentials persisted. For anything permanent, use one of the other helpers.

---

## Appendix D: Useful Commands

### Check if your repo is set up for SSH or HTTPS

```bash
git remote -v
```

- If it starts with `git@github.com:`, you are on SSH.
- If it starts with `https://github.com/`, you are on HTTPS.

### Switch from SSH to HTTPS

```bash
git remote set-url origin https://github.com/USERNAME/REPO.git
```

### Switch from HTTPS to SSH

```bash
git remote set-url origin git@github.com:USERNAME/REPO.git
```

### Check current credential helper (relevant for HTTPs)

```bash
git config --global credential.helper
```

### Check which file set the credential helper (relevant for HTTPs)

```bash
git config --show-origin --get credential.helper
```

### Set a credential helper (relevant for HTTPs)

```bash
git config --global credential.helper manager        # GCM (modern name)
git config --global credential.helper manager-core   # GCM (older alias, same helper)
git config --global credential.helper osxkeychain   # macOS
git config --global credential.helper wincred       # Windows
git config --global credential.helper store         # plaintext
git config --global credential.helper cache         # RAM
```

### Unset a credential helper (relevant for HTTPs)

```bash
git config --global --unset credential.helper
```

### Test what credentials Git would currently send (relevant for HTTPs)

```bash
printf "protocol=https\nhost=github.com\n" | git credential fill
```

This prints the username and password Git has stored for github.com. It does not make a network request. It just reads from the configured helper. If no credentials are stored, this command will prompt you for them (interactive mode).

### Clear stored credentials (relevant for HTTPs)

- **macOS (osxkeychain):** Keychain Access → Search for `github.com` → Delete the entry.
- **Windows (wincred):** Credential Manager → Windows Credentials → Find `git:https://github.com` → Remove.
- **Linux (libsecret):** Use `secret-tool` or your desktop environment's keyring GUI.
- **Plaintext (store):** Edit or delete `~/.git-credentials`.
- **GCM:** Delete the entry from the OS keychain (same as above). GCM reads from there.

### Force re-authentication (relevant for HTTPs)

Delete the stored credential (as above), then push. Git will prompt again.
