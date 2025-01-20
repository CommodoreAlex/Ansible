Setting up your Ansible situation, defer to the secure host configuration file in this repository.

Notice in these examples I am pointing toward a local hosts file using the `-i` switch.

Some of these commands are more ad-hoc commands, to gather information, make changes, or interact for generic reasons.

We can run modules in Ansible with the `-m` switch:
```bash
┌──(root㉿kali)-[/home/kali/ANSIBLE]
└─# ansible -i /home/kali/ANSIBLE/hosts linux -m ping                 

127.0.0.1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "changed": false,
    "ping": "pong"
}
```

We can run commands against Linux targets in ansible with the `-a` switch:
```bash
┌──(root㉿kali)-[/home/kali/ANSIBLE]
└─# ansible -i /home/kali/ANSIBLE/hosts linux -a 'cat /etc/passwd'  

127.0.0.1 | CHANGED | rc=0 >>
root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...
```

You can reboot targets remotely with:
```bash
┌──(root㉿kali)-[/home/kali/ANSIBLE]
└─# ansible -i /home/kali/ANSIBLE/hosts linux -a 'reboot'  
```

With going above ad-hoc tasks, we can defer to building out playbooks. 
--

**Playbooks** are YAML files in Ansible that define tasks for automation. They describe **what to do** on managed hosts, like installing packages, configuring files, or starting services. A playbook contains:

1. **Hosts**: Target systems to execute tasks on.
2. **Tasks**: Steps to perform (e.g., install software).
3. **Modules**: Specific actions (e.g., `apt`, `copy`).

You can name hosts in Ansible as:

- **Groups**: Logical names like `webservers` or `databases`.
- **Aliases**: Custom names with `ansible_host`.
- **IPs/Hostnames**: Directly use `192.168.1.10` or `server.example.com`.

```yaml
# This is an Ansible playbook in YAML to update NANO (.yaml or .yml)
---
  - name: Nano
    hosts: 127.0.0.1 # Could refer to 'linux' group in hosts file too
    tasks:
      - name: Ensure nano is there
        apt: # This could be 'yum' for centos or redhat
          name: nano
          state: latest
```

Playbooks make automation repeatable and scalable.

We can run our playbook:

```bash
┌──(root㉿kali)-[/home/kali/ANSIBLE]
└─# ansible-playbook -i /home/kali/ANSIBLE nanoupdate.yml 

PLAY [Nano] ******************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
[WARNING]: Platform linux on host 127.0.0.1 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of
another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [127.0.0.1]

TASK [Ensure nano is there] **************************************************************************************************************
fatal: [127.0.0.1]: FAILED! => {"cache_update_time": 1732908601, "cache_updated": false, "changed": false, "msg": "'/usr/bin/apt-get -y -o \"Dpkg::Options::=--force-confdef\" -o \"Dpkg::Options::=--force-confold\"       install 'nano=8.2-1'' failed: E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)\nE: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?\n", "rc": 100, "stderr": "E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)\nE: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?\n", "stderr_lines": ["E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)", "E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?"], "stdout": "", "stdout_lines": []}

PLAY RECAP *******************************************************************************************************************************
127.0.0.1                  : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
```

Notice that our Playbook failed due to lack of permissions. This incremented our 'failed=1' count.

We need to add the `become: yes` parameter to take advantage of SUDO (super user do) permissions. Reference the other document for HOSTS for more secure methods.
```yaml
# This is an Ansible playbook in YAML to update NANO (.yaml or .yml)
---
  - name: Nano
    hosts: 127.0.0.1 # Could refer to 'linux' group in hosts file too
    become: yes # Use sudo
    tasks:
      - name: Ensure nano is there
        apt: # This could be 'yum' for centos or redhat
          name: nano
          state: latest
```


Now we can run the playbook again successfully:
```bash
┌──(root㉿kali)-[/home/kali/ANSIBLE]
└─# ansible-playbook -i /home/kali/ANSIBLE nanoupdate.yml

PLAY [Nano] ******************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
[WARNING]: Platform linux on host 127.0.0.1 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of
another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [127.0.0.1]

TASK [Ensure nano is there] **************************************************************************************************************
changed: [127.0.0.1]

PLAY RECAP *******************************************************************************************************************************
127.0.0.1                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

Notice the next time, where it says 'changed=0'. Ansible will not make a change if the state is already there.

```bash
┌──(root㉿kali)-[/home/kali/ANSIBLE]
└─# ansible-playbook -i /home/kali/ANSIBLE nanoupdate.yml

PLAY [Nano] ******************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
[WARNING]: Platform linux on host 127.0.0.1 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of
another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [127.0.0.1]

TASK [Ensure nano is there] **************************************************************************************************************
ok: [127.0.0.1]

PLAY RECAP *******************************************************************************************************************************
127.0.0.1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

We could now work to remove the nano application by changing the state from 'latest' to 'absent':
```yaml
# This is an Ansible playbook in YAML to update NANO (.yaml or .yml)
---
  - name: Nano
    hosts: 127.0.0.1 # Could refer to 'linux' group in hosts file too
    become: yes # Use sudo
    tasks:
      - name: Ensure nano is not there
        apt: # This could be 'yum' for centos or redhat
          name: nano
          state: absent
```

This will remove the nano application from the target(s) systems.
