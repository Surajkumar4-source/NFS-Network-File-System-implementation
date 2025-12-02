# NFS Setup Implementation : Server and Client Configuration.

## Introduction

### *The Network File System (NFS) is a protocol that enables file sharing across a network. It allows a server to share directories and files with clients, enabling centralized storage and seamless access.*

## Goal of This Implementation

1. The server will share a directory (/mnt/suraj_server) that contains files accessible by specific clients in the network.

2. The clients will mount this shared directory and use it as if it’s a local directory.
   
3. This setup is ideal for shared storage in environments like web servers, centralized backups, or file sharing among multiple machines.

  <br>
  
## Prerequisites

### 1. Virtual Machines
- Two Virtual Machines (VMs):
    - NFS Server: This VM will host the shared directory.
    - NFS Client: This VM will access the shared directory.

### 2. Network Configuration
- Use two network adapters for each VM:
    - NAT Adapter:
      - Used to access the internet for package installations.
    - Bridge Adapter:
      - Used for communication between the server and client within the same local network.

### 3. Operating System
  - Both VMs we use Ubuntu 22.04 or a similar Linux distribution. Ensure the OS is installed and updated.


### Network Configuration in the VM Manager

1. Configure Network Adapters in the VM Manager
   1. Open your VM settings (e.g., in VirtualBox or VMware).
   2. Configure the following adapters:

    - Adapter 1: NAT (for internet access).
    - Adapter 2: Bridge Adapter (for LAN communication).

### 2. Check Network Details
   - On both the server and client:

 *Verify IP addresses assigned by the bridge adapter.*

```yml
ip a  or ifconfig
```

  - *Look for the bridge adapter network interface (e.g., enp0s8) and note its IP.*
  
     - *For example in my case:*
    
  - Server IP: 192.168.82.100
  - Client IP: 192.168.82.101


<br>
<br>


## Server-Side Setup

### 1. Install NFS Server Package
   - To host a shared directory, the NFS server software must be installed.

```yml
sudo apt update
sudo apt install -y nfs-kernel-server
```
  - *sudo apt update: Updates the system’s package index to ensure the latest       
     package versions are available.*
  - *sudo apt install -y nfs-kernel-server: Installs the NFS server software. The -y      flag auto-confirms the installation prompts.*






### 2. Create the Shared Directory
  - Create a directory on the server to store shared files.

```yml
sudo mkdir -p /mnt/suraj_server
sudo chmod 777 /mnt/suraj_server
```
  - *mkdir -p: Creates the directory /mnt/nfsdata. The -p ensures the parent directories are created if they don’t exist.*

  - *chmod 777: Sets full read, write, and execute permissions for all users. This is suitable for testing; in production, stricter permissions should be used.*

    
### 3. Configure NFS Exports
  - Specify the directories to share and the clients allowed to access them.

```yml
sudo nano /etc/exports
```
  - */etc/exports: The configuration file where NFS shares are defined.*
  - *Add the following line to share the directory with all machines in the subnet        192.168.82.0/24: (My LAN Network)*

```yml
/mnt/suraj_server 192.168.82.0/24(rw,sync,no_subtree_check)  OR

/mnt/suraj_server 192.168.82.0/24(rw,no_subtree_check,insecure,no_root_squash)

```
  - *rw: Enables read and write access for clients.*
  - *sync: Ensures data is written to disk before the server replies to the client.*
      *This prevents data corruption.*
  - *no_subtree_check: Disables subtree checking, improving performance and 
     reliability.*
      *Save and close the file.*

### 4. Export the Shared Directory
  *Apply the export configuration to make the directory available.*

```yml
sudo exportfs -rav
```
  - *exportfs: Configures NFS to export (share) directories defined in /etc/exports.*
    -r: Re-exports all entries in /etc/exports.
    -a: Exports all directories.
    -v: Displays verbose output for debugging.

### 5. Start and Enable NFS Server
  - *Ensure the NFS server is running and starts automatically on reboot.*

