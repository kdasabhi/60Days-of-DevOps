# User and Group Management in Linux

## Introduction

Linux is a multi-user operating system that allows multiple users to work on the same system simultaneously. Understanding user and group management is crucial for maintaining system security and controlling who can access what on your system.

Think of it like an apartment building: the building manager (root user) has access to everything, while residents (regular users) can only access their own apartments and shared spaces they're permitted to use.

---

## Understanding Users

### Types of Users

**1. Root User (The Superuser)**
- Has complete control over the system
- Can do anything without restrictions
- Home directory: `/root`
- Use with caution! Great power = great responsibility

**2. Regular Users**
- Created by administrators for day-to-day work
- Limited permissions for security
- Each user gets their own home directory: `/home/username`
- Cannot make system-wide changes without permission

---

## Creating and Managing Users

### Creating a New User

```bash
# Create a new user
sudo useradd john

# Set a password for the user
sudo passwd john
```

**What happens behind the scenes?**
- A new home directory is created at `/home/john`
- User information is added to system files
- A unique user ID (UID) is assigned

### Important User Files

**1. `/etc/passwd` - User Account Information**

Each line contains basic user details:
```
john:x:1001:1001:John Doe:/home/john:/bin/bash
```
- `john` - Username
- `x` - Password placeholder (actual password is in /etc/shadow)
- `1001` - User ID (UID)
- `1001` - Primary Group ID (GID)
- `John Doe` - Full name or description
- `/home/john` - Home directory
- `/bin/bash` - Default shell

**2. `/etc/shadow` - Encrypted Passwords**

Contains encrypted passwords and security settings (only root can read this file):
```
john:$6$abcd1234$encrypted_password:18500:0:99999:7:::
```

**3. `/etc/group` - Group Information**

Lists all groups and their members:
```
developers:x:1002:john,alice
```

---

## Understanding Groups

Groups are like teams in an organization. Instead of giving permissions to each person individually, you assign them to a group, and everyone in that group gets the same access rights.

### Creating Groups

```bash
# Create a new group
sudo groupadd developers
```

### Adding Users to Groups

```bash
# Add john to the developers group
sudo usermod -aG developers john

# The -aG flag means:
# -a = append (add to group without removing from others)
# -G = supplementary groups
```

### Viewing Group Membership

```bash
# See which groups a user belongs to
groups john

# Output: john : john developers
```

---

## Essential Commands Reference

### User Management

| Command | Purpose | Example |
|---------|---------|---------|
| `useradd` | Create a new user | `sudo useradd alice` |
| `passwd` | Set/change password | `sudo passwd alice` |
| `usermod` | Modify user account | `sudo usermod -aG engineers alice` |
| `userdel` | Delete a user | `sudo userdel alice` |
| `id` | Display user ID info | `id alice` |

### Group Management

| Command | Purpose | Example |
|---------|---------|---------|
| `groupadd` | Create a new group | `sudo groupadd engineers` |
| `groupdel` | Delete a group | `sudo groupdel engineers` |
| `groups` | Show user's groups | `groups alice` |

---

## File Permissions and Ownership

Every file and directory in Linux has:
- An **owner** (a user)
- A **group** 
- **Permissions** for owner, group, and others

### Understanding Permission Strings

When you run `ls -l`, you'll see something like:
```
-rwxr-xr--  1  john  developers  4096  Nov 28  example.txt
```

Let's break this down:

```
-rwxr-xr--
│└┬┘└┬┘└┬┘
│ │  │  └──── Others permissions (r--)
│ │  └──────── Group permissions (r-x)
│ └──────────── Owner permissions (rwx)
└────────────── File type (- = file, d = directory)
```

**Permission meanings:**
- `r` = Read (view contents)
- `w` = Write (modify contents)
- `x` = Execute (run as program)
- `-` = No permission

### Changing File Ownership

