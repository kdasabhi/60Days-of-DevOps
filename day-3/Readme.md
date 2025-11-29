# Comprehensive Git & Version Control Guide

## Table of Contents

1. [Introduction to Git](#1-introduction-to-git)
2. [Version Control & Git Basics](#2-version-control--git-basics)
3. [Essential Git Concepts & Commands](#3-essential-git-concepts--commands)
4. [Git Branch Management](#4-git-branch-management)
5. [Git Diff](#5-git-diff)
6. [Git Merge & Rebase](#6-git-merge--rebase)
7. [Git Stash & Pop](#7-git-stash--pop)
8. [Git Cherry-Pick](#8-git-cherry-pick)
9. [Git Tags](#9-git-tags)
10. [Git Squash](#10-git-squash)
11. [Merge Conflicts](#11-merge-conflicts)
12. [Git Revert](#12-git-revert)
13. [Git Reset](#13-git-reset)
14. [Git Errors & Resolutions](#14-git-errors--resolutions)
15. [Corporate Branching Strategy](#15-corporate-branching-strategy)

---

## 1. Introduction to Git

### What is Git and Why Use Version Control?

Git is a Version Control System (VCS) that solves critical problems in software development:

**Problem Statement:**
- A project has 15 developers working simultaneously
- Need a system where multiple people can work on application source code
- All team members should collaborate effectively
- Different versions of source code need proper versioning
- One person's work should not affect others' work
- Managers need to track progress

**Solution:** Git provides a distributed VCS where we can maintain multiple versions of source code while solving all these collaboration problems.

### Key Differences: Git, GitHub, GitLab, and Bitbucket

- **Git**: The actual Version Control System
- **GitHub/GitLab/Bitbucket**: Hosting services built on top of Git that help manage code and provide storage to keep source code

These platforms provide additional features beyond basic Git functionality, including:
- Remote repository hosting
- Collaboration tools
- Issue tracking
- Code review features

### Installation and Initial Setup

**Basic Git Configuration:**
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**Authentication:**
- Tokens are used to access repositories
- Tokens can be scoped for different purposes (read, write, delete)
- Different access levels can be assigned to different tokens
- Windows Credential Manager stores authentication details

---

## 2. Version Control & Git Basics

### Types of Version Control Systems

**Centralized vs. Distributed:**
- **Centralized**: Single central repository, all developers connect to it
- **Distributed**: Each developer has a complete copy of the repository (Git follows this approach)

### Creating a New Repository

**Local Repository:**
```bash
git init
```

**Remote Repository:**
```bash
git clone <repository-url>
```

### Understanding Core Git Areas

**Three Main Areas:**
1. **Working Directory**: Where you create and modify files
2. **Staging Area**: Temporary storage where files are tracked before commit
3. **Repository**: Where committed changes are permanently stored

**The .git Hidden Folder:**
- Contains all information about the repository (not the content itself)
- Required for Git to work properly
- Stores configuration, branch information, and commit history

### Basic Git Workflow

**Standard Workflow:**
```bash
# 1. Clone repository
git clone <repo-url>

# 2. Create/modify file
touch 1.txt

# 3. Add to staging area
git add .

# 4. Commit changes
git commit -m "message"

# 5. Push to remote
git push origin main
```

**File States:**
- **Untracked**: File created but not added to Git
- **Tracked**: File added using `git add` (Git knows about the file)
- **Committed**: Changes saved to repository
- **Pushed**: Changes uploaded to remote repository

---

## 3. Essential Git Concepts & Commands

### Daily Usage Commands

**Cloning a Repository:**
```bash
git clone <repository-url>
```

**Checking Status:**
```bash
git status
```

**Viewing Commit History:**
```bash
git log
git log --oneline  # User-friendly format
```

**Adding Files:**
```bash
git add .              # Add all files
git add filename       # Add specific file
git add -f filename    # Forcefully add file (even if in .gitignore)
```

### Git Pull vs Git Fetch

**Git Fetch:**
- Only tells if there are updates
- Updates the .git folder
- Does not merge changes

**Git Pull:**
- Tells about updates AND merges them
- Updates local repository with latest changes from remote
- Equivalent to `git fetch` + `git merge`

**Example:**
```bash
git fetch origin       # Check for updates
git pull origin main   # Fetch and merge updates
```

### Collaboration Workflow

When developers D1, D2, and D3 clone the repository at different times:
1. They must do `git pull` before pushing new features
2. This fetches new changes that may have been added to the repository
3. Then they can push their own changes

---

## 4. Git Branch Management

### Understanding Branches

Branches allow easy versioning where each branch can contain different code:

**Common Branch Types:**
- **Main**: Contains production-ready code
- **Dev**: Development branch for new features (f1, f2 branches merge here)
- **QA**: Contains code for different types of testing
- **PPD**: Pre-production branch version
- **DR**: Disaster Recovery branch (contains same code as main branch)

### View Branches

```bash
git branch              # View local branches
git branch -a           # View all branches (local + remote)
git branch -r           # View remote branches only
```

### Create and Switch Branches

```bash
git branch branch_name          # Create new branch
git checkout branch_name        # Switch to branch
git checkout -b new_branch      # Create and switch in one command
```

**Important:** When creating a new branch, it inherits all files and code from the existing branch.

### Push Branch to Remote

```bash
git push origin branch_name
```

### Delete Branches

**Delete Local Branch:**
```bash
git branch -d branch_name
```

**Delete Remote Branch:**
```bash
git push origin --delete branch_name
```

### HEAD Pointer

The HEAD pointer shows which branch has the latest push and where you're currently working.

---

## 5. Git Diff

### Compare Branches

```bash
git diff branch1 branch2
```

### Compare Files

```bash
git diff filename
git diff branch1:filename branch2:filename
```

### Compare Commits

```bash
git diff commit1_hash commit2_hash
git diff HEAD~1 HEAD  # Compare last commit with current
```

---

## 6. Git Merge & Rebase

### Git Merge

**Purpose:** Merge changes from one branch into another (typically into main branch)

**Process:**
1. Merge creates a merge commit file that preserves history
2. Shows clear information about how new changes were made
3. History is preserved

**Example:**
```bash
git checkout main
git merge feature_branch
```

**Corporate Practice:**
- Any new change before merging to main branch must be verified by architect
- Companies prefer merge as it provides clear audit trail

### Git Rebase

**Purpose:** Create a clean, linear history without merge commits

**Process:**
1. Reapplies commits on top of another branch
2. No merge commit file is created
3. Results in cleaner history

**Example:**
```bash
git checkout feature_branch
git rebase main
```

**Note:** Branches can only merge to specific branches if they originated from them.

### Which One to Use?

**Use Merge When:**
- You want to preserve complete history
- Working in corporate environment (better for auditing)
- Multiple people are collaborating

**Use Rebase When:**
- You want clean, linear history
- You don't want to show intermediate changes
- Working on personal feature branch

---

## 7. Git Stash & Pop

### Understanding Stash

Git stash temporarily stores current changes so you can work on something else.

**Use Cases:**
- You've added a.txt but realize you need to add b.txt first
- You're doing `git add` but remember you need to switch branches to add other files first

### Basic Stash Commands

**Stash Current Changes:**
```bash
git stash
```

**View Stashed Changes:**
```bash
git stash list
```

**Apply Stashed Changes:**
```bash
git stash pop    # Apply and remove from stash
git stash apply  # Apply but keep in stash
```

### Hands-On Example

**Scenario:** Stash existing file, add new file, then bring back stashed file

```bash
# 1. Stash current changes
git stash

# 2. Work on new file
git add new_file.txt
git commit -m "Added new file"
git push origin main

# 3. Restore stashed changes
git stash pop
git add .
git commit -m "Added stashed file"
git push origin main
```

### Stash Multiple Files

```bash
git stash                    # Stash current changes
# Make more changes
git stash                    # Stash again
git stash list              # View all stashes
git stash pop stash@{1}     # Pop specific stash
```

---

## 8. Git Cherry-Pick

### Understanding Cherry-Pick

Cherry-pick allows you to merge specific commits from one branch to another (instead of merging entire branch).

**Use Case:** You committed to the wrong branch and want to move specific commits to the correct branch.

### Implementing Cherry-Pick

**Process:**
```bash
# 1. Find the commit hash
git log --oneline

# 2. Switch to target branch
git checkout target_branch

# 3. Cherry-pick the commit
git cherry-pick <commit-hash>

# 4. Optional: Remove from original branch
git checkout wrong_branch
git reset --hard HEAD~1
```

**Example Scenario:**
- Accidentally committed to main branch
- Use cherry-pick to move commit to feature branch
- Delete commit from main branch using `git reset --hard`

---

## 9. Git Tags

### Understanding Tags

Tags are used to mark specific points in Git history, typically for releases.

**Types of Tags:**
- **Lightweight**: Simple pointer to a commit
- **Annotated**: Full object with tagger info, date, and message

### Creating Tags

```bash
# Lightweight tag
git tag v1.0

# Annotated tag
git tag -a v1.0 -m "Release version 1.0"

# View all tags
git tag

# Push tag to remote
git push origin v1.0

# Push all tags
git push origin --tags
```

---

## 10. Git Squash

### Understanding Git Squash

Squash combines multiple commits into a single commit for cleaner history.

**Use Case:** You made 5 small commits while developing a feature, but want to merge as 1 commit.

### Combining Commits Using Squash

**Interactive Rebase Method:**
```bash
# Squash last 3 commits
git rebase -i HEAD~3
```

**In the editor:**
- Keep first commit as `pick`
- Change others to `squash` or `s`
- Save and exit
- Edit the combined commit message

**Example:**
```
pick abc123 First commit
squash def456 Second commit
squash ghi789 Third commit
```

---

## 11. Merge Conflicts

### Understanding Merge Conflicts

Conflicts occur when:
- Same file is modified in different branches
- Different developers edit the same lines
- Git cannot automatically determine which changes to keep

### 3 Different Conflict Scenarios

**Scenario 1: Different Lines Modified**
- Developer 1 modifies line 5
- Developer 2 modifies line 10
- Usually auto-merged by Git

**Scenario 2: Same Line Modified**
- Developer 1 changes line 5 to "Hello"
- Developer 2 changes line 5 to "Hi"
- Manual resolution required

**Scenario 3: File Deleted vs Modified**
- Developer 1 deletes file
- Developer 2 modifies same file
- Manual decision required

### Resolving Conflicts

**Step-by-Step Resolution:**
```bash
# 1. Attempt merge
git merge feature_branch

# 2. Git shows conflict
# CONFLICT (content): Merge conflict in filename.txt

# 3. Open file and see conflict markers
<<<<<<< HEAD
Current branch content
=======
Incoming branch content
>>>>>>> feature_branch

# 4. Edit file to resolve
# Remove conflict markers
# Keep desired content

# 5. Complete merge
git add filename.txt
git commit -m "Resolved merge conflict"
git push origin main
```

### Resolving Conflicts Using Git Revert

You can also use `git revert` to undo conflicting commits while maintaining history.

---

## 12. Git Revert

### Understanding Git Revert

Revert creates a new commit that undoes changes from a previous commit, keeping commit history linear.

**Why Use Revert:**
- Keeps commit history intact
- Better for shared branches
- Creates audit trail
- Safer than reset

### Implementing Revert

**Revert Single Commit:**
```bash
git revert <commit-hash>
```

**Revert Range of Commits:**
```bash
git revert <oldest-commit>..<newest-commit>
```

**Example:**
```bash
git log --oneline
# abc123 Bad commit
# def456 Good commit

git revert abc123
# Creates new commit that undoes abc123
```

**Note:** Even though the file is deleted/reverted, it still appears in commit history (which is the advantage for auditing).

---

## 13. Git Reset

### Understanding Git Reset

Reset moves the HEAD pointer and optionally modifies staging area and working directory.

### Three Types of Reset

**1. Soft Reset:**
- Removes file from commit
- Keeps changes in staging area
```bash
git reset --soft HEAD~1
```

**2. Mixed Reset (Default):**
- Removes file from staging area
- Keeps changes in working directory
```bash
git reset --mixed HEAD~1
# or simply
git reset HEAD~1
```

**3. Hard Reset:**
- Permanently deletes changes
- **Warning:** Cannot be recovered easily
```bash
git reset --hard HEAD~1
```

### Reset vs Revert

**Use Reset When:**
- Working on local, unpushed commits
- Want to completely remove commits
- Not sharing branch with others

**Use Revert When:**
- Changes are already pushed
- Need to maintain history
- Working on shared branches
- **Revert is much better than reset in most cases**

---

## 14. Git Errors & Resolutions

### Common Errors and Solutions

#### 1. Buffer Size Error

**Problem:** Large file size exceeds Git buffer during clone/push

**Error Message:**
```
error: RPC failed; HTTP 413 curl 22 The requested URL returned error: 413
```

**Solution:**
```bash
# Temporary fix
git -c http.postBuffer=52428000 push origin main

# Permanent fix
git config --global http.postBuffer=53428000
```

**Note:** Buffer size should be at least equal to file size in bytes.

#### 2. .git Folder Deleted

**Problem:** Accidentally deleted .git folder

**Solution:**
```bash
# Re-initialize Git
git init

# Re-add remote
git remote add origin <repository-url>

# Fetch all branches
git fetch origin

# Reset to match remote
git reset --hard origin/main
```

#### 3. Committed to Wrong Branch

**Solution using Cherry-Pick:**
```bash
# 1. Note the commit hash
git log --oneline

# 2. Switch to correct branch
git checkout correct_branch

# 3. Cherry-pick the commit
git cherry-pick <commit-hash>

# 4. Remove from wrong branch
git checkout wrong_branch
git reset --hard HEAD~1
```

#### 4. Two Directories Pushing Same File

**Problem:** Conflict when two users push changes to same file

**Solution:**
```bash
git pull --rebase origin main
# Resolve conflicts if any
git add .
git rebase --continue
git push origin main
```

#### 5. Recovering Deleted Commits

**Solution using Reflog:**
```bash
# View reflog
git reflog

# Find the commit hash you want to recover
# Reset to that commit
git reset --hard <commit-hash>
```

### Git Fetch to Update Repository

Use `git fetch` to update your local repository with information from remote without merging:
```bash
git fetch origin
git status  # See what's different
```

---

## 15. Corporate Branching Strategy

### Common Branching Strategy #1

**Branch Structure:**
```
main (production)
├── dev (development)
│   ├── feature/f1
│   ├── feature/f2
│   └── feature/f3
├── qa (testing)
├── ppd (pre-production)
└── dr (disaster recovery)
```

**Environment Purposes:**

**Dev Environment:**
- Each developer has their own feature branch (f1, f2, f3)
- Code and test application in local repository
- Push to dev branch after local testing
- Mostly unit testing performed here

**QA Environment:**
- Functional testing performed
- If bug found, bugfix branch is created from QA
- Developer fixes bug and merges back to QA

**PPD (Pre-Production):**
- Identical to production environment
- Load testing with fake user traffic
- Security checks performed
- Final validation before production

**Production (Main):**
- Live environment with real users
- If critical issue occurs, hotfix branch is created
- Bug fixed and merged back to main
- Then deployed

**DR (Disaster Recovery):**
- Contains same code as main branch
- If anything goes wrong in main, traffic redirected to DR
- Requires database backup to connect to DR environment
- Can be costly (multiple clusters, multiple deployments)

### Common Branching Strategy #2 (Release Candidate)

**Branch Structure:**
```
main (production)
├── develop
├── qa
│   ├── feature/f1
│   ├── feature/f2
│   └── release/rc1
│       ├── bugfix/bug1
│       └── bugfix/bug2
```

**Release Candidate Process:**
1. Features f1, f2 added to QA branch and tested
2. Release Candidate (RC1) created
3. If issues found, bugfix branches created
4. Expected to complete fixes within RC1 to RC4 cycle
5. Once stable, merge to main

**Key Difference:** Uses release candidates to batch and stabilize changes before production.

### Environment vs Branch Clarification

**Environment:**
- Where application will run (e.g., K8s cluster, OpenShift)

**Branch:**
- Where application source code is stored
- Code from branch is deployed and runs inside the environment

### Cluster Strategy

**Options:**
1. **Multiple Clusters:** 4 environments = 4 clusters (costly)
2. **Single Cluster with Namespaces:** 1-2 clusters with different namespaces for different environments (cost-effective)

Most companies use limited clusters with multiple namespaces to save costs.

### Deployment Process

**How Deployment Works:**

```
Dev Environment (Complete Testing)
    ↓
Docker Image Created
    ↓
Tag Image for QA: app:qa
    ↓
Deploy to QA Environment
    ↓
Tag Image for PPD: app:ppd
    ↓
Deploy to PPD Environment
    ↓
Tag Image for Prod: app:prod
    ↓
Deploy to Production
```

**Why Track with Different Names?**
- Everything from code to testing done in dev environment
- Fully tested Docker image is created
- Proper process should be followed for audit compliance
- Same tested image deployed across all environments with different tags

### Testing & Pipelines

- Pipelines for automated testing written by QA teams
- Continuous Integration/Continuous Deployment (CI/CD) implemented
- Each environment has its own validation gates

### Best Practices

1. **Feature Branch Workflow:**
   - Create feature branch from dev
   - Develop and test locally
   - Merge to dev after code review
   - Dev merges to QA for testing

2. **Hotfix Protocol:**
   - Create hotfix branch from main
   - Fix critical production issues
   - Merge to main and deploy immediately
   - Backport fix to other branches

3. **Code Review:**
   - Architect/Senior developer must verify changes before main branch merge
   - Pull Request (PR) approval required
   - Automated tests must pass

4. **Version Control:**
   - Use semantic versioning (v1.0.0, v1.1.0, v2.0.0)
   - Tag releases in main branch
   - Document changes in release notes

---

## Additional Tips & Best Practices

### .gitignore

To ignore files from being committed to repository:

```bash
# Create .gitignore file
touch .gitignore

# Add patterns to ignore
echo "*.log" >> .gitignore
echo "node_modules/" >> .gitignore
echo ".env" >> .gitignore

# Force add ignored file if needed
git add -f filename.txt
```

### Useful Commands

```bash
# List files in detail
ls -lart

# View detailed commit history
git log --graph --oneline --all

# View who changed what
git blame filename

# Show changes in specific commit
git show <commit-hash>

# Clean untracked files
git clean -fd
```

### Git Configuration

```bash
# View all configurations
git config --list

# Set user information
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "vim"

# Set default branch name
git config --global init.defaultBranch main
```

---

## Conclusion

This comprehensive guide covers essential Git concepts and workflows used in corporate environments. Master these concepts through practice, and always remember:

- **Commit often** with meaningful messages
- **Pull before push** to avoid conflicts
- **Use branches** for features and fixes
- **Follow team conventions** for branching and commits
- **Test locally** before pushing to shared branches
- **Document your changes** for team collaboration

Git is a powerful tool that enables efficient collaboration and version control. With these concepts and practices, you'll be well-equipped to handle real-world development scenarios.
