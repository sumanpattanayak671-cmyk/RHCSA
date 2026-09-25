# RHEL 10 / Linux Administration Practice — Tasks 1–24

> **Exam/Lab Notes:** Commands are organized task-by-task. Review the verification commands before the practical exam.

---

## 1. Find All Files Owned by User `student`

### Requirement

Locate all files owned by user `student` and copy them into:

```text
/root/found
```

### Commands

```bash
mkdir -p /root/found
find / -user student -type f -exec cp -p {} /root/found/ \;
```

### Test File Example

```bash
touch /tmp/zimba
chown student: /tmp/zimba
```

Run the `find` command and verify:

```bash
ls -l /root/found
```

> `-type f` limits the search to regular files. This avoids attempting to copy directories.

---

# 2. Search Lines Containing `command`

### Requirement

Search `/usr/share/dict/words` for lines containing the string `command` and save the result to:

```text
/root/command.txt
```

Preserve the original order.

### Command

```bash
grep "command" /usr/share/dict/words > /root/command.txt
```

### Verify

```bash
cat /root/command.txt
```

---

# 2a. Search Lines Starting With `command`

### Requirement

Find lines that start with `command` and save them to:

```text
/root/command1.txt
```

### Command

```bash
grep "^command" /usr/share/dict/words > /root/command1.txt
```

### Important

```text
^
```

means **start of the line**.

---

# 2b. Search Lines Ending With `command`

### Requirement

Find lines that end with `command` and save them to:

```text
/root/command2.txt
```

### Command

```bash
grep "command$" /usr/share/dict/words > /root/command2.txt
```

### Important

```text
$
```

means **end of the line**.

---

# Create, Manage, and Delete Local Users and Groups

## 3. Password Aging Policy

### Requirement

All current users and future users should:

- Change their password every **45 days**
- Receive a warning **5 days** before password expiration

Edit:

```bash
vim /etc/login.defs
```

Set:

```text
PASS_MAX_DAYS   45
PASS_MIN_DAYS   0
PASS_WARN_AGE   5
```

### Important

`/etc/login.defs` mainly provides the default aging values for newly created users. To change the aging policy for **existing users**, use `chage`.

Example:

```bash
chage -M 45 -W 5 username
```

For all existing regular users, apply the policy individually as required by the lab.

---

# 4. Create Groups and Users

### Requirements

Create:

- Group: `sysadmin`
- GID: `10001`
- User `romeo` → secondary member of `sysadmin`
- User `juliet` → secondary member of `sysadmin`
- User `elvis` → no interactive shell and not a member of `sysadmin`
- Password for all users: `redhat@123`

### Create Group

```bash
groupadd -g 10001 sysadmin
```

### Create `romeo`

```bash
useradd -G sysadmin romeo
echo 'redhat@123' | passwd --stdin romeo
```

### Create `juliet`

```bash
useradd -G sysadmin juliet
echo 'redhat@123' | passwd --stdin juliet
```

### Create `elvis`

```bash
useradd -s /sbin/nologin elvis
echo 'redhat@123' | passwd --stdin elvis
```

### Verify

```bash
id romeo
id juliet
id elvis
getent group sysadmin
```

> On RHEL systems where `passwd --stdin` is unavailable, use `passwd username` interactively instead.

---

# 5. Configure `cloudadmin` for Passwordless Sudo

### Requirement

Members of `cloudadmin` must be able to execute any command as superuser without entering a password.

### Create Group

```bash
groupadd cloudadmin
```

### Configure sudo

Use:

```bash
visudo
```

Add:

```text
%cloudadmin ALL=(ALL) NOPASSWD: ALL
```

### Add a User to the Group

For example:

```bash
usermod -aG cloudadmin username
```

### Verify

```bash
id username
sudo -l -U username
```

---

# 6. Create User `hamlet`

### Requirements

- Username: `hamlet`
- UID: `6001`
- Password: `redhat@123`

### Commands

```bash
useradd -u 6001 hamlet
echo 'redhat@123' | passwd --stdin hamlet
```

### Verify

```bash
id hamlet
```

