# 🚀 Provision VM with Vagrant and Deploy an SFTP Server Using Ansible

## **Table of Contents**
1. [Introduction](#introduction)
2. [Features](#features)
3. [Prerequisites](#prerequisites)
4. [Directory Structure](#directory-structure)
5. [How to Use](#how-to-use)
   - [Clone the Repository](#clone-the-repository)
   - [Provision the VM](#provision-the-vm)
   - [Expected Output](#expected-output)
6. [Test the SFTP Server](#test-the-sftp-server)
   - [Option 1: Username/Password Authentication](#option-1-usernamepassword-authentication)
   - [Option 2: SSH Key-Based Authentication](#option-2-ssh-key-based-authentication)
7. [Clean Up](#clean-up)

---

### 📋 Introduction

Project provides solution for provisioning a virtual machine (VM) and deploying a secure SFTP server using Vagrant and Ansible. The setup supports both password and SSH key-based authentication.

### ⚙️ **Features**

- **Automated VM Provisioning**:
  - The VM is provisioned with Vagrant and configured with a private network IP (`192.168.56.101`).
  - Ansible is installed on the VM during provisioning.

- **SFTP Server Setup**:
  - Installs and configures OpenSSH server via Ansible to enable SFTP functionality.
  - Creates an SFTP user (`SFTPUSR`) with:
    - Password authentication (`SFTPUSRpass`).
    - SSH key-based authentication using the provided public key (`id_rsa.pub`).

### ✅ Prerequisites

-   **Vagrant**: [Installation Guide](https://www.vagrantup.com/downloads)
-   **VirtualBox**: [Installation Guide](https://www.virtualbox.org/wiki/Downloads)
-   **Generate SSH Keys**: Before you begin, generate a new SSH key pair in the **`vagrant\ansible\sftp`** directory.
    ```bash
    ssh-keygen -t rsa -b 4096 -f id_rsa -N ''
    ```

### 📂 Directory Structure

  ```plaintext
  vagrant/
  ├── ansible/
  │   ├── sftp/
  │   │   ├── Vagrantfile                  # Vagrant configuration file, defines the VM configuration, including network settings, synced folders, and the Ansible provisioner.
  │   │   ├── inventory.yml                # Ansible inventory file, here targets the localhost (the VM itself) for provisioning.
  │   │   ├── playbook.yml                 # Ansible main playbook that that applies the sftp role to the target host to configure the SFTP server.
  │   │   ├── id_rsa                       # SSH private key (need to generate)
  │   │   ├── id_rsa.pub                   # SSH public key (need to generate)
  │   │   ├── roles/
  │   │   │   ├── sftp/
  │   │   │   │   ├── tasks/
  │   │   │   │   │   ├── main.yml         # Ansible tasks to  install and configure the SFTP server
  │   │   │   │   ├── templates/
  │   │   │   │   │   ├── sshd_config.j2   # A Jinja2 template for the SSH daemon configuration.
  ```

### 🖥️ **How to use**

1. **Clone the Repository**:
  Clone the repository to your local machine:
    ```bash
    git clone <repository-url>
    cd vagrant/ansible/sftp
    ```

2. **Provision the VM**:
  Run the following command to provision the VM and configure the SFTP server:
    ```bash
    vagrant up
    ```

3. **Expected output**:

      ```plaintext
        PLAY [SFTP Server Setup] *******************************************************

        TASK [Gathering Facts] *********************************************************
        ok: [localhost]

        TASK [sftp : Install OpenSSH Server] *******************************************
        ok: [localhost]

        TASK [sftp : Create SFTP group] ************************************************
        changed: [localhost]

        TASK [sftp : Create SFTP user] *************************************************
        changed: [localhost]

        TASK [sftp : Create .ssh directory for SFTP user] ******************************
        changed: [localhost]

        TASK [sftp : Add SSH public key for SFTP user] *********************************
        changed: [localhost]

        TASK [sftp : Create directory structure] ***************************************
        changed: [localhost] => (item={'path': '/data/SFTPUSR', 'owner': 'root', 'group': 'sftp_users', 'mode': '0755'})
        changed: [localhost] => (item={'path': '/data/SFTPUSR/upload', 'owner': 'SFTPUSR', 'group': 'sftp_users', 'mode': '0755'})

        TASK [sftp : Configure SSH daemon for SFTP] ************************************
        changed: [localhost]

        TASK [sftp : Restart SSH service] **********************************************
        changed: [localhost]

        PLAY RECAP *********************************************************************
        localhost                  : ok=9    changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ```

### 🤖 **Test the SFTP Server**

1. **Option 1: Username/Password Authentication**:
    - Connect to the SFTP server using the following command:
      ```bash
      sftp -P 22 SFTPUSR@192.168.56.101
      ```
    - Enter the password when prompted:
      ```bash
      <secret-pass>
      ```
    - Test file upload:
      ```bash
      sftp> cd upload/
      sftp> put inventory.yml
      ```
    - Expected output:
      ```bash
      Uploading inventory.yml to /upload/inventory.yml
      inventory.yml 100%   63     7.9KB/s   00:00
      ```

2. **SSH Key-Based Authentication**:
    - Connect to the SFTP server using the private key:
      ```bash
      sftp -i ./id_rsa -P 22 SFTPUSR@192.168.56.101
      ```
    - Test file upload:
        ```bash
      sftp> cd upload/
      sftp> put playbook.yml
      ```
  - Expected output:
      ```bash
      Uploading playbook.yml to /upload/playbook.yml
      playbook.yml 100%   83    10.4KB/s   00:00
      ```

### ♻️ **Clean Up**

- **To destroy the VM and clean up resources, run:**:
    ```bash
    vagrant destroy -f
    ```

### 📋 **Notes:**

  - The VM in the current project is configured with a default private network IP address: **`192.168.56.101`**.  
    **If another device or VM in your network is already using `192.168.56.101`, the VM provisioning will fail due to an IP conflict.**  
    Similarly, if you are using a different subnet, the default IP may not work. Please verify it on your side before proceeding with the setup.

    To resolve this, feel free to update the IP address according to your needs:
    1. In the `Vagrantfile`, open it and locate the following line:
      ```ruby
      config.vm.network "private_network", ip: "<your-ip-address-here>"
      ```
    2. Also during command execution as needed:
      ```ruby
      sftp -P 22 SFTPUSR@<your-ip-address-here>
      ```
      ```ruby
      sftp -i ./id_rsa -P 22 SFTPUSR@<your-ip-address-here>
      ```
---
