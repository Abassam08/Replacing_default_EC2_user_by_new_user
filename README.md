# 🟦 OPS345 Lab 7 — Replace `ec2-user` With `asabra1` (Full Guide)

## ⭐ Overview

These steps show how to remove the default AWS `ec2-user` and replace it with your own user `asabra1`, including:

* SSH key login (no password)
* Root (sudo) privileges
* Passwordless local login
* Secure access only through the router

---

## 🟩 **1. Create the New User**

```bash
sudo adduser asabra1
```

Creates `/home/asabra1` and the new user.

---

## 🟩 **2. Give `asabra1` Full Root Privileges**

Add user to the `wheel` group (Amazon Linux admin group):

```bash
sudo usermod -aG wheel asabra1
```

Now the user can run sudo commands.

---

## 🟩 **3. Remove Local Password (Optional but Recommended)**

Allows passwordless local login (`su asabra1`):

```bash
sudo passwd -d asabra1
```

The user can now rely *only* on SSH keys for authentication.

---

## 🟩 **4. Configure SSH Key for `asabra1`**

Create `.ssh` directory:

```bash
sudo mkdir /home/asabra1/.ssh
```

Copy the router’s authorized key from the default user:

```bash
sudo cp /home/ec2-user/.ssh/authorized_keys /home/asabra1/.ssh/
```

Fix file permissions:

```bash
sudo chown -R asabra1:asabra1 /home/asabra1/.ssh
sudo chmod 700 /home/asabra1/.ssh
sudo chmod 600 /home/asabra1/.ssh/authorized_keys
```

`asabra1` now uses the **exact same .pem key** as the router VM.

---

## 🟩 **5. Enable Key-Only SSH Login**

Edit SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```

Ensure these lines are set to **no**:

```
PasswordAuthentication no
ChallengeResponseAuthentication no
```

Restart SSH:

```bash
sudo systemctl restart sshd
```

This disables password logins completely — keys only.

---

## 🟩 **6. Test SSH Login Through Router**

From your workstation:

```bash
ssh -i ~/Desktop/Keys/SSH/ops345-first-key.pem \
    -p 2212 asabra1@54.87.254.96
```

Verify sudo:

```bash
sudo whoami
```

Should return:

```
root
```

---

## 🟩 **7. Delete Default AWS User (`ec2-user`)**

Do this **ONLY** after confirming that `asabra1` can SSH and use sudo.

```bash
sudo userdel -r ec2-user
```

This removes the user and their home folder.

---

## 🎉 **Done!**

At this point:

* `asabra1` is the **only** user
* Has full root access
* Logs in with **no password**, SSH key only
* Cannot be accessed directly — only through the router (per lab requirements)
* System is secure and ready for the rest of the OPS345 email server setup

  
```mermaid
flowchart TD
    A[Start: Logged in as ec2-user via Router] --> B[Create user asabra1<br/>sudo adduser asabra1]
    B --> C[Give root privileges<br/>sudo usermod -aG wheel asabra1]
    C --> D[Remove password (optional)<br/>sudo passwd -d asabra1]
    D --> E[Create SSH folder<br/>sudo mkdir /home/asabra1/.ssh]
    E --> F[Copy authorized_keys<br/>sudo cp /home/ec2-user/.ssh/authorized_keys /home/asabra1/.ssh/]
    F --> G[Fix permissions<br/>sudo chown -R asabra1:asabra1<br/>chmod 700/600]
    G --> H[Disable password SSH login<br/>Edit sshd_config]
    H --> I[Restart SSH<br/>sudo systemctl restart sshd]
    I --> J[Test SSH via Router<br/>ssh -i key.pem -p 2212 asabra1@routerIP]
    J --> K[Test sudo<br/>sudo whoami → root]
    K --> L{Success?}
    L -->|No| M[Fix SSH or Sudo Issues<br/>Repeat steps]
    L -->|Yes| N[Delete ec2-user<br/>sudo userdel -r ec2-user]
    N --> O[Final System: Only asabra1 with SSH key login]
    O --> P[Done]
```




