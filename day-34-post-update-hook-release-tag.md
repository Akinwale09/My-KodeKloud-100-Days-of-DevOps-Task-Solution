# Day 34 : Git Hook

The Nautilus application development team was working on a git repository `/opt/cluster.git` which is cloned under `/usr/src/kodekloudrepos` directory present on `Storage server` in `Stratos DC`. The team want to setup a hook on this repository, please find below more details:

1. Merge the `feature` branch into the `master` branch, but before pushing your changes complete below point.

2. Create a `post-update` hook in this git repository so that whenever any changes are pushed to the `master` branch, it creates a release tag with name `release-2023-06-15`, where `2023-06-15` is supposed to be the current date. For example if today is `20th June, 2023` then the release tag must be `release-2023-06-20`. Make sure you test the hook at least once and create a release tag for today's release.

Finally remember to push your changes.
**Note:** Perform this task using the `natasha` user, and ensure the repository or existing directory permissions are not altered.

## Specific Requirements:

1. Merge the `feature` branch into the `master` branch, but before pushing your changes complete below point.
2. Create a `post-update` hook in this git repository so that whenever any changes are pushed to the `master` branch, it creates a release tag with name `release-2023-06-15`, where `2023-06-15` is supposed to be the current date. For example if today is `20th June, 2023` then the release tag must be `release-2023-06-20`. Make sure you test the hook at least once and create a release tag for today's release.
3. Finally remember to push your changes.
4. `Note:` Perform this task using the `natasha` user, and ensure the repository or existing directory permissions are not altered.

The working copy was on `feature`, while `master` still pointed to the initial commit. The `post-update` hook was installed in the bare repository `/opt/official.git` before the merge was pushed. The hook checks which references were updated, creates a date-based release tag only when `master` changes, and therefore the required `git push origin master` also tests the hook.
   
---

## 🔹 Step 1: Connect to the Storage Server and inspect the repository

```bash
ssh natasha@ststor01
cd /usr/src/kodekloudrepos/official
git status --short
git branch --show-current
git log

```

**Purpose**: Connect to the storage server (`ststor01`) as the `natasha` user and switch to `root` for administrative access to perform Git operations.

---

## 🔹 Step 2: Navigate to Repository Directory

```bash
cd /usr/src/kodekloudrepos/cluster/
```

**Purpose**: Navigate to the cloned Git repository where the merge and hook setup will be performed.

**Verify Location**:
```bash
pwd
ls -la
```

**Expected Output** (example):
```
total 20
drwxr-xr-x 3 natasha natasha 4096 Sep  6 12:48 .
drwxr-xr-x 3 natasha natasha 4096 Sep  6 12:48 ..
drwxr-xr-x 7 natasha natasha 4096 Sep  6 12:48 .git
-rw-r--r-- 1 natasha natasha   22 Sep  6 12:48 feature.txt
-rw-r--r-- 1 natasha natasha   34 Sep  6 12:48 info.txt
```
---

## 🔹 Step 3: Check Existing Branches

```bash
git status
sudo git branch
```

**Purpose**: List all available branches to confirm the presence of `feature` and `master` branches.

**Expected Output**:
```
* feature
  master
```
**Branch Information**:
- **Current branch**: Indicated by asterisk (*), e.g., `feature`

---

## 🔹 Step 4: Switch to Master Branch

```bash
sudo git checkout master
```
```
Output

Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

## 🔹 Step 5: Merge the feature branch into the master branch 

```bash
sudo git merge feature
```

**Purpose**: Switch to the `master` branch and merge the `feature` branch into it.

**Expected Output** (example):
```
Updating 4b6af24..2a03071
Fast-forward
 feature.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
```

**Merge Details**:
- If the merge is a fast-forward, it updates `master` to include all commits from `feature`.
- If conflicts occur, resolve them manually (see troubleshooting section).

**Verify Merge**:
```bash
git log --oneline
```

**Expected Output** (example):
```
2a03071 (HEAD -> master, origin/feature, feature) Add feature
4b6af24 (origin/master) initial commit
```
---

## 🔹 Step 5: Create Post-Update Hook

```bash
cd /opt/cluster.git
ls
```
Navigate to the hooks

```bash
cd hooks
ls

```
```Expected output
applypatch-msg.sample      pre-applypatch.sample    pre-rebase.sample          sendemail-validate.sample
commit-msg.sample          pre-commit.sample        pre-receive.sample         update.sample
fsmonitor-watchman.sample  pre-merge-commit.sample  prepare-commit-msg.sample
post-update.sample         pre-push.sample          push-to-checkout.sample

```
Create a `post-update` hook in the bare repository (`/opt/cluster.git`) to automatically create a release tag when changes are pushed to the `master` branch.

## 🔹6: Copy the post-update sample

```bash
cp post-update.sample post-update

```
Edit the file with vi

```bash
vi post-update
```
**Hook Script Content**:
```bash
#!/bin/bash
for ref in "$@"; do
    if [ "$ref" = "refs/heads/master" ]; then
        git tag "release-$(date +%F)" "$ref"
    fi
