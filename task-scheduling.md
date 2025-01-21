# **Automating Patch Management Cycles with Cron and Windows Task Scheduler**

Automating patch management ensures systems are consistently updated with minimal manual intervention. This guide provides system administrators with step-by-step instructions on scheduling Ansible patch management playbooks for both Linux and Windows systems using **Cron** and **Windows Task Scheduler**.

If you’re looking for detailed configurations of patch management playbooks themselves, such as setting up secure host files or tailoring playbooks for Linux and Windows environments, refer to the other files in this repository. Those files will help you establish a solid foundation before diving into automation.

By following this guide, you'll ensure your patching process is automated, efficient, and reliable—whether for weekly updates, critical fixes, or security patches.

---
## **What You'll Learn**

- How to schedule Ansible playbooks for patch management on Linux systems using Cron.
- How to configure Windows Task Scheduler to automate playbooks for patching Windows systems.
- Best practices for recurring automation to maintain secure and up-to-date environments.

---
## **Prerequisites**

Before scheduling your patch management playbooks:

1. Ensure your Ansible control node (machine where Ansible is installed, where you run commands to manage other systems) and target systems (managed nodes) are set up and configured.
2. Test your patch management playbooks to confirm they work as expected.
3. Store your playbooks in a reliable, accessible location (e.g., Git repository or local directory).

If you need help with initial setup or securing your Ansible environment, refer to other files in this repository for detailed guidance.

---
## **Scheduling Playbooks with Cron on Linux**

Cron is a time-based job scheduler for Unix-like operating systems. You can use it to execute your Ansible playbooks at regular intervals.
### **Steps to Schedule a Playbook with Cron**

1. **Open the Crontab File**

Use the `crontab` command to edit the Cron jobs for the current user:
```bash
crontab -e
```

This opens the crontab in your default editor.

2. **Set Required Environment Variables**  

Cron runs in a limited environment. Add necessary variables (e.g., `PATH`, `ANSIBLE_CONFIG`) at the top of the crontab file to avoid issues:
```bash
ANSIBLE_CONFIG=/path/to/ansible.cfg
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

This ensures the playbook can locate Ansible and other dependencies.

3.  **Add an Entry for the Playbook**

Use the following format to schedule your playbook:
```bash
<minute> <hour> <day_of_month> <month> <day_of_week> ansible-playbook -i /path/to/hosts /path/to/playbook.yml
```

For example, to run the playbook every Sunday at 2 AM:
```bash
0 2 * * 0 ansible-playbook -i /etc/ansible/hosts /home/admin/patch_linux.yml >>
/var/log/ansible_patch.log 2>&1
```

- **Log Outputs**: Always redirect output (`>>`) to a log file for troubleshooting and monitoring.
- **Error Handling**: Use `2>&1` to include error messages in the same log file.

4. **Save and Exit**

Once the entry is added, save the crontab file. The Cron service will automatically pick up the changes.

5. **Test Before Scheduling**

Before relying on automation, run the playbook manually to confirm there are no errors:
```bash
ansible-playbook -i /etc/ansible/hosts /home/admin/patch_linux.yml
```

If successful, verify the scheduled job’s execution by temporarily setting a short interval (e.g., every 5 minutes).

6. **Verify Job Execution**  
Use a temporary Cron entry to check the environment variables and confirm proper execution:
```bash
* * * * * env > /path/to/env_output.txt
```

Inspect `/path/to/env_output.txt` to ensure required variables like `PATH` and `ANSIBLE_CONFIG` are present. After testing, remove the entry.

---
## **Scheduling Playbooks with Windows Task Scheduler**

Windows Task Scheduler allows you to automate tasks, including running Ansible playbooks, on a recurring schedule.
### **Steps to Schedule a Playbook with Windows Task Scheduler**

1. **Open Task Scheduler**
   - Press `Win + S`, type **Task Scheduler**, and open it.

2. **Create a New Task**
   - Click **Create Task** on the right-hand panel.
   - Give the task a descriptive name (e.g., "Ansible Patch Management").

3. **Set the Trigger**
   - Go to the **Triggers** tab and click **New**.
   - Set the schedule (e.g., weekly, daily) and the start time.
   - Click **OK** to save the trigger.

4. **Set the Action**
   - Go to the **Actions** tab and click **New**.
   - In the **Action** field, select **Start a Program**.
   - In the **Program/script** field, enter the path to the Python executable (e.g., `C:\Python39\python.exe`).
   - In the **Add arguments** field, specify the command to run your playbook:
```
-m ansible.playbook -i C:\path\to\hosts C:\path\to\playbook.yml
```

5. **Configure Additional Settings**
   - In the **Conditions** tab, uncheck "Start the task only if the computer is on AC power" to ensure the task runs on laptops even when on battery.
   - In the **Settings** tab, enable "Run task as soon as possible after a scheduled start is missed" for resiliency.

6. **Test the Task**
   - Right-click the task and select **Run** to test its functionality.

---
## **Best Practices for Scheduling Patch Management**

- **Time Selection**: Schedule patching during maintenance windows to minimize disruptions.
- **Staggered Runs**: For large environments, stagger playbook execution times to reduce resource contention.
- **Notification Systems**: Configure email alerts or monitoring tools to notify administrators of task success or failure.
- **Backup Plans**: Always have recent backups or snapshots before running patch updates.

---

Automating your patch management playbooks with Cron or Windows Task Scheduler streamlines system maintenance and ensures consistent updates. By following the steps outlined here, you can maintain a secure and resilient infrastructure with minimal manual effort. For additional details on setting up your environment or configuring playbooks, explore other files in this repository. 
