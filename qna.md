# 1. **IP Address Configuration, Hostname Change:**
<img width="612" height="248" alt="WhatsApp Image 2026-06-11 at 8 27 41 AM" src="https://github.com/user-attachments/assets/dd6ea055-e689-4665-bc29-c4db9c685fe7" />

Two ways to do:

**1.nmcli (CLI)**
```
nmcli con mod <interface_name> ipv4.addresses <IP_Address/CIDR> ipv4.gateway <IP_Address> connection.autoconnect yes
```

**2.nmtui (GUI)**
```
#nmtui (To configure IP address)
#nmcli device show (To check)
```

Edit a connection> Select the interface> IPv4 Configuration: Manual> IPv4 Address Show> Enter the Content and OK> Back> Activate a connection> deactive and activate and Back> Set System Hostname, Change name, OK> Quit

# 2.**Configure an YUM repository** 
**(remember /etc/yum.repos.d/x.repo is a YUM/DNF repository configuration file that contains definitions of software repositories (repository ID, name, base URL, GPG settings, enable/disable options, etc.) used by the package manager to install and update packages.)**

<img width="736" height="283" alt="WhatsApp Image 2026-06-11 at 8 28 09 AM" src="https://github.com/user-attachments/assets/1bf3685b-22e5-4035-a827-a8138643501c" />

Step 1: Create the Configuration File
Open a terminal and create a new repository file (e.g., rhel10.repo) 

```
#vi /etc/yum.repos.d/rhel10.repo
```
Step 2: Add the Repository Configurations
Paste the following configuration
```
[BaseOS]
name=BaseOS
baseurl=http://content.example.com/rhel8.2/x86_64/dvd/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://content.example.com/rhel8.2/x86_64/dvd/AppStream
enabled=1
gpgcheck=0
```

**Note on Parameters:**
Save and close the file (in vim, press Esc, type :wq, and press Enter).

Step 3: Verify and Clean the Cache
Once the file is saved, clear the package manager cache and verify that the new repositories are correctly detected and enabled:
```
#sudo dnf clean all (cleans out the cached data that your system downloaded from those sources)
#sudo dnf repolist (Status Checker: It looks through all the configuration files inside /etc/yum.repos.d/ and prints out a clean summary of every repository your system is currently allowed to use.)
```
**Note:**

**[...]** defines the unique Repository ID.

**name=** defines the display name of the repository.

**baseurl=** points to the exact URL where the packages are hosted.
  - **file:///root/foldername/**... uses the local filesystem.
  How it works: The system looks for the packages directly on your local hard drive or SSD. The iso consists certain version of packages.
  Requirement: The entire RHEL installation ISO or repository folder must already be downloaded, extracted, or mounted inside that specific local directory (/root/foldername/...).
  - **http://content.example.com/**... uses the network (HTTP protocol).
  How it works: The system downloads packages over a network or the internet from a remote web server whenever you try to install something.
  Requirement: Your machine must have an active network connection and a properly configured DNS/routing setup to reach content.example.com.

**enabled=1** ensures the repository is active (as requested).

**gpgcheck=0** gpgcheck does authenticates the package sourcedisables GPG key signature checking, which is ideal here since no GPG key URL was provided.

**BaseOS** contains The Linux Kernel, systemd, glibc, OpenSSH, and the dnf package manager itself.

**AppStream** contains Databases (MySQL, PostgreSQL), programming languages (Python, Node.js, PHP), web servers (nginx, Apache), and system administration tools.

# 3.Create users and groups as per the following requirements
<img width="1082" height="213" alt="3" src="https://github.com/user-attachments/assets/7dbcc940-fe41-4cb9-ae6a-ce28b2ca1e42" />

```
#groupadd sysadms
#useradd -G sysadms natasha
#useradd -G sysadms jerry
#useradd -s /sbin/nologin sarah
#passwd natasha
#passwd jerry
#passwd sarah
(Checking)
#cat /etc/passwd
#cat /etc/group
#su <sarah> (user won't be able to login)
```

# 4. Setup collaborative directory
<img width="1259" height="317" alt="4" src="https://github.com/user-attachments/assets/18cf8fb9-30d3-4e15-a26f-e42efaea6fc1" />

