Setting Up Your Ansible Environment
In this section, we'll walk you through setting up your Ansible environment and show you a few ways to interact with your systems. Please refer to the secure host configuration file in this repository for a detailed setup.

For the examples provided here, we’ll be pointing to a local hosts file using the -i switch. This allows you to specify which hosts are included for any Ansible commands you execute.

We’ll also cover some ad-hoc commands, which are useful for gathering information, making quick changes, or performing general operations. These commands can be executed on the fly without creating a playbook.

Running Ad-Hoc Commands
You can execute ad-hoc commands with the -m switch to run specific modules. Here’s an example of using the ping module to check connectivity to a target system:

bash
Copy
Edit
┌──(root㉿kali)-[/home/kali/ANSIBLE]
└─# ansible -i /home/kali/ANSIBLE/hosts linux -m ping

127.0.0.1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "changed": false,
    "ping": "pong"
}
In this example, we used the ping module to confirm that the target host is reachable.

Next, we can run a command on the target systems by using the -a switch, which allows you to execute arbitrary commands remotely. For example, to view the contents of /etc/passwd:

bash
Copy
Edit
┌──(root㉿kali)-[/home/kali/ANSIBLE]
└─# ansible -i /home/kali/ANSIBLE/hosts linux -a 'cat /etc/passwd'

127.0.0.1 | CHANGED | rc=0 >>
root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...
You can also reboot a target system remotely with:

bash
Copy
Edit
┌──(root㉿kali)-[/home/kali/ANSIBLE]
└─# ansible -i /home/kali/ANSIBLE/hosts linux -a 'reboot'
Transitioning from Ad-Hoc Commands to Playbooks
While ad-hoc commands are useful for quick tasks, Playbooks provide more structured and repeatable automation. Playbooks are written in YAML and specify a series of tasks to run on target hosts. They describe what to do on your managed systems, such as installing packages, configuring files, or starting services.

Each playbook contains the following components:

Hosts: The systems targeted for the tasks.
Tasks: The steps to execute (e.g., install software).
Modules: The specific actions to be carried out (e.g., apt, copy).
Here’s a simple example of a playbook to ensure that nano is installed on the target system:

yaml
Copy
Edit
# This is an Ansible playbook in YAML to update NANO (.yaml or .yml)
---
  - name: Nano
    hosts: 127.0.0.1 # Could refer to 'linux' group in hosts file too
    tasks:
      - name: Ensure nano is there
        apt: # This could be 'yum' for centos or redhat
          name: nano
          state: latest
Running a Playbook
Once your playbook is ready, you can run it using the ansible-playbook command. Here’s how you would run the nanoupdate.yml playbook:

bash
Copy
Edit
┌──(root㉿kali)-[/home/kali/ANSIBLE]
└─# ansible-playbook -i /home/kali/ANSIBLE nanoupdate.yml

PLAY [Nano] ******************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
[WARNING]: Platform linux on host 127.0.0.1 is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of
another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [127.0.0.1]

TASK [Ensure nano is there] **************************************************************************************************************
fatal: [127.0.0.1]: FAILED! => {"cache_update_time": 1732908601, "cache_updated": false, "changed": false, "msg": "'/usr/bin/apt-get -y -o \"Dpkg::Options::=--force-confdef\" -o \"Dpkg::Options::=--force-confold\"       install 'nano=8.2-1'' failed: E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)\nE: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?\n", "rc": 100, "stderr": "E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)\nE: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?"}
In the output, you’ll notice the playbook failed because it doesn’t have permission to install the package. The error message suggests running as root to resolve the issue.

To fix this, add become: yes to the playbook, which allows Ansible to execute tasks with elevated (sudo) privileges:

yaml
Copy
Edit
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
Now, you can run the playbook again successfully:

bash
Copy
Edit
┌──(root㉿kali)-[/home/kali/ANSIBLE]
└─# ansible-playbook -i /home/kali/ANSIBLE nanoupdate.yml

PLAY [Nano] ******************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [127.0.0.1]

TASK [Ensure nano is there] **************************************************************************************************************
changed: [127.0.0.1]

PLAY RECAP *******************************************************************************************************************************
127.0.0.1                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
Notice the changed=1 in the output, meaning that the playbook successfully installed the package. The next time you run it, you’ll see changed=0 because the package is already up to date.

Removing a Package with Ansible
If you want to remove a package instead of installing it, simply set the state to absent in your playbook:

yaml
Copy
Edit
# This is an Ansible playbook in YAML to remove NANO (.yaml or .yml)
---
  - name: Nano
    hosts: 127.0.0.1 # Could refer to 'linux' group in hosts file too
    become: yes # Use sudo
    tasks:
      - name: Ensure nano is not there
        apt: # This could be 'yum' for centos or redhat
          name: nano
          state: absent
Running this playbook will remove the nano application from your target systems.

By using these techniques, you can automate repetitive tasks efficiently with Ansible and handle both ad-hoc commands and playbooks as needed for your infrastructure management.
