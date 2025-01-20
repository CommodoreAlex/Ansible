Standard means of creating a HOSTS file with Ansible:
--

We define a hosts file with the operating system below:
```bash
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

The better practice is to avoid hardcoding credentials and use 'secrets management' or defer to using SSH keys as an alternative path:

Addressing the alternative path of using SSH keys:
--

1) Set up SSH keys between local machine and the remote hosts
2) Add the public key to the `~/.ssh/authorized_keys` file on the target hosts
3) Ensure that the private key is available on your local machine for Ansible to use

```bash
# Example hosts file using SSH keys
[web_servers]
server1 ansible_ssh_user=ubuntu
server2 ansible_ssh_user=ubuntu
```