Expected:

```text
uid=6001(hamlet) gid=6001(hamlet) groups=6001(hamlet)
```

---

# Controlling Access to Files

## 7. Set `umask` for User `hamlet`

### Requirement

Files created by `hamlet` should have:

```text
Files       → 640
Directories → 750
```

### Calculate the Umask

Default permissions:

```text
Files       666
Directories 777
```

Desired permissions:

```text
Files       640
Directories 750
```

Therefore:

```text
umask = 027
```

### Configure

Edit:

```bash
vim /home/hamlet/.bashrc
```

Add:

```bash
umask 027
```

### Test

Log in as `hamlet`:

```bash
su - hamlet
```

Check:

```bash
umask
```

Expected:

```text
0027
```

Create a directory:

```bash
mkdir testdir
ls -ld testdir
```

Expected permissions:

```text
drwxr-x---
```

For a new file:

```bash
touch testfile
ls -l testfile
```

Expected permissions:

```text
-rw-r-----
```

---

# 8. Configure `/backup`

### Requirements

Create:

```text
/backup
```

The directory must:

- Be owned by group `sysadmin`
- Be readable, writable, and accessible by `sysadmin`
- Not be accessible to other users
- Automatically give new files the `sysadmin` group
- Allow only file owners to delete their own files

### Commands

```bash
mkdir /backup
chgrp sysadmin /backup
chmod 3770 /backup
```

Verify:

```bash
ls -ld /backup
```

Expected:

```text
drwxrws--T. 2 root sysadmin ... /backup
```

### Permission Breakdown

```text
3     → special permissions
7     → owner: rwx
7     → group: rwx
0     → others: ---
```

The special bits are:

```text
2 → SGID
1 → Sticky bit
```

Therefore:

```text
3770 = SGID + Sticky + 770
```

### Alternative

```bash
chmod 770 /backup
chmod g+s /backup
chmod o+t /backup
```

---

# Manage Time Across Multiple Systems

## 9. Configure NTP Client

### Requirement

Configure the system as an NTP client of:

```text
1.in.pool.ntp.org
```

### Enable NTP

```bash
timedatectl set-ntp true
```

### Edit Chrony Configuration

```bash
vim /etc/chrony.conf
```

Comment out the existing pool if required:

```text
#pool 2.rhel.pool.ntp.org iburst
```

Add:

```text
server 1.in.pool.ntp.org iburst
```

Restart Chrony:

```bash
systemctl restart chronyd
```

### Verify

```bash
chronyc sources
```

A working source normally shows `^*` for the currently selected source.

> The server name displayed by `chronyc sources` may be different from `1.in.pool.ntp.org` because the pool name can resolve to a particular NTP server.

---

# Configure Networking in RHEL

## 10. TCP/IP Configuration

### Required Configuration

```text
IP Address  : 192.168.X.101
Subnet Mask : 255.255.255.0
Gateway     : 192.168.X.1
DNS Server  : 192.168.X.1
Hostname    : station.domain101.example.com
```

Replace `X` according to the lab network.

### Console Configuration

Enable NetworkManager:

```bash
systemctl enable NetworkManager
systemctl restart NetworkManager
```

Launch the text-based NetworkManager interface:

```bash
nmtui
```

Configure:

- IPv4 address
- Prefix: `24`
- Gateway
- DNS server
- Hostname

### Verify

```bash
hostnamectl
ip addr
ip route
cat /etc/resolv.conf
```

---

# Archiving and Transferring Files

## 11. Create a Gzip-Compressed Archive

### Requirement

Create:

```text
/root/usrlocal.tgz
```

from:

```text
/usr/local
```

Use gzip compression.

### Commands

```bash
dnf install gzip -y
tar -czf /root/usrlocal.tgz /usr/local
```

### Verify

```bash
ls -lh /root/usrlocal.tgz
tar -tzf /root/usrlocal.tgz | head
```

### Options

```text
-c → create
-z → gzip
-f → specify archive filename
```

---

# 12. Create a Bzip2-Compressed Archive

### Requirement

Create:

```text
/root/etc.tar.bz2
```

from:

```text
/etc
```

Use bzip2 compression.

### Commands

```bash
dnf install bzip2 -y
tar -cjf /root/etc.tar.bz2 /etc
```

### Verify

```bash
ls -lh /root/etc.tar.bz2
tar -tjf /root/etc.tar.bz2 | head
```

### Options

```text
-c → create
-j → bzip2
-f → specify archive filename
```

---

# Installing and Updating Software Packages

## 13. Configure YUM/DNF Repositories

### Repository URLs

For CentOS Stream 9:

```text
https://mirror.stream.centos.org/9-stream/BaseOS/x86_64/os/
https://mirror.stream.centos.org/9-stream/AppStream/x86_64/os/
```

For CentOS Stream 10:

```text
https://mirror.stream.centos.org/10-stream/BaseOS/x86_64/os/
https://mirror.stream.centos.org/10-stream/AppStream/x86_64/os/
```

> Use the repository version appropriate to the lab environment. Do not mix a CentOS Stream repository with a RHEL installation unless the lab explicitly requires it.

### Create Repository File

```bash
vim /etc/yum.repos.d/local.repo
```

Example for CentOS Stream 9:

```ini
[BaseOS]
name=BaseOS
baseurl=https://mirror.stream.centos.org/9-stream/BaseOS/x86_64/os/
enabled=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=https://mirror.stream.centos.org/9-stream/AppStream/x86_64/os/
enabled=1
gpgcheck=0
```

### Clean Metadata

```bash
dnf clean all
```

### Verify Repositories

```bash
dnf repolist
```

Expected repositories include:

```text
BaseOS
AppStream
```

---

# Scheduling Future Tasks

## 14. Daily Cron Job at 17:23

### Requirement

User `sarah` must run:

```bash
/bin/echo Hello World
```

every day at:

```text
17:23
```

### Configure

```bash
crontab -u sarah -e
```

Add:

```cron
23 17 * * * /bin/echo Hello World
```

### Verify

```bash
crontab -u sarah -l
```

Expected:

```text
23 17 * * * /bin/echo Hello World
```

Ensure cron service is enabled:

```bash
systemctl enable --now crond
```

### Cron Field Order

```text
Minute Hour Day Month Weekday Command
  23    17   *    *      *      command
```

---

# 15. Cron Job Every 3 Minutes

### Requirement

User `sarah` must execute:

```bash
logger "The exam is going on"
```

every 3 minutes.

### Configure

```bash
crontab -u sarah -e
```

Add:

```cron
*/3 * * * * logger "The exam is going on"
```

### Verify

```bash
crontab -u sarah -l
```

### Meaning of `*/3`

```text
*/3
```

means every 3 minutes:

```text
00, 03, 06, 09, 12, ...
```

---

# 16. Systemd Service and Timer

## Requirement

Create:

```text
/usr/local/bin/log_capture
```

The script must list all files in `/tmp` and save the output to:

```text
/root/log_output/system_logs.trc
```

Then create:

```text
log_capture.service
log_capture.timer
```

The timer should run the service every 1 minute, start immediately, and continue after reboot.

---

## Step 1: Create the Script

```bash
vim /usr/local/bin/log_capture
```

Add:

```bash
#!/bin/bash
ls -l /tmp > /root/log_output/system_logs.trc
```

Create the output directory first:

```bash
mkdir -p /root/log_output
```

Make the script executable:

```bash
chmod +x /usr/local/bin/log_capture
```

Verify:

```bash
ls -l /usr/local/bin/log_capture
```

---

## Step 2: Create the Service

Create:

```bash
vim /etc/systemd/system/log_capture.service
```

Add:

```ini
[Unit]
Description=Log Capture

[Service]
Type=oneshot
ExecStart=/usr/local/bin/log_capture
```

---

## Step 3: Create the Timer

Create:

```bash
vim /etc/systemd/system/log_capture.timer
```

Add:

```ini
[Unit]
Description=Log capture timer

[Timer]
OnBootSec=1min
OnUnitActiveSec=1min

[Install]
WantedBy=timers.target
```