```
#mkdir /home/newfolder
#groupadd sysadmin
#chgrp sysadmin /home/newfolder
#chmod 770 /home/newfolder
#chmod g+s /home/newfolder (If a user named rupankar (who belongs to the sysadms group) creates a file called test.txt, Even if rupankar's primary group is student, the file test.txt will automatically have its group set to sysadms.)
#ls -ld /home/newfolder (Checking)
```

# 5. Configure NTP Client

<img width="806" height="209" alt="5" src="https://github.com/user-attachments/assets/c1842752-91fd-4295-8d85-5b1b6e499072" />

```
#timedatectl set-ntp true (enables Network Time Protocol (NTP) synchronization on your Linux system based on the system's time synchronization daemon chronyd)
#vim /etc/chrony.conf (chrony.conf the configuration file for the time synchronization daemon)
```

Only write "<domain.com> iburst" and save

```
#timedatectl (Check)
```

**Note:**
/etc/chrony.conf contains the configuration settings for the Chrony service, including NTP servers and time synchronization options.
iburst (initial burst) tells the time daemon (chronyd) to aggressively sync the system clock right when the service starts up or when a network connection is first established.

# 6.create a user called system with ID number 2026

```
#sudo useradd -u 2026 <username>
```

# 7.Create a tar archive
<img width="1097" height="176" alt="6" src="https://github.com/user-attachments/assets/33ce5d87-f3ed-473b-ab64-ec86c169d7e6" />

Create an archive called archive.tar.bz2 which contains the contents of /usr/local directory using bzip2 compression to perform this task
using bzip2
```
#yum install bzip*
#tar -Jcvf archive.tar.bz2 /usr/local/
```
**Note:**
using gzip
```
#yum install gzip*
#tar -zcvf archive.tar.gz2 /usr/local/
```

Decompress Not needed
Using gzip
```
gzip -d file.txt.gz   # or gunzip file.txt.gz
```
Using bzip2
```
bzip2 -d file.txt.bz2 # or bunzip2 file.txt.bz2
```

# 8.Password Reset RHEL 10
**(/usr/sbin/reboot is the executable command used to safely restart the Linux system.)**
<img width="734" height="123" alt="8" src="https://github.com/user-attachments/assets/37cdcbc1-1818-4d7c-a862-351d91ecf505" />

Restart the machine first go to GRUB menu then press e to enter
```
init=/bin/bash (After the Linux line)
ctrl+x
mount -o remount,rw /
passwd root
touch /.autorelabel
/usr/sbin/reboot -f
```
login with the new passwords.


# 9.Create an Application message while login
<img width="885" height="119" alt="8" src="https://github.com/user-attachments/assets/4b3d8343-379b-4998-b2d6-bf9954128b19" />

Write an application called rhcsa which should display "EX200 Exam whenever the user abid login.

```
vim /usr/local/bin/rhcsa (To create a binary executable)
```
write 
echo "EX200 Exam"
```
chmod +x /usr/local/bin/rhcsa 
vim /home/abid/.bash_profile (To modify user-specific startup commands)
```
write
/usr/local/bin/rhcsa (To execute the exucutable file while login)

Check: su - abid
**Note:**
/usr/local/bin/ 

# 10.

<img width="767" height="187" alt="10" src="https://github.com/user-attachments/assets/a8c33976-987e-4716-8e7a-2aa6c922cb7c" />

```
tuned-adm recomended (To Find the recommended profile)
tuned-adm profile virtual-guest (To set the profile)
systemctl enable --now tuned (To start the profile service)
tuned-adm active (To check Which profile the system has been configured)
```

# 11. Default Umask value change

<img width="833" height="195" alt="11" src="https://github.com/user-attachments/assets/e7d4439f-c384-4558-9307-ec5051468c23" />

```
useradd sam
su - sam
touch a1
touch a2
ls -l
umask (To check value of current user)
vim /home/username/.bash_profile (To change umask value for a particular user)
vim /etc/login.defs (To change value for all users)
```
change the umask value, logout and login again

# 12.Password Policy
**(remember /etc/login.defs defines default settings for user account creation, password policies, and system login behavior.)**
<img width="896" height="124" alt="12" src="https://github.com/user-attachments/assets/ba302121-c285-4623-87f5-e46bac579c28" />