done
```

**Explanation**:
- The `post-update` hook runs after a push to the remote repository.
- The `if` condition checks if the pushed reference is `refs/heads/master` (the `master` branch).
- If true, it creates a tag named `release-YYYY-MM-DD` based on the current date and applies it to the `master` branch.

**Save and Exit**:
- Save the file in `vi` using `:wq`.

---

## 🔹 Step 7: Set Execute Permissions for Hook

```bash
sudo chmod +x post-update  or 
sudo chmod 777 post-update

```

**Purpose**: Ensure the `post-update` hook is executable without altering other permissions.

**Note**: Using `chmod +x` adds execute permissions for all users (owner, group, others) while preserving existing read/write permissions, avoiding the use of `chmod 777` to maintain security.

**Verify Permissions**:
```bash
ls -l 
```

**Expected Output** (example):
```
total 68
-rwxr-xr-x 1 natasha natasha  482 Sep  6 12:48 applypatch-msg.sample
-rwxr-xr-x 1 natasha natasha  900 Sep  6 12:48 commit-msg.sample
-rwxr-xr-x 1 natasha natasha 4726 Sep  6 12:48 fsmonitor-watchman.sample
-rwxr-xr-x 1 natasha natasha  317 Sep  6 13:16 post-update
-rwxr-xr-x 1 natasha natasha  193 Sep  6 12:48 post-update.sample
-rwxr-xr-x 1 natasha natasha  428 Sep  6 12:48 pre-applypatch.sample
-rwxr-xr-x 1 natasha natasha 1653 Sep  6 12:48 pre-commit.sample
-rwxr-xr-x 1 natasha natasha  420 Sep  6 12:48 pre-merge-commit.sample
-rwxr-xr-x 1 natasha natasha 1378 Sep  6 12:48 pre-push.sample
-rwxr-xr-x 1 natasha natasha 4902 Sep  6 12:48 pre-rebase.sample
-rwxr-xr-x 1 natasha natasha  548 Sep  6 12:48 pre-receive.sample
-rwxr-xr-x 1 natasha natasha 1496 Sep  6 12:48 prepare-commit-msg.sample
-rwxr-xr-x 1 natasha natasha 2787 Sep  6 12:48 push-to-checkout.sample
-rwxr-xr-x 1 natasha natasha 2312 Sep  6 12:48 sendemail-validate.sample
-rwxr-xr-x 1 natasha natasha 3654 Sep  6 12:48 update.sample
```
---

## 🔹 Step 8: Go Back to Previous Directory to Push to the Master Branch

```bash
cd /usr/src/kodekloudrepos/cluster
ls
git status
git push origin master

```
output
```
[natasha@ststor01 cluster]$ ls
feature.txt  info.txt
[natasha@ststor01 cluster]$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
[natasha@ststor01 cluster]$ git push origin master
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/cluster.git
   4b6af24..2a03071  master -> master
```


## 🔹 Step 9: Verify Release Tag Creation

```bash
sudo git fetch --tags
```

**Purpose**: Fetch all updates from the remote repository and list tags to confirm the creation of the release tag.

**Expected Output**:
```
From /opt/cluster
 * [new tag]         release-2026-09-06 -> release-2026-09-06
```

**Verification**:
- The tag `release-2026-09-06` should appear, confirming the hook worked as intended.


## 🔹 Step 9: Validate File Changes (Optional)

```bash
cat info.txt
```

**Purpose**: Verify that the file changes from the `feature` branch merge were applied correctly to `master`.

**Expected Output** (example):
```
Updated content from feature branch
```

## 💡 What is a Git Post-Update Hook?

**Post-Update Hook Definition**: A `post-update` hook is a Git server-side hook that runs after a push operation updates references in the remote repository. It’s commonly used for automation tasks like notifications, deployments, or tagging.

### **Key Benefits**:
- **Automation**: Automatically perform actions after a push, like creating tags.
- **Consistency**: Ensures release tags are created for every push to `master`.
- **Team Collaboration**: Tags provide clear markers for releases, aiding team workflows.
- **Traceability**: Release tags link to specific commits for version tracking.

### **How Post-Update Hook Works**:
1. **Trigger**: Executes after a `git push` updates refs (e.g., `refs/heads/master`).
2. **Parameters**: Receives updated ref names as arguments (e.g., `refs/heads/master`).
3. **Script Execution**: Runs the script in the bare repository (`/opt/demo.git/hooks/post-update`).
4. **Tag Creation**: In this case, creates a tag with the current date (`release-YYYY-MM-DD`).

### **Common Use Cases**:
- **Release Tagging**: Mark stable commits with version or date-based tags.
- **Notifications**: Send alerts to team members or CI/CD systems.
- **Deployment Triggers**: Initiate automated deployments after pushes.
- **Audit Logging**: Record push events for tracking.

---

## ⚠️ Important Production Notes

🔧 **Automated Tagging**: The `post-update` hook ensures consistent release tagging without manual intervention.

🔐 **Permission Security**: Using `chmod +x` maintains existing permissions, enhancing security.

📊 **Team Collaboration**: Release tags make it easier for the team to track and deploy specific versions.

🛡️ **Change Validation**: The hook only triggers for `master` branch pushes, avoiding unnecessary tags.