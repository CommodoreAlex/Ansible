
# Offline Installation of Ansible for Red Hat-based Systems  

This guide will show you how to create an offline installation of **Ansible** for a Red Hat-based system (e.g., RHEL or CentOS) in an offline environment.  
## Prerequisites  - A Red Hat-based Linux host (RHEL, CentOS, Fedora, etc.) in an offline environment.

- A Red Hat-based Linux host (RHEL, CentOS, Fedora, etc.) in an offline environment. 
- A second machine with internet access to download the necessary packages. 
- Access to both systems with **sudo** privileges.

- ---  
## Steps to Create an Offline Installation  
### 1. Download Ansible Package and Dependencies 

On a machine with internet access, we will first download the Ansible package and all necessary dependencies.  
#### a. Enable EPEL Repository  

Ansible is available in the **EPEL** (Extra Packages for Enterprise Linux) repository. Ensure that the EPEL repository is enabled on the machine with internet access.  
```bash
sudo yum install -y epel-release
```
#### b. Download Ansible and Dependencies

Next, use `yum` to download the Ansible package and its dependencies. The `yumdownloader` tool can help with this:

1. Install `yum-utils` if not already installed:

```bash
sudo yum install -y yum-utils
```

2. Download the Ansible package and its dependencies:
```bash
sudo yum install --downloadonly --downloaddir=/path/to/your/folder ansible
```

This will download the necessary `.rpm` files for Ansible and its dependencies into the specified directory (replace `/path/to/your/folder` with the actual directory where you want to store the files).
#### c. Copy the Downloaded Files

Once the download is complete, copy the downloaded `.rpm` files to a portable storage medium (e.g., USB drive or external hard drive).
```bash
cp /path/to/your/folder/*.rpm /path/to/usb-drive/
```

---
### 2. Install Ansible in the Offline Environment

Now, on the offline system, we will install Ansible using the copied `.rpm` files.

#### a. Transfer Files to the Offline System

Transfer the `.rpm` files from your portable storage to the offline system (e.g., by connecting the USB drive).

#### b. Install the Dependencies

On the offline system, install the downloaded `.rpm` files using the following command:
```bash
sudo rpm -ivh /path/to/usb-drive/*.rpm
```

This will install Ansible and its dependencies from the `.rpm` files you copied.

#### c. Resolve Any Missing Dependencies

If there are any missing dependencies or errors during installation, you can resolve them using `yum`:
```bash
sudo yum localinstall /path/to/usb-drive/*.rpm
```

This will automatically handle any missing dependencies from the local `.rpm` files.

---
### 3. Verify the Installation

After the installation is complete, verify that Ansible is installed by checking its version:
```bash
ansible --version
```

You should see output displaying the version of Ansible that was installed, for example:
```bash
ansible 2.9.6   config file = /etc/ansible/ansible.cfg   configured module search path = ['/usr/share/ansible']   python version = 3.8.5 (default, Jul 20 2020, 12:20:09) [GCC 8.4.0]
```

---
## Troubleshooting

- If you encounter issues during installation, ensure that all required `.rpm` files were transferred, including dependencies.
- If you see errors related to missing dependencies after using `rpm`, run `yum localinstall` to automatically resolve missing dependencies from the `.rpm` files.
- Ensure that any required repositories (like EPEL) are available in the offline environment, either through local mirrors or other means.

---
## Conclusion

With Ansible successfully installed in your offline Red Hat-based environment, you can now use it for automation and configuration management on your target systems. If you need to install additional packages or update Ansible, repeat the process for any necessary files.
