# SSH: Guide to Fast and Secure Remote Access

> **Note:** In this guide, you’ll learn how to set up and use SSH keys to make remote access to your server.

Have you ever wondered if there’s another way to access your GitHub repo? Or what other authentication methods are out there? Maybe you’ve felt frustrated when a system refuses to recognize you, asking those so-called credentials before letting you in. If you already know the answer, this guide isn’t for you. But if not, carry on, good student! 

## What Are SSH Keys?

**Secure Shell (SSH)** is a cryptographic network protocol used for securely operating network services over an unsecured network. it relies on public-key cryptography (see diagram below).. 

![Asymmetric Encryption](image.png)

Think of your public key as a padlock that you can freely share with any server you want to access. Your private key functions like the unique key that unlocks those padlocks. During connection, the remote system verifies that your private key corresponds to its stored public key. If the keys match, you gain immediate access without password authentication.


The public key cannot be used to figure out your private key because it’s generated through a one-way mathematical function which makes the system highly secure. Your private key must remain secret and should stay on your computer—never share it with anyone.

![One Way Math Function](image-1.png)

SSH keys are widely supported and work with almost any system that uses SSH, including Linux servers, cloud platforms, and code hosting services like GitHub and GitLab.

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


## How to Login using SSH

```bash
ssh root@hiveos.fiit.stuba.sk -p <port> -i <key>
```

Parameter details:

- port: Your assigned port number (provided in email)
- key: Path to your private key file (provided in email)


## Configuring SSH for key authentication

To edit the SSH server configuration, open the main config file, which is located at: 

```bash
vim /etc/ssh/sshd_config
```

In this file, you can disable direct root login via SSH. This forces users to log in with their personal accounts and use sudo for administrative tasks.

**Edit the SSH configuration file:**
```bash
sudo vim /etc/ssh/sshd_config
```

**Find and modify this line:**
```bash
# Change this line from:
#PermitRootLogin yes

# To:
PermitRootLogin no
```

**Restart the SSH service to apply changes:**
```bash
sudo systemctl restart sshd
```


# Lab Assignment: SSH Key Setup and User Management

**Step 1: Find Your Group**
Students sharing the same port number form a group. Identify your group members and the assigned port or from the mailing list.

**Step 2: Initial Root Access**
Try to log in to your root user on the assigned virtual machine. Once successful, **STOP** - do not proceed further yet, wait for your classmates in your group to reach this point.

**Step 3: Designate Group Administrator**
Collaborate with your group to select one person who will act as the root administrator. This person will be responsible for helping and managing the setup process for the entire group.

**Step 4: User Account Creation**
From the root account, each group member should create his/her/their individual user account and 
remember THE PASSWORD, then Add the user to sudo group.

**Step 5: Verify User Login And sudo command access**
Each student should then attempt to log in using their newly created personal account.

**Step 6: Adminstrator Only**
The designated administrator should change the root password and disable SSH access.
