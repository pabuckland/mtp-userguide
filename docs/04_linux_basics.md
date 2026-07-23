# Tutorial 4 – Linux Basics

## Overview

DRAC clusters run using Linux operating systems. Unlike a personal computer, most interactions with a cluster are performed through the command line terminal.

This tutorial introduces the basic Linux commands required to navigate files, create folders, and manage data on a DRAC cluster.

By the end of this tutorial, you should understand:

- How Linux organizes files
- How to navigate directories
- How to create and modify files
- How to transfer and manage simulation data

---

# 1. Understanding the Linux File System

Linux organizes files using a hierarchical directory structure.

The top directory is called the root directory:

```bash
/
```

Your DRAC home directory is located below:

```bash
/home/username
```

where `username` is your Alliance username.

When you first connect to a cluster, you normally start inside your home directory.

---

# 2. Checking Your Current Location

Use:

```bash
pwd
```

`pwd` means:

```
print working directory
```

Example:

```bash
pwd
```

Output:

```text
/home/abc123
```

This confirms your current location.

---

# 3. Listing Files and Folders

To view files in your current directory:

```bash
ls
```

Example:

```text
projects
scratch
results
```

For additional information:

```bash
ls -l
```

This displays file permissions, ownership, and file sizes.

---

# 4. Moving Between Directories

Use:

```bash
cd
```

which means:

```
change directory
```

Example:

```bash
cd projects
```

Move back one directory:

```bash
cd ..
```

Return to your home directory:

```bash
cd ~
```

---

# 5. Creating Directories

Create a new folder using:

```bash
mkdir folder_name
```

Example:

```bash
mkdir MTP_simulations
```

This creates a folder for storing MTP calculations.

---

# 6. Creating and Viewing Files

Create a blank file:

```bash
touch file_name
```

Example:

```bash
touch input.mtp
```

View file contents:

```bash
cat file_name
```

Example:

```bash
cat input.mtp
```

---

# 7. Copying and Moving Files

Copy a file:

```bash
cp original destination
```

Example:

```bash
cp input.mtp backup.mtp
```

Move or rename a file:

```bash
mv old_name new_name
```

Example:

```bash
mv test.mtp training.mtp
```

---

# 8. Removing Files

Remove a file:

```bash
rm filename
```

Example:

```bash
rm old_file.txt
```

Be careful: deleted files generally cannot be recovered.

---

## Next Tutorial

The next tutorial introduces High Performance Computing (HPC) concepts and explains how DRAC clusters are structured.