### Meaning

```text
OnBootSec=1min
```

Runs approximately one minute after boot.

```text
OnUnitActiveSec=1min
```

Runs the service again every minute after the service is activated.

---

## Step 4: Reload Systemd

```bash
systemctl daemon-reload
```

---

## Step 5: Enable and Start Timer

```bash
systemctl enable --now log_capture.timer
```

Run the service manually once for immediate verification:

```bash
systemctl start log_capture.service
```

---

## Step 6: Verify

Check timer:

```bash
systemctl status log_capture.timer
```

or:

```bash
systemctl list-timers --all | grep log_capture
```

Check output:

```bash
cat /root/log_output/system_logs.trc
```

Check service:

```bash
systemctl status log_capture.service
```

---

# Tuning System Performance

## 17. Configure Recommended Tuned Profile

### Install Tuned

```bash
dnf install tuned -y
```

Enable and start:

```bash
systemctl enable --now tuned
```

### Check Current Profile

```bash
tuned-adm active
```

### Check Recommended Profile

```bash
tuned-adm recommend
```

Example:

```text
virtual-guest
```

### Apply Recommended Profile

```bash
tuned-adm profile virtual-guest
```

### Verify

```bash
tuned-adm active
```

Expected:

```text
Current active profile: virtual-guest
```

> Always use the profile returned by `tuned-adm recommend` for the actual system rather than memorizing `virtual-guest`.

---

# Protect and Manage Security With SELinux

## 18. Run HTTPD on TCP Port 82

### Requirement

A web application uses:

```text
TCP port 82
```

Configure the system so that:

- `httpd` is installed
- HTTPD starts after reboot
- SELinux permits HTTPD to use TCP port 82
- Firewall allows TCP/82
- The application is accessible over the network

---

## Step 1: Install HTTPD

```bash
dnf install httpd -y
```

---

## Step 2: Enable HTTPD

```bash
systemctl enable httpd
```

Start/restart:

```bash
systemctl restart httpd
```

If it fails because SELinux does not allow port 82, continue with the SELinux configuration.

---

## Step 3: Allow Port 82 in SELinux

Check the current HTTP ports:

```bash
semanage port -l | grep http_port_t
```

Add port 82:

```bash
semanage port -a -t http_port_t -p tcp 82
```

If the port already exists with another definition, use:

```bash
semanage port -m -t http_port_t -p tcp 82
```

Restart HTTPD:

```bash
systemctl restart httpd
```

Verify:

```bash
systemctl status httpd
```

---

## Step 4: Allow Port 82 Through Firewalld

```bash
firewall-cmd --permanent --add-port=82/tcp
firewall-cmd --reload
```

Verify:

```bash
firewall-cmd --list-ports
```

---

## Step 5: Verify HTTPD Listening Port

```bash
ss -lntp | grep :82
```

---

# Managing Basic Storage

## 19. Create a Persistent 700 MiB Swap Partition

### Requirement

Create a new:

```text
700 MiB swap partition
```

without disturbing the existing swap.

Example disk:

```text
/dev/sdb
```

---

## Step 1: Check Disks

```bash
lsblk
```

Example:

```text
sda
├─sda1
├─sda2
├─sda3  [SWAP]
...
sdb
└─sdb1
```

---

## Step 2: Create Partition

```bash
fdisk /dev/sdb
```

Inside `fdisk`:

```text
n
```

Choose the next partition number.

For size:

```text
+700M
```

Change the partition type:

```text
t
```

Select:

```text
19
```

for Linux swap on GPT.

Write changes:

```text
w
```

---

## Step 3: Reload Partition Table

```bash
partprobe
```

---

## Step 4: Format as Swap

```bash
mkswap /dev/sdb2
```

Record the UUID returned by `mkswap`.

---

## Step 5: Add to `/etc/fstab`

Edit:

```bash
vim /etc/fstab
```

Add:

```text
UUID=<NEW-SWAP-UUID> none swap defaults 0 0
```

Example:

```text
UUID=06da2a37-7dfd-441f-b821-897177ed63a3 none swap defaults 0 0
```