```bash
# Change owner to john, group to developers
sudo chown john:developers filename

# Change only the owner
sudo chown john filename

# Change only the group
sudo chown :developers filename
```

---

## Changing File Permissions

There are two ways to change permissions: **symbolic** and **numeric**.

### Symbolic Mode (Easy to Remember)

**Who:**
- `u` = user (owner)
- `g` = group
- `o` = others
- `a` = all

**What:**
- `+` = add permission
- `-` = remove permission
- `=` = set exact permission

**Examples:**
```bash
# Give owner execute permission
chmod u+x script.sh

# Remove write permission from group
chmod g-w document.txt

# Set read and execute for everyone
chmod a=rx file.txt

# Give owner all permissions, group read only
chmod u=rwx,g=r,o= private.txt
```

### Numeric Mode 

Each permission has a number:
- `r` = 4
- `w` = 2
- `x` = 1

Add them up for each group (owner, group, others):

| Number | Permissions | Calculation |
|--------|-------------|-------------|
| 7 | rwx | 4 + 2 + 1 |
| 6 | rw- | 4 + 2 |
| 5 | r-x | 4 + 1 |
| 4 | r-- | 4 |
| 0 | --- | 0 |

**Examples:**
```bash
# rwxr-xr-x (755)
# Owner: all, Group: read+execute, Others: read+execute
chmod 755 script.sh

# rw-r--r-- (644)
# Owner: read+write, Group: read, Others: read
chmod 644 document.txt

# rwx------ (700)
# Owner: all, Group: none, Others: none
chmod 700 private_script.sh
```

---

## Practical Examples

### Example 1: Setting Up a Developer Account

```bash
# 1. Create user
sudo useradd sarah

# 2. Set password
sudo passwd sarah

# 3. Create a group for developers
sudo groupadd webdev

# 4. Add sarah to the webdev group
sudo usermod -aG webdev sarah

# 5. Verify group membership
groups sarah
# Output: sarah : sarah webdev
```

### Example 2: Creating a Shared Team Workspace

```bash
# 1. Create a shared directory for the team
sudo mkdir /teamspace/frontend

# 2. Change ownership to sarah and webdev group
sudo chown sarah:webdev /teamspace/frontend

# 3. Set permissions: owner and group full access, others no access
sudo chmod 770 /teamspace/frontend

# 4. Verify the setup
ls -ld /teamspace/frontend
# Output: drwxrwx--- 2 sarah webdev 4096 Nov 28 /teamspace/frontend
```

### Example 3: Setting Up a Collaborative Document

```bash
# 1. Create a file
touch team_goals.md

# 2. Set ownership
sudo chown sarah:webdev team_goals.md

# 3. Set permissions (owner and group can edit, others can only read)
chmod 664 team_goals.md

# 4. Check the result
ls -l team_goals.md
# Output: -rw-rw-r-- 1 sarah webdev 0 Nov 28 team_goals.md
```

---

## Common Permission Patterns



| Permissions | Use Case |
|-------------|----------|
| `755` (rwxr-xr-x) | Executable scripts and programs |
| `644` (rw-r--r--) | Regular files (documents, text files) |
| `700` (rwx------) | Private scripts or directories |
| `600` (rw-------) | Private files (config files, SSH keys) |
| `775` (rwxrwxr-x) | Shared directories for team projects |
| `664` (rw-rw-r--) | Shared files for team collaboration |

---

## Quick Tips for Beginners

1. **Always use `sudo` carefully** - It gives you root powers, so double-check your commands!

2. **Use `-aG` with usermod** - The `-a` is important! Without it, you'll remove the user from all other groups.

3. **Test permissions with a regular user** - After setting permissions, log in as a regular user to make sure everything works.

4. **Start with symbolic mode** - It's easier to understand `chmod u+x` than `chmod 755` when you're learning.

5. **Back up before making system changes** - Especially when modifying user accounts or permissions on important files.

---