```yml
sudo systemctl start nfs-server
sudo systemctl enable nfs-server
sudo systemctl status nfs-server
```
  - systemctl start: Starts the NFS server service.systemctl enable: Enables the NFS   - service to start automatically at boot.
  - systemctl status: Displays the current status of the NFS server.

### 6. Configure Firewall (Optional)
Allow NFS traffic through the server’s firewall.
#### *For ufw (Uncomplicated Firewall):*

```yml
sudo ufw allow from 192.168.82.0/24 to any port nfs
```
####  *For firewalld:*

```yml
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --reload
```
  *These commands allow NFS, mount daemon, and RPC bind services through the firewall.*
  
### 7. Verify the NFS Exports
- Check the current shared directories and their configurations.

```yml
sudo exportfs -v
```
  - *Displays a list of directories shared by NFS and their permissions.*





<br>
<br>




## Client-Side Setup
<br>

### 1. Install NFS Client Package
  - Install the client software to access NFS shares.

```yml
sudo apt update
sudo apt install -y nfs-common
```
  *nfs-common: Contains essential utilities for mounting NFS shares.*
  
### 2. Create a Mount Point
  - Create a directory to mount the shared NFS directory.

```yml
sudo mkdir -p /mnt/suraj_client
```
  - */mnt/suraj_client: This will be the access point for the shared directory on 
     the client.*
    
### 3. Mount the NFS Share
  - Mount the shared directory from the server. Replace <server_ip> with the server’s IP address.

```yml
sudo mount <server_ip>:/mnt/suraj_server /mnt/suraj_client
```
  - *mount <server_ip>:/mnt/suraj_server: Specifies the NFS server and directory to 
     mount.*
  - */mnt/suraj_client: Specifies the local directory to mount the share.*


### 4. Verify the Mount

  - Check if the NFS share is mounted correctly.

```yml
df -h
```

  - *Displays the file systems and their mount points. Look for the NFS share.*


### 5. Auto-Mount on Boot (Optional)
  - Configure the NFS share to mount automatically after reboot.

```yml
sudo nano /etc/fstab
```

  - Add this line:

```yml
<server_ip>:/mnt/suraj_server /mnt/suraj_client nfs defaults 0 0
```

  - *This entry ensures the NFS share mounts during system boot.*


### 6. Test the NFS Share

  - Create a file on the client and check if it appears on the server.
    On the client:

```yml
sudo touch /mnt/nfsdata/testfile
```

## Check On the server:
```yml
ls /mnt/nfsdata
```
  - *If the file testfile appears on the server, the setup works correctly.*


### 7. Unmount the NFS Share (Optional)
  - To unmount the NFS share:

```yml
sudo umount /mnt/nfsdata
```

### 8. Troubleshooting Tips

  - Check Server Status:

    - sudo systemctl status nfs-server

  - Verify Mount on Client:

```yml
mount | grep nfs
```
  - Check Logs for Errors:
```yml
journalctl -xe
```

  - Ensure Correct Firewall Configuration:
      - Verify that NFS traffic is allowed through firewalls on both the server and 
       client.



<br>
<br>
<br>

##  Conclusion

*This implementation provides a comprehensive approach to setting up NFS for shared storage. The NFS server shares directories, and clients access them as local directories. This setup is useful in networks where multiple systems need access to common files. Let me know if you need further clarifications! 🎉*














<br>
<br>
<br>
<br>



**👨‍💻 𝓒𝓻𝓪𝓯𝓽𝓮𝓭 𝓫𝔂**: [Suraj Kumar Choudhary](https://github.com/Surajkumar4-source) | 📩 **𝓕𝓮𝓮𝓵 𝓯𝓻𝓮𝓮 𝓽𝓸 𝓓𝓜 𝓯𝓸𝓻 𝓪𝓷𝔂 𝓱𝓮𝓵𝓹**: [csuraj982@gmail.com](mailto:csuraj982@gmail.com)





<br>