Reload systemd:

```bash
systemctl daemon-reload
```

Enable all fstab swap entries:

```bash
swapon -a
```

---

## Step 6: Verify

```bash
free -h
swapon --show
```

Example:

```text
Swap: 2.7Gi  0B  2.7Gi
```

The original swap remains active and the new 700 MiB swap is added.

---

# Managing Logical Volumes

## 20. Create an LVM Logical Volume

### Requirement

Create:

- VG: `datastore`
- LV: `database`
- Extent size: `32 MiB`
- LV size: `15 extents`
- Filesystem: `ext3`
- Mount point: `/database`

### Calculate Size

```text
15 × 32 MiB = 480 MiB
```

So the LV should be:

```text
480 MiB
```

---

## Step 1: Calculate

```bash
bc
```

Then:

```text
15*32
```

Result:

```text
480
```

---

## Step 2: Create an LVM Partition

```bash
fdisk /dev/sdb
```

Create a partition of approximately:

```text
+544M
```

Change its type to:

```text
Linux LVM
```

Write:

```text
w
```

Reload:

```bash
partprobe
```

---

## Step 3: Create Physical Volume

```bash
pvcreate /dev/sdb3
```

---

## Step 4: Create Volume Group

Set physical extent size to 32 MiB:

```bash
vgcreate -s 32M datastore /dev/sdb3
```

Verify:

```bash
vgs
```

---

## Step 5: Create Logical Volume

Create 15 extents:

```bash
lvcreate -n database -l 15 datastore
```

Verify:

```bash
lvs
```

Expected size:

```text
database  datastore  ...  480.00m
```

---

## Step 6: Create ext3 Filesystem

```bash
mkfs -t ext3 /dev/datastore/database
```

---

## Step 7: Create Mount Point

```bash
mkdir /database
```

---

## Step 8: Configure `/etc/fstab`

Add:

```text
/dev/datastore/database /database ext3 defaults 0 0
```

> Make sure the mount point is `/database`. A typo such as `/databse` will prevent the requested mount from being configured correctly.

Reload:

```bash
systemctl daemon-reload
```

Mount:

```bash
mount -a
```

---

## Step 9: Verify

```bash
df -hT
```

Expected:

```text
/dev/mapper/datastore-database  ext3  ~440M  ...  /database
```

---

# 21. Extend `/image` by 1 GiB

### Requirement

An LVM logical volume is mounted at:

```text
/image
```

Extend its filesystem by:

```text
1 GiB
```

### Check Current Size

```bash
df -hT
```

Example:

```text
/dev/mapper/rhel-image  xfs  736M  38M  699M  6%  /image
```

### Extend LV and Filesystem

```bash
lvresize /dev/mapper/rhel-image -L +1G -r
```

The `-r` option resizes the filesystem automatically.

For XFS, the operation uses:

```bash
xfs_growfs
```

### Verify

```bash
df -hT
```

Expected:

```text
/dev/mapper/rhel-image  xfs  1.8G  45M  1.7G  3%  /image
```

---

# 22. Mount NFS Home Directory Automatically Using Autofs

### Requirement

NFS server:

```text
classroom.example.com
```

Export:

```text
/rhome
```

User:

```text
natasha
```

Expected local path:

```text
/rhome/natasha
```

Remote path:

```text
classroom.example.com:/rhome/natasha
```

---

## Step 1: Install Packages

```bash
dnf install nfs-utils autofs -y
```

---

## Step 2: Configure Autofs Master Map

Create:

```bash
vim /etc/auto.master.d/rhome.autofs
```

Add:

```text
/rhome  /etc/auto.rhome
```

---

## Step 3: Configure Map

Create:

```bash
vim /etc/auto.rhome
```

Add:

```text
*  -rw,sync,fstype=nfs4  classroom.example.com:/rhome/natasha
```

---

## Step 4: Enable and Start Autofs

```bash
systemctl enable autofs
systemctl restart autofs
```

---

## Step 5: Test

Log in as `natasha` as required by the lab.

Then:

```bash
pwd
```

Expected:

```text
/rhome/natasha
```

Verify the NFS mount:

```bash
mount | grep /rhome
```

---

# 23. Reset Root Password

### Requirement

Set the new root password to:

```text
redhat@123
```

### Command

```bash
passwd root
```

Enter:

```text
redhat@123
```

when prompted for the new password and confirmation.

Verify by opening a new root login/session if appropriate.

---

# Installing and Updating Applications Using Flatpak

## Flatpak Overview

Flatpak is a Linux application packaging and distribution framework.

### Main Features

- **Universal format:** Applications can run on multiple Linux distributions.
- **Sandboxing:** Applications run with isolation from the host system.
- **Repository-based:** Applications can be distributed from repositories such as Flathub.
- **Independent application updates:** Applications can be updated independently of many OS packages.
- **Runtimes:** Shared Flatpak runtimes provide application dependencies.

---

# 24. Configure Flatpak for User `student`

### Requirements

Configure Flatpak so that:

1. Repository name is `flatdb`.
2. Repository is configured **only for user `student`**.
3. Codium is installed for `student`.
4. Do **not** use `su - student`; use SSH.

Repository:

```text
https://flathub.org/repo/flathub.flatpakrepo
```

---

## Step 1: Install Flatpak

As root:

```bash
dnf install flatpak -y
```

---

## Step 2: SSH as `student`

Use:

```bash
ssh student@station
```

Do not use:

```bash
su - student
```

---

## Step 3: Check Flatpak Remotes

As `student`:

```bash
flatpak remotes
```

---

## Step 4: Add User-Only Repository

```bash
flatpak remote-add --no-gpg-verify --user flatdb https://flathub.org/repo/flathub.flatpakrepo
```

### Important

The:

```text
--user
```

option means the remote is configured for the current user rather than system-wide.

Verify:

```bash
flatpak remotes
```

---

## Step 5: Find Codium

Use the repository name you created:

```bash
flatpak remote-ls flatdb --app | grep -i codium
```

If the output provides the application ID, install that application.

---

## Step 6: Install Codium

```bash
flatpak install --user flatdb <CODIUM_APPLICATION_ID>
```

Follow the prompts.

---

## Step 7: Verify

```bash
flatpak list --user
```

> Common mistakes:
>
> - `flatpack` is incorrect; use `flatpak`.
> - `flatrepo` does not match the required remote name; use `flatdb`.
> - `flatpak install codium` may not be sufficient if the exact application ID is required. Use the application ID returned by `flatpak remote-ls`.

---

# 25. Bash Script — Find SGID Files

## Requirement

Create:

```text
/usr/local/bin/myscript
```

The script must search under:

```text
/usr
```

and copy files into:

```text
/root/filefound
```

### Conditions

Files must:

1. Be larger than 30 KB.
2. Be smaller than 50 KB.
3. Have the SGID permission.
4. Create `/root/filefound` automatically if it does not exist.

---

## Step 1: Create Script

```bash
vim /usr/local/bin/myscript
```

Add:

```bash
#!/bin/bash

if [ ! -d /root/filefound ]
then
    mkdir /root/filefound
fi

find /usr -type f -size +30k -size -50k -perm -2000 -exec cp -p {} /root/filefound/ \;
```

---

## Step 2: Make It Executable

```bash
chmod +x /usr/local/bin/myscript
```

---

## Step 3: Run

```bash
myscript
```

---

## Step 4: Verify

```bash
ls -l /root/filefound/
```

Example:

```text
-rwx--s--x. 1 root slocate 41032 Aug 10 2021 locate
```

### Understanding the `find` Command

```bash
find /usr -type f -size +30k -size -50k -perm -2000 -exec cp -p {} /root/filefound/ \;
```

| Option | Meaning |
|---|---|
| `/usr` | Start searching under `/usr` |
| `-type f` | Regular files only |
| `-size +30k` | Larger than 30 KB |
| `-size -50k` | Smaller than 50 KB |
| `-perm -2000` | SGID bit is set |
| `-exec` | Execute a command for every match |
| `cp -p` | Copy while preserving file attributes |
| `{}` | Current matched file |
| `/root/filefound/` | Destination |
| `\;` | End of the `-exec` command |