```
#vim /etc/login.defs
```
maximum date will be 2 days


# 13. 
<img width="1027" height="211" alt="WhatsApp Image 2026-06-18 at 2 43 50 PM" src="https://github.com/user-attachments/assets/4e588e3f-f209-4c5e-b4ec-392b2e0aa1fe" />

for perticular user
```
useradd sarah
crontab -e -u sarah
*/ * * * * logger "ex200"
```
:wq
crontab -lu sarah

# 14.Criteria Based Search
<img width="1218" height="171" alt="14" src="https://github.com/user-attachments/assets/1ea6607b-0706-4e4e-9e69-b1b3988e9e2f" />

```
grep "command" /usr/share/dict/word >> /root/command.txt
```


# 15.
<img width="490" height="183" alt="15" src="https://github.com/user-attachments/assets/b460110c-e30b-42a6-9d68-5fac72431553" />

```
#fdisk /dev/sda (To partition the Drive (/dev/sda) with partition editor tool called fdisk)
n (New partition)
p (Primary Partition)
Last: +512M (Sets the size of the partition. Leaving the "First sector" prompt at default and typing +512M here creates a partition that is exactly 512 Megabytes in size.)
t (Changes the partition's type ID)
82 (hexadecimal code for a Linux Swap partition)
w (write)
partprobe /dev/sda (reload the partition table without doing reboot)
mkswap /dev/sda1 (formatting the partition specifically for swap use)
vim /etc/fstab (contains a permanent list of all disk partitions and storage devices that the system should automatically mount at boot time.)
/dev/sda1 swap swap defaults 0 0 (tells Linux to mount the partition automatically when the computer boots up.)
swapon /dev/sda1 (Activating the Swap)
```
Check: swapon -s (Displays a summary table of all currently active swap spaces on your system)

