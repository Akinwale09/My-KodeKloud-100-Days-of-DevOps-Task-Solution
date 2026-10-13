# Day 05: SELinux Installation and Configuration

Following a security audit, the xFusionCorp Industries security team has opted to enhance application and server security with SELinux. To initiate testing, the following requirements have been established for App server 3 in the Stratos Datacenter:

## Specific Requirements:

1. Install the required SELinux packages.
2. Permanently disable SELinux for the time being; it will be re-enabled after necessary configuration changes.
3. No need to reboot the server, as a scheduled maintenance reboot is already planned for tonight.
4. Disregard the current status of SELinux via the command line; the final status after the reboot should be disabled.

## Solution

The target server is App Server 3, `stapp03`, and the administrative user for this host is `banner`. The required SELinux packages are installed first. SELinux is then disabled in `/etc/selinux/config`, which controls the state that will be applied during the next boot.

The challenge specifically says not to reboot and to disregard the current command-line status. Therefore, this walkthrough does not run `getenforce`, does not use `setenforce`, and does not reboot the server. The important result is the persistent configuration value `SELINUX=disabled`.

### 🔌 Step 1: Connect to App Server 3

```bash
ssh banner@stapp03
```

The lab connection succeeded and opened a shell as `banner`:

```text
thor@jump-host ~$ ssh banner@stapp03
The authenticity of host 'stapp03 (10.244.29.219)' can't be established.
ED25519 key fingerprint is SHA256:i86BPiXEqSQKQzMmkGnWEjIVMx/8C1YUx9fX3V2syW8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
banner@stapp03's password:
[banner@stapp03 ~]$
```

> **Why:** `ssh` opens an encrypted remote shell session. `banner` is the user assigned to App Server 2, and `stapp03` is the server hostname. The first connection asks whether the server identity should be trusted; answering `yes` stores the host key in the jump host's known-hosts file. The password is then used to authenticate the lab user.

### 📦 Step 2: Install the SELinux packages

```bash
sudo yum install -y selinux-policy selinux-policy-targeted
```

The package installation completed successfully. The transaction installed the two requested SELinux packages and their dependencies, and it also upgraded packages required by the transaction:

```text
[banner@stapp03 ~]$ sudo yum install -y selinux-policy selinux-policy-targeted

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner:

Complete!
[banner@stapp03 ~]$
```

> **Why:** `sudo` runs the package operation with administrative privileges. `yum` is the compatibility command for the DNF package manager on this RHEL 9 host. `install` adds packages from the configured repositories, and `-y` automatically answers yes to the confirmation prompt. `selinux-policy` provides the SELinux policy framework, while `selinux-policy-targeted` provides the targeted policy package. The package manager may also install dependencies and upgrade related packages as part of the transaction. The `Complete!` message confirms that the transaction finished successfully.

### 🔧 Step 3: Permanently disable SELinux for the next reboot

```bash
sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
```

The command completed without an error and returned to the shell prompt:

```text
[banner@stapp03 ~]$ sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
[banner@stapp03 ~]$
```

> **Why:** `sed` edits text from the command line. Its `-i` option saves the change directly to the file. The expression searches for the active configuration line that starts with `SELINUX=`, matches the rest of that line with `.*`, and replaces it with `SELINUX=disabled`. The `/etc/selinux/config` path is SELinux's persistent configuration file. This changes what will happen at the next boot; it does not require or perform an immediate reboot.

### ✅ Step 4: Verify the persistent SELinux configuration

```bash
sudo grep SELINUX /etc/selinux/config
```

The output confirmed that the persistent SELinux state is disabled:

```text
[banner@stapp03 ~]$ sudo grep SELINUX /etc/selinux/config
# SELINUX= can take one of these three values:
# NOTE: Up to RHEL 8 release included, SELINUX=disabled would also
SELINUX=disabled
# SELINUXTYPE= can take one of these three values:
SELINUXTYPE=targeted
[banner@stapp03 ~]$
```

> **Why:** `grep` searches the file for lines containing `SELINUX`, allowing us to inspect the persistent configuration without checking the current runtime state. The important line is `SELINUX=disabled`. `SELINUXTYPE=targeted` identifies the policy type that is configured for future SELinux use, but SELinux will remain disabled after the reboot because the `SELINUX` setting takes precedence. No `getenforce` check or reboot was performed because the challenge explicitly says to disregard the current command-line status and wait for the scheduled maintenance reboot.

## The Steps below can also be follow:

- `ssh banner@stapp03` and provide the password provided
- Check the type of OS first: `cat /etc/os-release`
- Search for Selinux if already installed: `yum search selinux`
- Now install Selinux: `sudo yum install policycoreutils selinux-policy selinux-policy-targeted` 
- Now disable SELinux after reboot: `sudo vi /etc/selinux/config`
- Look for SELINUX and change `from enforcing to disabled`, and save the config file
- Check the SE status: sestatus should show disabled.  


## Best Practices

- **Install the required policy packages.** Install both the base `selinux-policy` package and the `selinux-policy-targeted` package so the host has the policy components needed for future SELinux configuration.
- **Separate persistent configuration from runtime state.** Changing `/etc/selinux/config` controls the next boot; it does not necessarily change the mode of a running system immediately.
- **Do not reboot outside the maintenance window.** The challenge explicitly provides a scheduled reboot, so the change was prepared without interrupting the server.
- **Avoid disabling SELinux as a general security practice.** Red Hat recommends using an appropriate SELinux mode instead of permanently disabling the security control; this lab requires disabling it temporarily for a later configuration phase.
- **Use `sudo` for the smallest required operation.** Administrative privileges are needed to install packages and modify `/etc/selinux/config`, but no full root shell is necessary.

### 📚 Official Documentation

- [Project Nautilus infrastructure](https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus)
- [Red Hat Enterprise Linux 9 — Managing software with the DNF tool](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/htmlsingle/managing_software_with_the_dnf_tool/index)
- [Red Hat Enterprise Linux 8 — Changing SELinux states and modes](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/using_selinux/changing-selinux-states-and-modes_using-selinux)
- [GNU `sed` manual](https://www.gnu.org/software/sed/manual/sed.html)
- [GNU `grep` manual](https://www.gnu.org/software/grep/manual/grep.html)