---

# Quick Revision Sheet

## File Search

```bash
find / -user student -type f -exec cp -p {} /root/found/ \;
```

## grep

```bash
grep "command" file
grep "^command" file
grep "command$" file
```

## Password Aging

```text
PASS_MAX_DAYS 45
PASS_WARN_AGE 5
```

## Group/User

```bash
groupadd -g 10001 sysadmin
useradd -G sysadmin romeo
useradd -G sysadmin juliet
useradd -s /sbin/nologin elvis
useradd -u 6001 hamlet
```

## Sudo

```text
%cloudadmin ALL=(ALL) NOPASSWD: ALL
```

## Umask

```bash
umask 027
```

Result:

```text
Files       → 640
Directories → 750
```

## SGID + Sticky Directory

```bash
chmod 3770 /backup
```

## Chrony

```bash
vim /etc/chrony.conf
server 1.in.pool.ntp.org iburst
systemctl restart chronyd
chronyc sources
```

## Network

```bash
nmtui
```

## Archives

```bash
tar -czf /root/usrlocal.tgz /usr/local
tar -cjf /root/etc.tar.bz2 /etc
```

## Cron

```cron
23 17 * * * /bin/echo Hello World
*/3 * * * * logger "The exam is going on"
```

## Systemd Timer

```bash
systemctl daemon-reload
systemctl enable --now log_capture.timer
systemctl list-timers --all | grep log_capture
```

## Tuned

```bash
tuned-adm recommend
tuned-adm profile <recommended-profile>
tuned-adm active
```

## SELinux HTTP Port

```bash
semanage port -a -t http_port_t -p tcp 82
firewall-cmd --permanent --add-port=82/tcp
firewall-cmd --reload
```

## Swap

```bash
mkswap /dev/sdb2
swapon -a
free -h
swapon --show
```

## LVM

```bash
pvcreate /dev/sdb3
vgcreate -s 32M datastore /dev/sdb3
lvcreate -n database -l 15 datastore
mkfs -t ext3 /dev/datastore/database
```

## Extend LVM + XFS

```bash
lvresize /dev/mapper/rhel-image -L +1G -r
df -hT
```

## Autofs

```text
/rhome  /etc/auto.rhome
```

```text
* -rw,sync,fstype=nfs4 classroom.example.com:/rhome/natasha
```

## Flatpak

```bash
flatpak remote-add --no-gpg-verify --user flatdb https://flathub.org/repo/flathub.flatpakrepo
flatpak remote-ls flatdb --app | grep -i codium
flatpak list --user
```

## Script

```bash
chmod +x /usr/local/bin/myscript
myscript
```

---

# Important Corrections to the Original Notes

### 1. Task Numbering

The original notes contain two separate items numbered `2`; they are organized here as:

```text
2   → contains "command"
2a  → starts with "command"
2b  → ends with "command"
```

### 2. `/database` Mount Point

Use:

```text
/dev/datastore/database /database ext3 defaults 0 0
```

not:

```text
/dev/datastore/database /databse ext3 defaults 0 0
```

because `databse` is a spelling error.

### 3. Flatpak Remote Name

If the remote is named:

```text
flatdb
```

then use:

```bash
flatpak remote-ls flatdb --app
```

not:

```bash
flatpak remote-ls flatrepo --app
```

### 4. Flatpak Command

Correct:

```bash
flatpak
```

Incorrect:

```bash
flatpack
```

### 5. `/root/found`

Prefer:

```bash
mkdir -p /root/found
```

and:

```bash
find / -user student -type f -exec cp -p {} /root/found/ \;
```

This explicitly searches regular files only.

### 6. Current RHEL/CentOS Repository Caution

The CentOS Stream repositories shown in the exercise are **not automatically interchangeable with RHEL repositories**. For an actual RHEL installation, use the repository configuration specified by your lab/environment unless the instructor explicitly requires the CentOS Stream URLs.