**Note:**
This is the configuration line you added inside that file. It breaks down like this:
/dev/sda1: The device to mount.
swap: The mount point (swap doesn't use a standard folder path, just swap).
swap: The filesystem type.
defaults: Uses standard mount settings (read/write, auto-mount, etc.).
0 0: Controls backup dumps and filesystem integrity checks (both turned off for swap).

# 16. Search and store files
<img width="827" height="141" alt="image" src="https://github.com/user-attachments/assets/15f2d273-e8a7-4bea-9fc5-cf63403786bd" />

```
mkdir -p /root/filefound
find / -type f -user sarah -exec cp {} /root/filefound/ \;
```
Note:
**-exec ... \;**: This is an action flag. It tells **find** to execute an external command on every single file it successfully matches. The trailing escaped semicolon (\; or ';') is required to tell the shell where the command string ends.
**{}**: A placeholder used by find. For every file discovered, find replaces {} with the actual, full path of that file.

# 17.
<img width="1705" height="291" alt="image" src="https://github.com/user-attachments/assets/9ef78ce7-ab82-41cd-b979-41f26eaa96b7" />

```
# useradd geo
groupadd dba
passwd geo
visudo
```
write

Allow root to run any commands anywhere
geo ALL=(ALL) NOPASSWD: ALL

Same thing without a passwd
%dba ALL=(ALL) NOPASSWD: ALL

save and exit.

# 18.
Add a HDD Disk
```
fdisk /dev/sda ( Opens the partition table editor for the first storage drive (sda))
n
+512M
t
l
8e
w
partprobe /dev/sda (Forces the Linux kernel to read the new partition table without requiring a system reboo)
pvcreate /dev/sda1 (Initializes the newly created partition (/dev/sda1) as an LVM Physical Volume so LVM can use it.)
pvdisplay /dev/sda1 (Check properties of the new Physical Volume)
vgcreate -s 8M datastore /dev/sda1 (Creates a Volume Group named datastore using the physical partition. The -s 8M flag explicitly sets the Physical Extent (PE) size to 8 Megabytes (the default is usually 4M))
vgdisplay datastore
lvcreate -l 50 -n database datastore (Allocates 50 logical extents to create a Logical Volume named database inside the datastore group.)
lvdisplay datastore
mkfs.vfat /dev/datastore/database (Formats the new logical volume with the FAT32 (vfat) filesystem.)
mkdir -p /mnt/archive (Creates a target directory (mount point) named /mnt/archive to access the storage.)
blkid (Displays the unique attributes of block devices (like UUIDs))
vim /etc/fstab (Opens the system's file system table configuration file to add a line ensuring the disk mounts automatically on boot.)
systemctl daemon-reload (Reloads the systemd manager configuration so it recognizes changes made to the system files.)
mount -a
```


# 19.
<img width="1760" height="207" alt="image" src="https://github.com/user-attachments/assets/1d8405a0-91da-47e8-87ad-78ff0a5893e7" />


```
lsblk (Check current)
lvextend -L +300M /images
resize2fs /image (For xfs filesystem, use fatlabel for vfat file system)
lsblk (Check)
```


# 20.
<img width="939" height="352" alt="image" src="https://github.com/user-attachments/assets/a23c6f88-d1da-4545-a2b9-589dce0da85a" />

Corrected Command ManualServer 1 (NFS Server)bash# CHANGE: Use specific package instead of wildcard 'nfs*' to save space and reduce conflicts
yum install nfs-utils -y

```S1:
yum install nfs* -y
mkdir -p /remote/netuser1
echo "NFS Test" > /remote/netuser1/t1.txt
chmod 777 /remote/netuser1
vim /etc/exports (/remote/netuser1  *(rw,sync,no_root_squash))
systemctl enable --now rpcbind
systemctl enable --now nfs-server
exportfs -rav
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload

S2:
yum install autofs* -y
yum install nfs* -y
vim /etc/auto.master (/rhome    /etc/auto.rhome)
vim /etc/auto.rhome (netuser1   -rw   <SERVER1_IP>:/remote/netuser1)
systemctl enable --now autofs
ls /rhome
cd /rhome/netuser1
```
# 21.
<img width="1079" height="188" alt="image" src="https://github.com/user-attachments/assets/0030716a-c0b2-46e1-9687-30d455731292" />

Configuration of the Apache Server
```
yum install httpd* -y (Installs the Apache HTTP Server (httpd) and related packages)
vim /var/www/html/index.html (Write the content)
restorecon -Rv /var/www/html/ (Restores the correct SELinux security contexts recursively under the web document root.)
systemctl restart httpd.service (Stops and starts the Apache web server service to apply changes.)
curl http://localhost:80 (You can see the index.txt content on terminal)
```
Answer
```
vim /etc/httpd/conf/httpd.conf (Opens Apache's main configuration file for editing, such as changing the listening port.)
semanage port -a -t http_port_t -p tcp 82 (Adds TCP port 82 to the list of ports that SELinux allows Apache to use.)
semanage port -l | grep http_port_t ((Lists all SELinux ports assigned to the http_port_t type and filters the output.))
firewall-cmd --permanent --add-port=82/tcp (Permanently allows incoming TCP traffic on port 82 through the firewall.)
firewall-cmd --reload (Reloads the firewall configuration to apply the new rule.)
restorecon -Rv /var/www/html/ (Reapplies the correct SELinux labels to the web content directory.)
systemctl restart httpd.service (Restarts Apache so it begins listening on the newly configured port.)
systemctl reload  httpd.service (Reloads Apache's configuration without fully stopping the service.)
curl http://localhost:82 (To check)
```
# 22.
<img width="1338" height="195" alt="image" src="https://github.com/user-attachments/assets/a716505c-9b25-417f-95dc-9fab0f732e30" />

```
cd /usr/local/bin
vim mysearch
#!/bin/bash
mkdir -p /root/mysearch
find /usr -type f -size -10M -size +2M -perm 2000 -exec cp -p {} /root/mysearch/ \;
echo "Files are copied to /root/mysearch.""
mysearch (To run the script)
```

# 23.
<img width="1048" height="387" alt="WhatsApp Image 2026-07-07 at 11 23 07 PM" src="https://github.com/user-attachments/assets/f221a2c5-6c52-443e-9c35-7d717c1a0340" />

```
cp /etc/fstab /var/tmp/
chown root:root /var/tmp/fstab (To make any file under this directory will be owned by root user and belong to root)
chmod 644 /var/tmp/fstab (file should not be executable by anyone rw-r--r--)
setfacl -m u:natasha:rw /var/tmp/fstab (natasha will be able to read and write)
setfacl -m u:jerry:- /var/tmp/fstab (deny jerry all access)
getfacl /var/tmp/fstab (To check/verify changes)
```

# 24. Register our system

Two Ways: 
**1.Subscription-manager**
```
subscription-manager register
<username>
<password>
```
**2.rhc**
```
rhc connect
<username>
<password>
```
# 25. Add a user named ram and the user to the wheel group
```
useradd ram
usermod -G wheel 
id norm (To check)
```

# 26. Passwordless SSH
SSH Server Configuration
```
sudo dnf install openssh-server -y
sudo systemctl enable --now sshd
sudo systemctl status sshd
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
ip addr
```
SSH client configuration
```
ssh-keygen
ssh-copy-id username@server_ip
ssh username@server_ip
```
Note:
- By default root user login with SSH is disabled to check than run "grep -E "PermitRootLogin|PasswordAuthentication" /etc/ssh/sshd_config"
Solution:
Option 1: Root login enable
```
vi /etc/ssh/sshd_config
PermitRootLogin yes
PasswordAuthentication yes
systemctl restart sshd
```
Option 2: Use normal user (A normal user's ssh login permit stays yes

# 27. How to List and Kill Processes
```
top (To find)
k <pid> (To kill a process)
q (To exit top)
```
```
ps -ef (display a detailed snapshot of every currently running process on the system)
kill <option> <PID> 
```
Note:
kill -l (to see options of kill command)
2. SIGINT to interrupt your process
9. SIGKILL to forcefully kill process
15. SIGTERM to kill processes gracefully

# 28. Manage Services
```
systemctl status <service_name> (Displays the current status and runtime information of the specified service.)
systemctl stop <service_name> (Stops the specified service immediately.)
systemctl diasble <service_name> (Prevents the specified service from starting automatically at boot.)
systemctl enable <service_name> (Configures the specified service to start automatically at boot.)
systemctl reload <service_name> (Reloads the specified service's configuration without stopping or restarting it.)
```

# 29.Symbolic Links
**1. Soft Link** (Like shortcut in windows)
```
echo "this is a file" > test.txt (To Create a file first)
ls -l test.txt (You will see there is no link)
ln -s test.txt sltest.txt (To create soft link)
ls -l sltest.txt (To check anchor, the main file is anchor)
```
**2. Hard Links** (Additional name pointing directly to the data, It can survive the deletion of the original file, all hard linked files shares the same inode number)
```
ls -li file.txt (List inode information also)
ln file.txt ffile.txt (To create hard link)
ls -li f* (To check)
```


# 30. Disk Quota Configuration
Attach New Volume
```
fdisk /dev/nvme1n1
mkfs.ext4 /dev/nvme1n1p1
mkdir /data
mount /dev/nvme1n1p1 /data
vi /etc/mtab
```
Go to last line enter Esc+yy (To copy/yank)
Esc:n /etc/fstab
Esc+p (To paste)
Esc:wq
```
yum install quota
yum list installed quota
useradd a
useradd b
useradd c
useradd d
groupadd quota
usermod -g quota b
vi /etc/fstab
```
Write realtime, usrquota, grpquota
Esc:wq
```
systemctl daemon-reload
mount -o remount /data
mount | grep /data
passwd root
passwd a
chmod 777 /data
quotacheck -cug /data
edquota a
```
Blocks  Soft  Hard  indoes  Soft  Hard
0   100000  200000   0       10    20
```
quotaon /data
su - a
touch a.user{1..21}.txt
dd if=/dev/zero of=/data/file bs=1M count=300
```

**Groupquota:**
```
chgrp quota /data
edquota -g quota
```
Blocks  Soft  Hard  indoes  Soft  Hard
4   100000  200000   1       400    500
```
quotaoff /data
quotaon /data
sudo usermod -aG quota b
su - b
cd /data
touch abc{1..700}.txt
```

**Modify Grace Period for a specific user when they exceed their soft disk quota limits:**
```
edquota -T a (Remove the time and add 7 days)
quotaoff /data
quotaon /data
```

