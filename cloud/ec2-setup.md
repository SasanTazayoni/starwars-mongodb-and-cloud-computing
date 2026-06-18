# Setting Up an EC2 Instance on AWS

Amazon EC2 (Elastic Compute Cloud) lets you rent virtual servers in the cloud. This guide walks through launching an Ubuntu instance and serving a web page via Nginx.

> **What does "virtual server in the cloud" actually mean?**
> When you launch an EC2 instance, AWS allocates a portion of a physical computer sitting in one of their data centres — large warehouse-scale buildings filled with racks of servers, power systems, and networking equipment. That physical machine runs virtualisation software that carves it into multiple isolated virtual machines, each behaving like a standalone computer. Your EC2 instance is one of those virtual machines.
>
> Crucially, this computer is **remote** — it exists in an AWS data centre, not in front of you. You cannot plug in a monitor or keyboard. The only way to access it is over the internet, which is what SSH is for.

---

## 1. Sign In

Sign in to the AWS Management Console with your credentials.

![Sign In](../images/sign-in.png)

---

## 2. Select a Region

Once signed in, select the appropriate regional server from the top-right dropdown. Choose the region closest to your users for the best performance.

![Choose Server](../images/choose-server.png)

---

## 3. Create a Key Pair

A key pair is used to securely SSH into your instance. For a deeper explanation of how SSH keys work, see the [DataCamp SSH Keys tutorial](https://www.datacamp.com/tutorial/ssh-keys). In the left-hand menu, navigate to **Network & Security → Key Pairs**.

![Key Pairs Menu](../images/key-pairs-menu.png)

Click **Create Key Pair**.

![Create Key Pair Button](../images/create-key-pair-button.png)

Give the key pair a name and leave the default settings, then click **Create**.

![Key Pair Settings](../images/key-pair-settings.png)

![Created Key Pair](../images/created-key-pair.png)

The `.pem` file will download automatically. Move it to your `.ssh` folder:

```
C:\Users\[your_username]\.ssh\
```

If the `.ssh` folder does not exist, create it manually, then move the `.pem` file into it.

---

## 4. Launch an Instance

In the left-hand menu, navigate to **Instances**.

![New Instance Menu](../images/new-instance-menu.png)

Click **Launch Instances**.

![Launch New Instance](../images/launch-new-instance.png)

Give your server a name.

![Server Name](../images/server-name.png)

Scroll down to **Application and OS Images**, select **Ubuntu**, then change the version to **24.04** from the dropdown and confirm when prompted.

![Ubuntu](../images/ubuntu.png)

![Confirm Changes](../images/confirm-changes.png)

Scroll down to **Instance Type** and select **t3.micro**.

---

## 5. Configure Network Settings

Scroll down to **Network Settings** and click **Edit**. Give the security group a descriptive name.

![Edit Network Settings](../images/edit-network-settings.png)

![Security Group Name](../images/security-group-name.png)

Click **Add Security Group Rule** and configure it as follows:

- **Port range:** 80
- **Source:** 0.0.0.0/0 *(allows public HTTP access — for this project only)*

![Security Group Rules](../images/security-group-rules.png)

> **What is a security group?**
> A security group is a virtual firewall that controls what traffic is allowed in and out of your EC2 instance. By default, all inbound traffic is blocked — you have to explicitly open ports.
>
> **What is port 80?**
> Port 80 is the standard port for HTTP (unencrypted web traffic). When you visit `http://some-ip-address` in a browser, your browser sends the request to port 80 on that server.
>
> **What does `0.0.0.0/0` mean?**
> This is CIDR notation meaning "any IP address anywhere on the internet". Combined with port 80, this rule says: *allow any browser from anywhere to make an HTTP request to this server*. For a production app you would typically restrict this further, but for this project it is fine.
>
> The existing **SSH rule (port 22)** was added automatically when you created the instance. It allows you to connect to the server remotely from your terminal using your key pair.

---

## 6. Storage and Advanced Details

Scroll down to **Configure Storage** — leave the defaults as they are.

Scroll down to **Advanced Details** — leave everything blank.

---

## 7. Select Key Pair and Launch

Scroll down to **Key Pair (login)** and select the key pair you created earlier.

![Appropriate Key Pair](../images/appropriate-key-pair.png)

Review the summary on the right, then click **Launch Instance**.

![Preview](../images/preview.png)

Once created, click the link in the success message to go to the instance dashboard.

![Create Instance Success](../images/create-instance-success.png)

---

## 8. Connect via SSH

On the instance dashboard, click the **Connect** button at the top.

![Dashboard](../images/dashboard.png)

This opens a connection instructions page. Switch to the **SSH client** tab and keep it open for reference.

![SSH Instructions](../images/ssh-instructions.png)

Open **Git Bash** (Windows) and navigate to your `.ssh` folder:

```bash
cd C:/Users/[your_username]/.ssh
```

Confirm your `.pem` file is present:

```bash
ls -a
```

Set the correct permissions on your key file:

```bash
chmod 400 "your-key-pair-name.pem"
```

> **Why `chmod 400`?**
> `chmod` changes file permissions. `400` means the file is readable only by you, and no one else can touch it. SSH is strict about this — it will refuse to use a private key that is readable by other users because a world-readable key is considered insecure. You only need to run this once.

Then connect to your instance using the SSH command shown on the instructions page:

```bash
ssh -i "your-key-pair-name.pem" ubuntu@<your-public-ip>
```

> **Breaking down the SSH command:**
> - `ssh` — the Secure Shell program, which opens an encrypted remote terminal session.
> - `-i "your-key-pair-name.pem"` — tells SSH which private key to use for authentication. The server holds the matching public key (AWS put it there when you launched the instance), so only someone with this `.pem` file can log in.
> - `ubuntu` — the default username on Ubuntu EC2 instances.
> - `@<your-public-ip>` — the IP address of your server.

> **How does SSH key authentication actually work?**
> SSH key pairs use **asymmetric cryptography** — a mathematical system where two keys are linked: anything encrypted with one can only be decrypted with the other.
>
> - Your `.pem` file is the **private key** — it never leaves your machine.
> - When you launched the instance, AWS placed the corresponding **public key** on the server (in a file called `~/.ssh/authorized_keys`). Think of the public key as a padlock and the private key as the only key that opens it.
>
> When you run `ssh -i "your-key.pem" ubuntu@<ip>`, here is what happens under the hood:
> 1. Your machine and the server negotiate a shared encryption algorithm.
> 2. The server generates a random challenge and encrypts it with your public key — only your private key can decrypt it.
> 3. Your SSH client decrypts the challenge and sends back a proof derived from it.
> 4. The server verifies the proof. If it matches, authentication succeeds and an encrypted session is opened.
>
> No password is ever transmitted. Someone intercepting the connection cannot log in because they do not have your private key.

When prompted with a host authenticity warning, type `yes` and press Enter to authorise the login.

> This warning appears the first time you connect. SSH is telling you it has never seen this server before and is asking you to confirm you trust it. Once you say yes, the server's fingerprint is saved locally so you are not prompted again.

Verify the connection with:

```bash
whoami
```

You should see `ubuntu` printed in the console.

---

## Generating SSH Keys Locally

In the AWS workflow above, AWS generates the key pair for you and you download the `.pem` file. In other contexts — connecting to a non-AWS server, setting up GitHub access, or managing your own infrastructure — you generate the key pair yourself using `ssh-keygen`.

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

> **Breaking down the command:**
> - `ssh-keygen` — the tool that generates a new key pair.
> - `-t ed25519` — specifies the key type. Ed25519 is the modern recommended algorithm: faster and more secure than the older RSA default. Use RSA (`-t rsa -b 4096`) only if the remote system does not support Ed25519.
> - `-C "your_email@example.com"` — adds a comment to the public key, typically your email, to help identify which key belongs to whom when you have many keys.

When prompted, choose a save location (default is `~/.ssh/id_ed25519`) and optionally enter a passphrase to encrypt the private key.

This produces two files:

| File | Purpose |
|---|---|
| `~/.ssh/id_ed25519` | Your **private key** — never share this |
| `~/.ssh/id_ed25519.pub` | Your **public key** — safe to share with servers |

---

## Adding a Public Key to a Remote Server

Once you have a key pair, you need to place your public key on the remote server so it can verify your identity.

### Using `ssh-copy-id` (Linux/macOS)

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@remote-server-ip
```

This appends your public key to `~/.ssh/authorized_keys` on the remote server automatically. You will be prompted for the user's password once — after that, key-based login replaces it.

### Manually (Windows / no `ssh-copy-id`)

Copy the contents of your public key file:

```bash
cat ~/.ssh/id_ed25519.pub
```

Then on the remote server, append it to the `authorized_keys` file:

```bash
mkdir -p ~/.ssh
echo "your-public-key-contents" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

> **Why these permissions?**
> SSH is strict: if `authorized_keys` is writable by anyone other than the owner, SSH refuses to use it as a security measure. `chmod 600` (owner read/write only) and `chmod 700` (owner full access) satisfy this requirement.

---

## SSH Agent

Typing a passphrase every time you SSH somewhere is tedious. The **SSH agent** is a background process that holds your decrypted private key in memory for the duration of your session, so you only enter the passphrase once.

Start the agent and add your key:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

You will be prompted for your passphrase once. After that, any SSH or SCP command in that session uses the key automatically.

> On macOS, add `--apple-use-keychain` to persist the passphrase across reboots:
> ```bash
> ssh-add --apple-use-keychain ~/.ssh/id_ed25519
> ```

---

## Using SSH Keys with GitHub / GitLab

SSH keys are the recommended way to authenticate with GitHub and GitLab instead of HTTPS passwords or personal access tokens.

### Add your public key to GitHub

1. Copy your public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

2. Go to **GitHub → Settings → SSH and GPG keys → New SSH key**
3. Paste the key, give it a descriptive title (e.g. `Work Laptop`), and click **Add SSH key**

### Test the connection

```bash
ssh -T git@github.com
```

A successful response looks like:

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

### Clone repositories over SSH

```bash
git clone git@github.com:username/repo-name.git
```

Using SSH instead of HTTPS means you never need to enter a password or manage personal access tokens for git operations.

---

## Managing SSH Keys

### List your existing keys

```bash
ls -la ~/.ssh
```

Files ending in `.pub` are public keys. Files without `.pub` are the corresponding private keys.

### View a public key's fingerprint

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

Fingerprints are a shorter representation of the key — useful for verifying you have the right key without comparing the full string.

### Remove a key from a remote server

Open `~/.ssh/authorized_keys` on the remote server and delete the line containing the public key you want to revoke. Each line is one key.

### Troubleshooting common issues

| Problem | Likely cause | Fix |
|---|---|---|
| `Permission denied (publickey)` | Wrong key, wrong user, or key not in `authorized_keys` | Check `-i` flag points to the right key; verify the public key is in `~/.ssh/authorized_keys` on the server |
| `WARNING: UNPROTECTED PRIVATE KEY FILE!` | Key file permissions too open | Run `chmod 400 your-key.pem` |
| `Connection refused` | SSH not running or wrong port | Confirm the server is running SSH on port 22; check firewall/security group rules |
| `Host key verification failed` | Server fingerprint changed (re-provisioned server) | Remove the old entry: `ssh-keygen -R <server-ip>` |

---

## SSH Key Best Practices

- **Use a passphrase.** An unencrypted private key is a credential that can be used immediately if stolen. A passphrase adds an essential second layer of protection.
- **Use one key per device, not per service.** If a device is compromised, you revoke only that device's key without affecting others.
- **Never share your private key.** The public key is designed to be shared; the private key is not. If you need someone else to access a server, add their public key to `authorized_keys` instead.
- **Rotate keys regularly.** Generate a new key pair and update `authorized_keys` on all servers periodically, especially after a team member leaves.
- **Disable password authentication on servers you control.** Once key-based auth is confirmed working, edit `/etc/ssh/sshd_config` and set `PasswordAuthentication no` to prevent brute-force attacks.
- **Use Ed25519 over RSA.** Ed25519 keys are shorter, faster, and considered more secure than RSA-2048.

---

## 9. Install and Verify Nginx

> **What is Nginx?**
> Nginx (pronounced "engine-x") is a web server — a program that runs on your EC2 instance and listens for incoming HTTP requests. When a browser navigates to your server's IP address, it sends an HTTP request to port 80. Nginx receives that request and sends back a response (an HTML page, a file, a forwarded API call, etc.).
>
> Nginx is extremely common in production because it is fast, handles many simultaneous connections efficiently, and can also act as a **reverse proxy** (sitting in front of an app like a Node.js server and forwarding requests to it).

You can install Nginx either by running the commands manually, or by using the provided script which does the same thing in one step.

---

### Option A — Run the script

From your local machine, open Git Bash and navigate to the root of this project:

```bash
cd /c/Users/[your_username]/path/to/this/project
```

Copy the script to your EC2 instance using `scp` (Secure Copy):

```bash
scp -i "C:/Users/[your_username]/.ssh/your-key-pair-name.pem" scripts/deploy-nginx.sh ubuntu@<your-public-ip>:~/
```

> **Breaking down the `scp` command:**
> - `scp` — works just like `ssh` but transfers files instead of opening a terminal session.
> - `-i "..."` — path to your private key, same as with `ssh`.
> - `scripts/deploy-nginx.sh` — the local file to upload (relative to your current directory).
> - `ubuntu@<your-public-ip>:~/` — the destination: the `ubuntu` user's home directory (`~/`) on the remote instance.

SSH into your instance, then make the script executable and run it:

```bash
ssh -i "C:/Users/[your_username]/.ssh/your-key-pair-name.pem" ubuntu@<your-public-ip>
chmod +x deploy-nginx.sh   # gives the file permission to be executed as a program
./deploy-nginx.sh          # runs the script
```

> **Why `chmod +x`?** Files uploaded via `scp` do not automatically have execute permission. `+x` adds it. Without this step, the shell would refuse to run the file.

---

### Option B — Run the commands manually

If you prefer not to copy the script, run the following commands directly in your SSH session:

```bash
sudo apt update -y       # refreshes the package list from Ubuntu's repositories
sudo apt upgrade -y      # upgrades all installed packages to their latest versions
sudo apt install nginx -y  # installs the Nginx web server
```

---

Then, regardless of which option you chose, confirm Nginx is running:

```bash
sudo systemctl status nginx
```

> **What each command does:**
> - `sudo` — runs the command as a superuser (administrator). Most system-level changes require this.
> - `apt update` — syncs your local list of available packages with Ubuntu's remote repositories. This does not install anything — it just updates what your system knows is available.
> - `apt upgrade` — installs the latest versions of all already-installed packages. Good practice before adding new software.
> - `apt install nginx` — downloads and installs Nginx. After installation, Ubuntu automatically starts it and configures it to start on every reboot.
> - `systemctl status nginx` — queries **systemd** (Ubuntu's service manager) for the current status of Nginx. You want to see `active (running)`.
>
> **How the full picture works:**
> 1. Your browser sends an HTTP request to `http://<your-public-ip>` (port 80).
> 2. The EC2 security group rule you created allows that traffic through.
> 3. The request reaches the instance and is received by Nginx.
> 4. Nginx serves back a response — at this point, the default welcome page.
>
> Later, you would configure Nginx to serve your actual application instead of the default page (by editing files in `/etc/nginx/`), or set it up as a reverse proxy that forwards requests to a Node/Python/other backend running on a different port.

You should see the Nginx service reported as **active (running)**.

![Running](../images/running.png)

Return to the instance dashboard and copy the **Public IPv4 address**. Paste it into your browser — make sure the URL starts with `http://` not `https://` (some browsers add the `s` automatically).

> If your browser silently upgrades to `https://`, the request will fail because you have not configured an SSL certificate. Either force `http://` in the address bar, or temporarily try a different browser.

You should see the default Nginx welcome page, confirming the server is live.

![Nginx](../images/nginx.png)
