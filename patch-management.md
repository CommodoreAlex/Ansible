# Patching Systems with Ansible: Linux and Windows

Welcome to this guide on using **Ansible** for automating patch management on **Linux** and **Windows** systems. This page focuses on creating and configuring patching playbooks to ensure your infrastructure stays secure and up-to-date. You'll explore how to effectively use Ansible modules, customize patching for different environments, and handle key differences between system types.
### What You'll Learn on This Page

In this guide, you will learn:

- **How to Configure Patching Playbooks**: Write Ansible playbooks for Linux distributions like Ubuntu, RHEL, CentOS, and Windows systems to apply updates, including security patches.
- **Understanding Ansible Modules**: Gain insights into modules like `apt`, `yum`, and `win_updates` to manage updates and understand how their parameters differ based on system requirements.
- **Handling Post-Patch Reboots**: Automate reboots after applying updates to ensure patches take effect, especially for kernel or critical updates.

> ⚠️ **Important**:  
> This guide focuses solely on creating patching playbooks. For guidance on initial setup, securing your `hosts` file, or scheduling patch management cycles using **cron** (Linux) and **Task Scheduler** (Windows), please refer to other files in this repository.

----
## **Ansible Task Modules Overview**

In the context of managing patching on both Linux and Windows systems using Ansible, several task modules are commonly used. Here's a breakdown of these modules for understanding their purpose and how to use them:

### **1. `apt` (for Debian-based Systems like Ubuntu, Debian)**

The `apt` module is used for managing packages and updates on Debian-based systems. It provides a straightforward way to update package repositories, install, and upgrade packages, and remove them if needed.
#### Key Parameters:

- `update_cache`: Ensures the package cache is updated before installing packages.
- `upgrade`: Upgrades all installed packages to the latest version. You can set it to `dist` to perform a distribution upgrade.
- `only_upgrade`: Ensures that only packages that are already installed get upgraded.
- `cache_valid_time`: Specifies how long the cached data should be considered valid.
#### Example Usage:
```yaml
- name: Update all packages on Ubuntu/Debian
  apt:
    update_cache: yes
    upgrade: dist
```

---

### **2. `yum` (for Red Hat-based Systems like RHEL, CentOS, Fedora)**

The `yum` module manages packages on Red Hat-based systems. For newer versions of Fedora, `dnf` is the default, but `yum` is backward compatible.

#### Key Parameters:

- `name`: The package or pattern to manage. Use `'*'` to indicate all packages.
- `state`: Desired state of the package (e.g., `latest` to upgrade to the latest version, `absent` to remove the package).
- `security`: (RHEL/CentOS) Ensures that only security-related updates are applied.
#### Example Usage:
```yaml
- name: Update all packages on RHEL/CentOS/Fedora
  yum:
    name: '*'
    state: latest
```

---

### **3. `win_updates` (for Windows Systems)**

The `win_updates` module manages Windows updates. You can use this module to install specific categories of updates, check for pending updates, or even reboot the system if updates require it.
#### Key Parameters:

- `category_names`: A list of update categories to install (e.g., `SecurityUpdates`, `CriticalUpdates`).
- `reboot`: If set to `yes`, the system will automatically reboot after updates if required.
- `state`: Defines the update state (e.g., `searched` to check for updates, `installed` to install updates).
#### Example Usage:
```yaml
- name: Install all Windows updates
  win_updates:
    category_names:
      - SecurityUpdates
      - CriticalUpdates
    reboot: yes
```

---

### **4. `reboot` (for Both Linux and Windows Systems)**

The `reboot` module is used to reboot a system after a patching or update process. Rebooting may be required for updates to take effect, especially when kernel or critical system components are updated.

#### Key Parameters:

- `reboot_timeout`: Defines the time (in seconds) to wait for the reboot to complete.
- `test_command`: A command to test whether the system is up after rebooting (e.g., `whoami`).

#### Example Usage:
```yaml
- name: Reboot the machine after updates
  reboot:
    reboot_timeout: 600
    test_command: whoami
```

---

## ## **## **Patching Linux Systems with Ansible**

Ansible can automate the patching process for Linux systems (e.g., RHEL, CentOS, Ubuntu). The typical approach is to use Ansible to manage the installation of updates, including security patches.

### **1. Use `apt` for Debian-based Systems (Ubuntu, Debian)**

For systems that use `apt` (Debian-based systems like Ubuntu), the following playbook will update all packages:

