

How to Create a Secure Ansible Hosts File: SSH, Vault, and Environment Variables
--

Below is an example of a hosts file with a simple method; using hard-coded credentials:
```yaml
# Operating system:
[linux]

# Target IPs:
192.168.1.1

# Addressing variables for hard-coded user accounts (not ideal)
# Should not be the root user; example only.
[linux:vars]
ansible_user=root # Should not be root, for example purposes
ansible_password=Password123
# better practice is to avoid hardcoding a password and use secrets management.
```

### **Better Practices for Authentication in Ansible**

It is highly recommended to avoid hardcoding credentials, as this can lead to security vulnerabilities. Instead, use **secrets management** tools or opt for **SSH keys** as a more secure and efficient alternative.

#### **Why Avoid Hardcoding Credentials?**

Hardcoding passwords or other sensitive information in your configuration files, such as the `hosts` file, increases the risk of exposing credentials. This practice is vulnerable to unauthorized access, especially when repositories are shared or exposed to multiple people.

Addressing the alternative path of using SSH keys:
--

1) Set up SSH keys between local machine and the remote hosts
2) Add the public key to the `~/.ssh/authorized_keys` file on the target hosts
3) Ensure that the private key is available on your local machine for Ansible to use

Example hosts file using SSH keys:
```yaml
[linux]
127.0.0.1

[linux:vars] # No password necessary
ansible_user=kali
```
Generating an SSH key pair:
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
````

- **Options Explained:**
    - `-t rsa`: Specifies the RSA key algorithm.
    - `-b 4096`: Creates a 4096-bit key for stronger encryption.
    - `-f ~/.ssh/id_rsa`: Saves the key to `~/.ssh/id_rsa` (default location).
- When prompted for a passphrase, press **Enter** twice to leave it empty for passwordless login.

Copy the Public Key to the target machine.

Use the `ssh-copy-id` command to copy your public key to the target machine (`127.0.0.1`):
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub root@127.0.0.1
```
- Enter the password for the `kali` user when prompted.

Verify Key-Based Authentication and test connection using SSH:
```bash
ssh root@127.0.0.1
```
- If successful, it should log in without asking for a password.

Since we edited our Ansible `hosts` file (removed the hardcoded password), the SSH key will now handle authentication.

You can test this by running the Ansible ping module to test:
```bash
ansible -i /home/kali/ANSIBLE/hosts linux -m ping
```

You should see a successful response:
`127.0.0.1 | SUCCESS => {     "changed": false,     "ping": "pong" }`

Ansible Vault (for Secrets Management):
--

If you need to use passwords (e.g. for sudo access or service accounts), Ansible Vault allows you to encrypt sensitive information, such as passwords, in a secure and manageable way.
* You can encrypt your hosts file or create a separate encrypted file for passwords.
* Use `ansible-vault` to create or edit encrypted files.

Example:
```bash
ansible-vault create passwords.yml
```

Inside of the passwords file you can have:
```yaml
ansible_ssh_pass: "your_password"
```

You can then reference your password file in your playbook:
```yaml
- name: Ensure the service is running
  service:
    name: apache2
    state: started
  become: yes 
  become_method: sudo 
  vars_files:
    - passwords.yml
```

To run a playbook with Ansible Vault, you'll need to provide the Vault password:
```bash
ansible-vault playbook.yml --ask-vault-pass
```

Taking advantage of Environment Variables:
--

If you prefer not to hardcode passwords, you can also use environment variables to pass sensitive data. Ansible will automatically detect environment variable like "ANSIBLE_PASSWORD" and "ANSIBLE_SUDO_PASS" for connection and privilege escalation:
```bash
export ANSIBLE_PASSWORD='your_password'
export ANSIBLE_SUDO_PASS='your_sudo_password'
```

Then in your playbook, you can reference those variables without storing them in your hosts file. Keep in mind, Ansible will automatically read the environment variables without you needing to reference them in your playbook. You can use the `vars` section or in the `ansible_ssh_pass` and `ansible_sudo_pass` variables.

Example playbook where environment variables are automatically picked up:
```yaml
---
- name: Deploy and configure the web server
  hosts: web_servers
  gather_facts: yes
  become: yes

  tasks:
	- name: Install Apache web server
	  apt:
	    name: apache2
	    state: present

	- name: Ensure Apache is running
	  service:
	    name: apache2
	    state: started
	    enabled: yes
```

If you want to reference the environment variables explicitly within the playbook, you can use the `lookup` plugin to read them:
```yaml
---
- name: Deploy and configure web server
  hosts: web_servers
  gather_facts: yes
  become: yes

# Using the 'lookup' plugin to read them:
  vars:
    ansible_ssh_pass: "{{ lookup('env', 'ANSIBLE_PASSWORD') }}"
    ansible_sudo_pass: "{{ lookup('env', 'ANSIBLE_SUDO_PASS') }}"

  tasks:
    - name: Install Apache web server
      apt:
        name: apache2
        state: present
        
	- name: Ensure Apache is running
	  service:
	    name: apache2
	    state: started
	    enabled: yes
```

In this example:

- `lookup('env', 'ANSIBLE_PASSWORD')` will fetch the environment variable `ANSIBLE_PASSWORD` and assign it to `ansible_ssh_pass`.
- Similarly, `lookup('env', 'ANSIBLE_SUDO_PASS')` will fetch the sudo password from the environment and assign it to `ansible_sudo_pass`.

One of the caveats to doing this is you're making it more clear to cyber threat actors as to what environment variables are high-value targets. 
Run the Playbook:
---

When you run the playbook, Ansible will automatically use the environment variables for authentication:

```bash
ansible-playbook playbook.yml
```
Ansible Configuration File (`ansible.cfg`):
---

You can also store connection-related settings in the `ansible.cfg` file, but it's still better to use SSH keys or Ansible Vault for security.

Example of an Ansible configuration file:
```ini
[defaults]
inventory = ./hosts
private_key_file = /path/to/your/private_key
```

It is always better to avoid hardcoding passwords into your hosts file; unless in a controlled testing environment. 

Use SSH keys, Ansible Vault, environment variables, or configuration files for a more secure and manageable approach. 
