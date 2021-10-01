# How to Navigate This Project's Folder Structure

To best understand how to use these files, it's important to first have a basic understanding of how Ansible works. There are three main concetps you should know:

- Ad-Hoc Commands: Used at the command line, these are commands that you type and send all in one line on the command-line interface. They are incredubly useful for in-the-moment information gathering or applying intervantions quickly across a set of devices. 
- Playbooks: Playbooks are codified versions of ansible commands. They can be thought of as an analogue to scripts and make your actions easily re-usable. You can also use playbooks to call one or more roles (more on those below)
- Roles: Roles are essentially packages of commands that you can use when you want to create a set of actions to accomplish a task. If playbooks are scripts, you can think of roles as a library that you can call from a playbook to make certain tasks easier to accomplish. 


# Playbooks 
The root of the `kgs` folder cotains a number of `.yml` files; These are Playbooks! The title should be fairly descriptive, but be sure to read through them to understand what they do.
Many of these playbooks call other `roles`. 

## Playbook Example
As an example, `add_kgs_users.yml` has the following code:

```---
- hosts: kgs
  become: yes

  roles:
  - kgs_groups
  - kgs_lab_user_access
```

- The `hosts` variable is set to `kgs` which means all computers in the `kgs` group in your hosts file (`/etc/ansible/hosts`) will be targeted. You could also target a single computer by replacing `kgs` with the name like `seele` or `lev`. 
- `become` means that this play will run as the root user - this is why it's important to have your ssh key saved in `/root/.ssh/authorized_keys` on all of the workstations you support!
- `roles:` indicated that we're using content from the `ansible-playbooks/kgs/roles/` directory. To see what is happening in a given role, navigate to /ansible-playbooks/kgs/roles/NameOfRole/tasks/main.yml`. The next section will take a look at the kgs_groups role. 

# Roles
Roles let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure. After you group your content in roles, you can easily reuse them and share them with other users.

## Role Example - kgs_groups

Let's take a look at the content in our `kgs_groups` role. The tasks folder is a good place to start, so let's view the contents of `ansible-playbooks/kgs/roles/NameOfRole/tasks/main.yml`

```
---
# tasks file for kgs_groups
- name: Ensure group "research-computing_sni-vcs" exists
  group:
    name: research-computing_sni-vcs
    gid: 2514
    state: present
  ignore_errors: true

# Excluding [research-computing_sni-vcs-wandell] and [research-computing_sni-vcs-kalanit] until I can figure out how to create a group name with >32 characters!
# Update - this was a bad idea. As it turns out, while these are a prohibited length, they have been added over the years by directly editing the /etc/group file, and thereby overcoming the 32 character limit in groupadd. I've switched to lineinfile to accommodate this oddity. 
- name: Ensure /etc/group contains 'research-computing_sni-vcs-wandell'
  lineinfile:
   path: /etc/group
   regexp: '(research-computing_sni-vcs-wandell)'
   state: absent
  check_mode: yes
  changed_when: false
  register: wandellvcs

- debug:
   msg: "Yes, 'research-computing_sni-vcs-wandell' exists in /etc/group."
  when: wandellvcs.found

- debug:
   msg: "No, 'research-computing_sni-vcs-wandell' is NOT in /etc/group."
  when: not wandellvcs.found

- name: Add the 'research-computing_sni-vcs-wandell' to /etc/group
  lineinfile:
   path: /etc/group
   state: present
   line: 'research-computing_sni-vcs-wandell:x:2515:'
  when: not wandellvcs.found

- name: Ensure /etc/group contains 'research-computing_sni-vcs-kalanit'
  lineinfile:
   path: /etc/group
   regexp: '(research-computing_sni-vcs-kalanit)'
   state: absent
  check_mode: yes
  changed_when: false
  register: kalanitvcs

- debug:
   msg: "Yes, 'research-computing_sni-vcs-kalanit' exists in /etc/group."
  when: kalanitvcs.found

- debug:
   msg: "No, 'research-computing_sni-vcs-kalanit' is NOT in /etc/group."
  when: not kalanitvcs.found

- name: Add the 'research-computing_sni-vcs-kalanit' to /etc/group
  lineinfile:
   path: /etc/group
   state: present
   line: 'research-computing_sni-vcs-kalanit:x:2563:'
  when: not kalanitvcs.found

- name: Ensure group "research-computing_sni-vcs-fmri" exists
  group:
    name: research-computing_sni-vcs-fmri
    gid: 2542
    state: present
  ignore_errors: true

- name: Ensure group "fmri" exists
  group:
    name: fmri
    gid: 31
    state: present
  ignore_errors: true

- name: Ensure group "docker" exists
  group:
    name: docker
    state: present
  ignore_errors: true

- name: Ensure group "oak_kalanit" exists
  group:
    name: oak_kalanit
    state: present
  ignore_errors: true
```

This role uses a series of tasks to add user groups to the target system. This content makes sense for a role, becuase it's something that you may want to include in other playbooks as a check to make sure that the target system is already configured with the correct groups. Examples of such scenarios include:

- When adding users to the lab, you need the correct user groups before you add the user accounts, or else new users would be created without proper access. 
- When configuring Native Oak, you need a user group called `oak_kalanit` before users can access the share, this role ensures it's there.
- Access to OakZFS can be hindered without the `fmri` group, the kgs_groups role ensures it's present. 

# Takeaways
Ad-Hoc Commands, Playbooks and Roles can do much more than what is shown here. Here are a few resources you can check out to learn more:

- Ad-Hoc Commands: https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html
- Playbooks: https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html
- Roles: https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html