#### Example Playbook for Ubuntu/Debian:
```yaml
---
- name: Update all packages on Ubuntu/Debian
  hosts: ubuntu_servers
  become: yes  # Use sudo for privileged commands
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Upgrade all packages
      apt:
        upgrade: dist
```

### **2. Use `yum` for Red Hat-based Systems (RHEL, CentOS, Fedora)**

For RHEL-based systems (including CentOS and Fedora), you can use `yum` (or `dnf` for newer versions of Fedora) to apply updates:

#### Example Playbook for RHEL/CentOS/Fedora:
```yaml
---
- name: Update all packages on RHEL/CentOS/Fedora
  hosts: rhel_servers
  become: yes  # Use sudo for privileged commands
  tasks:
    - name: Update all packages
      yum:
        name: '*'
        state: latest
```

### **3. Apply Security Updates Only**

To only install security updates, you can modify the playbook to target security updates specifically:
#### Example Playbook for Ubuntu/Debian Security Updates:
```yaml
---
- name: Apply only security updates on Ubuntu/Debian
  hosts: ubuntu_servers
  become: yes
  tasks:
    - name: Apply security updates
      apt:
        upgrade: dist
        only_upgrade: yes
        cache_valid_time: 3600
```
#### Example Playbook for RHEL/CentOS Security Updates:
```yaml
---
- name: Apply only security updates on RHEL/CentOS
  hosts: rhel_servers
  become: yes
  tasks:
    - name: Apply security updates
      yum:
        name: '*'
        security: yes
        state: latest
```

### **4. Reboot After Patching**

Sometimes, after applying updates, especially kernel updates, a reboot is necessary. Ansible can handle this for you as well:

#### Example Reboot Task:
```yaml
---
- name: Reboot server after updates
  hosts: all
  become: yes
  tasks:
    - name: Reboot the machine if necessary
      reboot:
        reboot_timeout: 600
        test_command: whoami
```

---

## **Patching Windows Systems with Ansible**

For Windows systems, you can use the `win_updates` module to install patches.

### **1. Update All Windows Patches**

You can use Ansible to trigger a patch update on all Windows systems.

#### Example Playbook for Windows:
```yaml
---
- name: Update all Windows patches
  hosts: windows_servers
  tasks:
    - name: Install all Windows updates
      win_updates:
        category_names:
          - SecurityUpdates
          - CriticalUpdates
        reboot: yes  # Automatically reboot if required
```

### **2. Update Specific Categories of Patches**

You may want to install only specific categories of patches, such as security updates or critical updates.

#### Example for Security and Critical Updates:
```yaml
---
- name: Apply security and critical updates on Windows
  hosts: windows_servers
  tasks:
    - name: Install security updates only
      win_updates:
        category_names:
          - SecurityUpdates
        reboot: yes
```

### **3. Check for Pending Updates**

You can also use Ansible to check whether there are pending updates that need to be installed on Windows systems.

#### Example for Checking Pending Updates:
```yaml
---
- name: Check for pending updates on Windows
  hosts: windows_servers
  tasks:
    - name: Get Windows update information
      win_updates:
        state: searched
      register: result

    - name: Show pending updates
      debug:
        var: result
```

### **4. Reboot Windows Systems After Updates**

If patches require a reboot to complete the installation, Ansible can automate this process:
```yaml
---
- name: Reboot Windows after updates
  hosts: windows_servers
  tasks:
    - name: Reboot the machine
      win_reboot:
        reboot_timeout: 600
        test_command: whoami
```

---
### Wrapping Up

You've now learned how to create and configure Ansible playbooks for patch management on both Linux and Windows systems. By understanding the nuances of modules like `apt`, `yum`, and `win_updates`, you can confidently automate updates and reboots, ensuring your systems remain secure and up-to-date.

Patch management is a critical aspect of maintaining a reliable and resilient infrastructure. The techniques and best practices covered here are designed to simplify your workflow and reduce manual effort while minimizing risks.

> 🚀 **Next Steps**:  
> If you're looking to extend your automation further:

- Visit other files in this repository to set up your environment, including secure `hosts` file configurations.
- Learn how to automate recurring patch cycles using **cron** jobs on Linux or **Task Scheduler** on Windows.
- Explore advanced features in Ansible, such as tags, notifications, and custom scripts, to enhance your patch management strategy.

Feel free to adapt these playbooks to suit your organization's needs, and remember—regular testing and monitoring are key to ensuring your patching process is seamless and effective.

