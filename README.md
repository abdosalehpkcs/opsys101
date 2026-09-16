# SSH: Guide to Fast and Secure Remote Access

> **Note:** In this guide, you’ll learn how to set up and use SSH keys to make remote access to your server.

Have you ever wondered if there’s another way to access your GitHub repo? Or what other authentication methods are out there? Maybe you’ve felt frustrated when a system refuses to recognize you, asking those so-called credentials before letting you in. If you already know the answer, this guide isn’t for you. But if not, carry on, good student! 

> **Student machines (2025/26 setup):** Each student now has their **own virtual machine**. On your **first login you will be asked to set a new password** — once you do, the session **disconnects**, so you simply **reconnect**. Students **no longer have `sudo`**, so the user-management and root-hardening steps in this guide are **optional**.

## Table of Contents

- [What Are SSH Keys?](#what-are-ssh-keys)
- [🔑 How to Generate SSH Keys](#-how-to-generate-ssh-keys)
- [How to Login using SSH](#how-to-login-using-ssh)
- [Configuring SSH for Key Authentication](#configuring-ssh-for-key-authentication)
- [SSH Config Setup Guide (Optional)](#ssh-config-setup-guide-optional)
- [Optional: User Management and Root Access](#optional-user-management-and-root-access)

## What Are SSH Keys?

**Secure Shell (SSH)** is a cryptographic network protocol used for securely operating network services over an unsecured network. it relies on public-key cryptography (see diagram below).. 

![Asymmetric Encryption](image.png)

Think of your public key as a padlock that you can freely share with any server you want to access. Your private key functions like the unique key that unlocks those padlocks. During connection, the remote system verifies that your private key corresponds to its stored public key. If the keys match, you gain immediate access without password authentication.


The public key cannot be used to figure out your private key because it’s generated through a one-way mathematical function which makes the system highly secure. Your private key must remain secret and should stay on your computer—never share it with anyone.

![One Way Math Function](image-1.png)

SSH keys are widely supported and work with almost any system that uses SSH, including Linux servers, cloud platforms, and code hosting services like GitHub and GitLab.

[⬆ Back to top](#table-of-contents)

## 🔑 How to Generate SSH Keys

### Linux/MacOS 🍺

**ED25519 (Recommended):**
```bash
ssh-keygen -t ed25519 -b 4096 -f ~/.ssh/key_name -C "note-$(date +%Y%m%d)" -N "your password"
```

**RSA (Alternative):**
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/key_name -C "note-$(date +%Y%m%d)" -N "your password"
```

**Flag Explanations:**
- `-t`: Specifies the key algorithm (ed25519 or rsa)
- `-b 4096`: Sets key size to 4096 bits for maximum security
- `-f ~/.ssh/key_name`: Specifies custom file location and name
- `-C "note-$(date +%Y%m%d)"`: Adds a comment with current date for identification
- `-N "your password"`: Sets the passphrase directly (replace with your actual password)

**About the Algorithms:**
- **ED25519**: Modern, faster, and more secure algorithm. Recommended for new keys.
- **RSA**: Widely compatible with older systems, battle-tested and reliable.

These commands create highly secure keys with 4096-bit encryption and automatically add a date-based comment for identification. The **passphrase** adds an extra layer of security, even if someone gets your private key file, they can't use it without knowing this phrase.

### Windows 🪟 🤮

> **Note:** On Windows (with OpenSSH or Git Bash) → they also go under C:\Users\YourName\.ssh\

Windows offers several approaches for SSH key generation, each with its own advantages:
- **Git Bash (Recommended for simplicity):**
If Git for Windows is installed, Git Bash provides a Unix-like terminal experience. Simply open Git Bash and execute the same ssh-keygen commands shown in the Linux/MacOS section above.

- **Windows Subsystem for Linux (WSL):**
For developers who prefer authentic Linux environments, WSL delivers native Linux functionality within Windows. All SSH commands function identically to standard Linux systems, making this ideal for cross-platform development workflows.

- **PuTTYgen (GUI Alternative):**
Users who favor graphical interfaces can utilize PuTTYgen, which provides point-and-click SSH key generation with visual configuration options.

![alt text](image-2.png)

[⬆ Back to top](#table-of-contents)

## How to Login using SSH

```bash
ssh root@hiveos.fiit.stuba.sk -p <port> -i <key>
```

Parameter details:

- port: Your assigned port number (provided in email)
- key: Path to your private key file (provided in email)


[⬆ Back to top](#table-of-contents)

## Configuring SSH for Key Authentication

Key authentication lets you log in with your **private key** instead of a password. For this to work, your **public key** must be stored on the server, inside the target user's `~/.ssh/authorized_keys` file.

**Option A — Copy your public key automatically (easiest).** Run this on your **local** machine:
```bash
ssh-copy-id -i ~/.ssh/key_name.pub -p <port> <user>@hiveos.fiit.stuba.sk
```

**Option B — Add the public key manually.** On the **server**, as the target user:
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "ssh-ed25519 AAAA... your-public-key" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

**Test key-based login** from your local machine:
```bash
ssh -i ~/.ssh/key_name -p <port> <user>@hiveos.fiit.stuba.sk
```

Once key login works, you can optionally **turn off password login** to enforce key-only access. On the server, open the SSH server configuration file (you are `root`, so no `sudo` is needed):
```bash
vim /etc/ssh/sshd_config
```

Set these options:
```bash
PasswordAuthentication no
PubkeyAuthentication yes
```

Apply the changes by restarting the SSH service:
```bash
service ssh restart
```

> **Careful:** Confirm that key login already works **before** you disable passwords, otherwise you may lock yourself out.

[⬆ Back to top](#table-of-contents)

## SSH Config Setup Guide (Optional)

### Template

```
Host <alias>
    HostName <hostname>
    Port <port>
    User <username>
    IdentityFile <path-to-private-key>
```

### Configuration Fields

- **Host**: An alias/shortcut name you choose for easy connection
- **HostName**: The actual server address (domain or IP)
- **Port**: SSH port number (default is 22 if not specified)
- **User**: Username for SSH login
- **IdentityFile**: Path to your private SSH key file

---

### Setup Instructions

#### Linux/macOS

1. **Open your SSH config file:**
   ```bash
   nano ~/.ssh/config
   ```
   Or use your preferred editor (vim, gedit, etc.)

2. **Add your configuration:**
   ```
   Host sujohn
       HostName hiveos.fiit.stuba.sk
       Port 9999
       User root
       IdentityFile ~/.ssh/john.pem
   ```

3. **Set proper permissions:**
   ```bash
   chmod 600 ~/.ssh/config
   chmod 600 ~/.ssh/john.pem
   ```

4. **Connect using the alias:**
   ```bash
   ssh sujohn
   ```

#### Windows

1. **Create the .ssh directory (if it doesn't exist):**
   ```powershell
   mkdir $HOME\.ssh
   ```

2. **Open the config file in Notepad:**
   ```powershell
   notepad $HOME\.ssh\config
   ```

3. **Add your configuration:**
   ```
   Host sujohn
       HostName hiveos.fiit.stuba.sk
       Port 8004
       User root
       IdentityFile C:/Users/USERNAME/.ssh/john.pem
   ```

5. **Connect using the alias:**
   ```powershell
   ssh sujohn
   ```

---

### Additional Examples

#### Example 1: Multiple servers with different keys
```
Host webserver
    HostName web.example.com
    User deploy
    IdentityFile ~/.ssh/webserver_key

Host database
    HostName db.example.com
    Port 2222
    User admin
    IdentityFile ~/.ssh/db_key
```

#### Example 2: Using jump host/bastion
```
Host production
    HostName 10.0.1.100
    User ubuntu
    IdentityFile ~/.ssh/prod_key
    ProxyJump bastion

Host bastion
    HostName bastion.example.com
    User jumpuser
    IdentityFile ~/.ssh/bastion_key
```

#### Example 3: GitHub configuration
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key
```

---

### Best Practices

✅ **DO:**
- Use descriptive Host aliases that are easy to remember
- Keep one configuration per server
- Set restrictive permissions on your private keys (600)
- Store keys in the `.ssh` directory
- Use different keys for different servers/purposes
- Comment your config with `#` for clarity

❌ **DON'T:**
- Share your private key files
- Use overly permissive file permissions
- Commit SSH keys to version control
- Use the same key for everything - Creating Key pairs is for free

[⬆ Back to top](#table-of-contents)

## Optional: User Management and Root Access

> **Optional — not required for the course.** These steps show how to create a personal user account, grant it `sudo` rights, and disable direct root login over SSH. All commands are run **as `root`** on the machine.

### 1. Create a New User

Create a personal account and set its password when prompted:
```bash
adduser bob
```

On systems without `adduser`, use the lower-level tool and set the password separately:
```bash
useradd -m -s /bin/bash bob
passwd bob
```

### 2. Grant sudo Permissions

Add the user to the group that grants administrative rights (`sudo` on Debian/Ubuntu, `wheel` on RHEL/Fedora):
```bash
usermod -aG sudo bob
```

Verify the new privileges by switching to the user and running a test command:
```bash
su - bob
sudo whoami   # should print: root
```

### 3. Disable Root Login over SSH

Once a normal user with `sudo` exists, direct root login over SSH can be turned off so everyone logs in with a personal account. Open the SSH server configuration:
```bash
vim /etc/ssh/sshd_config
```

Change the root-login option:
```bash
# From:
#PermitRootLogin yes

# To:
PermitRootLogin no
```

Restart the SSH service to apply the change:
```bash
service ssh restart
```

> **Careful:** Keep an active session open and confirm you can log in as the new `sudo` user **before** you close it, so you do not lock yourself out.

[⬆ Back to top](#table-of-contents)



