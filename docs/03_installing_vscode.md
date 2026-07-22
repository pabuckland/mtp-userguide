# Tutorial 1: Installing Visual Studio Code

## Objective

By the end of this tutorial, you will have:

- Installed Visual Studio Code
- Installed the Remote - SSH extension
- Generated an SSH key
- Added your SSH key to your Digital Research Alliance of Canada (DRAC) account
- Connected to the Trillium cluster

---

## Background

Visual Studio Code (VS Code) is a free code editor that can connect directly to remote computers using SSH.

Throughout this guide, VS Code will be used to:

- Edit files stored on the cluster
- Run Linux commands
- Submit jobs
- Develop and test workflows

Although Windows PowerShell can also be used to connect to DRAC clusters, VS Code provides an easier interface for beginners.

---

## Prerequisites

Before starting, ensure that you have:

- A Digital Research Alliance of Canada (DRAC) account
- Your Alliance username
- A Windows computer
- An internet connection

---

## Step 1 – Install Visual Studio Code

Download VS Code from the official website:

https://code.visualstudio.com/download

Install using the default settings.

---

## Step 2 – Install the Remote - SSH Extension

Open VS Code.

Select the **Extensions** icon from the Activity Bar.

Search for:

Remote - SSH

Install the extension published by Microsoft.

![Remote SSH Extension](../images/remote_ssh_extension.png)

---

## Step 3 – Generate an SSH Key

Open Windows PowerShell.

Run:

```bash
ssh-keygen -t ed25519 -C your_email@example.com
```

Press **Enter** to accept the default file location.

Your SSH keys will normally be saved in:

```text
C:\Users\<YourUsername>\.ssh\
```

The public key is stored in:

```text
id_ed25519.pub
```

Open this file using Notepad and copy its entire contents.

---

## Step 4 – Add the SSH Key to DRAC

Log in to your DRAC account.

Navigate to:

**My Account → SSH Keys**

Paste the contents of your public key into the SSH key field.

Select **Add Key**.

*Insert screenshot here.*

---

## Step 5 – Connect to Trillium

Open VS Code.

Click the **Remote Connection** button in the lower-left corner.

Choose:

**Connect to Host**

Enter:

```bash
ssh your_username@trillium.alliancecan.ca
```

Replace:

- `your_username` with your Alliance username.
- `trillium` with another cluster name if applicable.

---

## Step 6 – Open the Integrated Terminal

After connecting, open the integrated terminal by selecting:

**Terminal → New Terminal**

This terminal is running on the remote Linux system.

You can now execute Linux commands directly on the cluster.

---

## Step 7 – Verify the Connection

Run:

```bash
pwd
```

You should see your home directory.

Next run:

```bash
hostname
```

The output should indicate that you are connected to one of the Trillium login nodes.

Congratulations! You are now connected to the cluster.

---

## Common Issues

### Permission denied (publickey)

Your SSH key may not have been uploaded correctly.

### Could not establish connection

Verify that your username and cluster name are correct.

### Remote - SSH not installed

Ensure the Microsoft Remote - SSH extension is installed.

---

## Next Tutorial

Continue to:

**Tutorial 2 – Connecting to DRAC**
