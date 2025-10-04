# General Notes

Ansible is an open-source automation tool for configuration management, application deployment, orchestration, and provisioning. Below are structured, detailed notes.

## 1. Core Concepts

* **Agentless architecture**
  Runs over SSH or WinRM, no agent software required on managed nodes. Reduces attack surface and maintenance overhead.
* **Idempotency**
  Tasks always converge systems to the desired state. Re-running produces the same result if the system is already compliant.
* **Declarative style**
  Users describe the desired state, not the steps to reach it.
* **Push model**
  Control node pushes configurations to managed hosts.

## 2. Components

* **Control node**
  Machine where Ansible is installed and playbooks are executed.
* **Managed nodes (hosts)**
  Target machines that Ansible configures, requiring Python (for Linux) or PowerShell (for Windows).
* **Inventory**
  A list of managed nodes. Formats: INI, YAML, dynamic inventories via scripts, plugins, or cloud APIs.
* **Modules**
  Units of work (e.g., install a package, copy a file, start a service). Written in Python, but can be any executable returning JSON.
* **Plugins**
  Extend Ansible’s behaviour (e.g., connection plugins, callback plugins, lookup plugins).
* **Playbooks**
  YAML files defining automation workflows with tasks and roles.
* **Roles**
  Structured way to organise playbooks into reusable units (tasks, handlers, templates, files, vars, defaults).
* **Collections**
  Packaged modules, plugins, and roles, distributed via **Ansible Galaxy**.
* **Facts**
  System information collected by `setup` module (e.g., OS, IP addresses, CPU).

## 3. Architecture

1. **Inventory definition** → static (file) or dynamic (cloud APIs).
2. **Playbook execution** → sequential tasks applied to groups of hosts.
3. **Modules invoked** → executed on nodes via SSH/WinRM.
4. **Results gathered** → control node collects JSON outputs.
5. **Handlers triggered** → run only if notified by a task change.

## 4. Playbooks

* **Format**: YAML, indentation sensitive.

* **Structure**:

  ```yaml
  - name: Install and start Nginx
    hosts: webservers
    become: yes   # escalate privilege
    tasks:
      - name: Install Nginx
        apt:
          name: nginx
          state: present
      - name: Ensure Nginx is running
        service:
          name: nginx
          state: started
  ```

* **Key elements**:

  * `hosts`: target group.
  * `tasks`: actions.
  * `vars`: variables.
  * `handlers`: conditional actions on change.
  * `templates`: Jinja2 templates for dynamic file creation.

## 5. Execution Workflow

1. Define **inventory**.
2. Create **playbooks** or **ad-hoc commands**.
3. Run `ansible-playbook` against inventory.
4. Ansible establishes SSH/WinRM sessions.
5. Modules execute remotely, results sent back in JSON.

## 6. Variables and Templates

* **Variables** can come from:

  * Inventory (host/group vars).
  * Playbooks.
  * Extra-vars at runtime (`-e`).
  * Fact gathering.
* **Precedence** matters (extra-vars override all).
* **Templating** with Jinja2:

  ```jinja2
  server_name {{ inventory_hostname }};
  ```

## 7. Handlers

* Triggered when a task changes state.
* Example:

  ```yaml
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
    listen: restart nginx
  ```

## 8. Security

* **Vault**: encrypts sensitive data (passwords, keys).
* **SSH keys**: secure authentication.
* **Principle of least privilege**: use `become` only when necessary.

## 9. Orchestration

* Beyond configuration, can coordinate multi-node actions:

  * Rolling updates.
  * Blue/green deployments.
  * Cluster management.

## 10. Advantages

* Simple to learn (YAML, human-readable).
* No agents required.
* Large module ecosystem.
* Integrates with CI/CD pipelines.
* Scales to thousands of nodes.

## 11. Disadvantages

* **Performance**: slower than agent-based tools for very large infrastructures due to SSH overhead.
* **Error handling**: limited complex conditional flows compared to full scripting.
* **Python dependency**: required on Linux nodes.
* **State management**: not as robust as tools like Terraform for infra provisioning.

## 12. Best Practices

* Use **roles** and **collections** for modular design.
* Keep **inventory dynamic** via cloud provider plugins.
* Encrypt secrets with **Vault**.
* Enforce **idempotency** in all tasks.
* Use **linting tools** (`ansible-lint`) to maintain quality.
* Test with **Molecule**.

## Ansible Cheat Sheet

### Basics

```bash
ansible --version                   # Show Ansible version installed on the control node
ansible all -m ping -i inventory    # Ping all hosts defined in inventory file to test connectivity
ansible all -a "uptime"             # Run an ad-hoc command ("uptime") on all hosts
ansible-playbook playbook.yml       # Execute a playbook against inventory hosts
ansible-galaxy install <role>       # Download and install a reusable role from Ansible Galaxy
ansible-vault create secrets.yml    # Create a new encrypted file for secrets (opens editor)
ansible-vault edit secrets.yml      # Edit an encrypted file (requires vault password)
ansible-vault encrypt file.yml      # Encrypt an existing plaintext file
ansible-vault decrypt file.yml      # Decrypt an encrypted file back to plaintext
```

### Inventory File

It can be in INI format or YAML.

**INI format**

```ini
[webservers]                                  # Group name
web1 ansible_host=192.168.1.10 ansible_user=ubuntu   # Alias web1 points to host at 192.168.1.10 using user ubuntu
web2 ansible_host=192.168.1.11                # Alias web2 points to host at 192.168.1.11 (default user assumed)

[dbservers]                                   # Another group
db1 ansible_host=192.168.1.20                 # Alias db1 points to host at 192.168.1.20
```

**YAML format**

```yaml
all:
  hosts:
    web1:                                     # Alias web1
      ansible_host: 192.168.1.10              # Actual connection IP
    web2:                                     # Alias web2
      ansible_host: 192.168.1.11              # Actual connection IP
  children:
    dbservers:                                # Group of database servers
      hosts:
        db1:                                  # Alias db1
          ansible_host: 192.168.1.20          # Actual connection IP
```

Run with a custom inventory:

```bash
ansible-playbook -i inventory.ini site.yml    # Use inventory.ini explicitly when running a playbook
```

## CLI Commands

```bash
ansible --version                        # Show version
ansible all -m ping -i inventory.yml     # Ping all hosts listed in inventory
ansible webservers -m apt -a "name=nginx state=present" -b   # Install nginx via apt on all webservers with sudo
ansible dbservers -m service -a "name=mysql state=started" -b  # Ensure MySQL service started on dbservers
ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts"       # Copy file from control node to all hosts
ansible all -m file -a "path=/tmp/test state=directory"       # Ensure a directory exists
ansible all -a "uptime"                  # Run ad-hoc command "uptime" everywhere

ansible-playbook playbook.yml            # Run a playbook
ansible-playbook site.yml -C             # Dry run mode (check only, no changes made)
ansible-playbook site.yml -v             # Run with verbose output (-vvv for maximum detail)
ansible-playbook site.yml --limit web1   # Limit execution to only web1
ansible-playbook site.yml --step         # Step through tasks interactively
ansible-inventory -i inventory.yml --list  # Print expanded/parsed inventory

ansible-vault create secrets.yml         # Create new encrypted secrets file
ansible-vault edit secrets.yml           # Edit encrypted secrets
ansible-vault encrypt file.yml           # Encrypt existing file
ansible-vault decrypt file.yml           # Decrypt existing file
ansible-playbook site.yml --ask-vault-pass   # Run playbook with password prompt for vault
ansible-playbook site.yml --vault-password-file ~/.vault_pass.txt  # Run playbook using saved password file

#ad-hoc commands
ansible all -m ping                       # Ping all nodes
ansible webservers -m apt -a "name=nginx state=present" -b   # Install nginx on webservers
ansible dbservers -m service -a "name=mysql state=started" -b # Start MySQL on dbservers
ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts"       # Copy file to all
ansible all -m file -a "path=/tmp/test state=directory"       # Ensure directory exists
ansible all -m shell -a "uptime"          # Run uptime command
```

Listed above are the **core CLI commands** for ad-hoc runs, playbooks, inventory, and Vault. They are the most frequently used.

For **roles and collections**, there is an additional set of `ansible-galaxy` commands:

### 🔹 Role & Collection Commands

```bash
# Install all roles and collections defined in requirements.yml
ansible-galaxy install -r requirements.yml

# Install a specific role from Galaxy
ansible-galaxy role install geerlingguy.apache

# Install a role from GitHub
ansible-galaxy role install git+https://github.com/geerlingguy/ansible-role-docker.git

# List all installed roles
ansible-galaxy role list

# Remove a role
ansible-galaxy role remove geerlingguy.apache

# Init (scaffold) a new role with boilerplate structure
ansible-galaxy role init myrole

# Install a collection from Galaxy
ansible-galaxy collection install community.general

# Install a collection from Git repo or tarball
ansible-galaxy collection install git+https://github.com/ansible-collections/community.general.git

# List installed collections
ansible-galaxy collection list
```

### 🔹 Playbook Role Usage

Inside a playbook, roles are invoked like this:

```yaml
- name: Apply base and web roles
  hosts: all
  roles:
    - base
    - webserver
```

### 🔹 Role Project Scaffolding

When you create a role with:

```bash
ansible-galaxy role init myrole
```

It generates:

```
myrole/
├── defaults/main.yml
├── files/
├── handlers/main.yml
├── meta/main.yml
├── tasks/main.yml
├── templates/
├── tests/test.yml
└── vars/main.yml
```

### Playbook Structure

In playbooks, always target groups:

```yaml
- name: Example playbook                  # Descriptive playbook name
  hosts: webservers                       # Target group of hosts
  become: yes                             # Run with privilege escalation (sudo)
  vars:
    nginx_port: 80                        # Define variables inline
  tasks:
    - name: Install nginx                 # Task to install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Start nginx                   # Task to ensure nginx running and enabled
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Copy template                 # Deploy config template
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      notify: restart nginx               # Trigger handler on change

  handlers:
    - name: restart nginx                 # Handler definition
      service:
        name: nginx
        state: restarted
```

## Common Modules

### System

```yaml
apt:
  name: nginx
  state: present                          # Ensure nginx installed

yum:
  name: httpd
  state: latest                           # Ensure httpd installed with latest version

service:
  name: nginx
  state: restarted                        # Restart nginx
  enabled: yes                            # Ensure nginx enabled at boot

user:
  name: deploy
  state: present                          # Ensure user deploy exists
  groups: sudo                            # Add user to sudo group

group:
  name: admins
  state: present                          # Ensure group admins exists
```

### Files

```yaml
copy:
  src: file.txt
  dest: /tmp/file.txt                     # Copy file to remote

file:
  path: /tmp/dir
  state: directory                        # Create directory if absent

lineinfile:
  path: /etc/ssh/sshd_config
  regexp: '^PermitRootLogin'
  line: 'PermitRootLogin no'              # Ensure this line exists/updated in file

template:
  src: config.j2
  dest: /etc/app/config.ini               # Render Jinja2 template with vars
```

### Packages

```yaml
apt:
  name: nginx
  state: present                          # Install nginx with apt

pip:
  name: flask
  version: 2.0.1                          # Install specific version of flask with pip

npm:
  name: express
  global: yes                             # Install express globally with npm
```

### Archive/Transfer

```yaml
unarchive:
  src: app.tar.gz
  dest: /opt/
  remote_src: yes                         # Extract archive already on remote host

fetch:
  src: /etc/hosts
  dest: ./backup/                         # Download file from remote to control node
```

## Variables

```yaml
vars:
  app_port: 8080                          # Define variable inside playbook

tasks:
  - name: Example
    debug:
      msg: "App running on port {{ app_port }}"  # Use variable with Jinja2 syntax
```

**Variable precedence (lowest to highest)**

1. Defaults in roles                      # Lowest priority
2. Vars in roles/playbooks
3. Host/group vars
4. Extra vars (`-e`)                      # Highest priority

## Conditionals and Loops

```yaml
tasks:
  - name: Install nginx on Debian
    apt:
      name: nginx
      state: present
    when: ansible_os_family == "Debian"   # Conditional execution based on fact

  - name: Create multiple users
    user:
      name: "{{ item }}"
      state: present
    loop:
      - alice
      - bob
      - carol                             # Loop over list to create users
```

## Templates (Jinja2)

```jinja2
server {
  listen {{ nginx_port }};                # Insert variable value
  server_name {{ inventory_hostname }};   # Insert current host alias
}
```

## Facts

```yaml
- name: Print facts
  debug:
    var: ansible_facts                    # Dump all facts

- name: Print OS family
  debug:
    msg: "{{ ansible_facts['os_family'] }}"  # Access specific fact
```

Gathered facts: `ansible_hostname`, `ansible_distribution`, `ansible_memtotal_mb`, etc.

## Handlers

```yaml
tasks:
  - name: Update config
    template:
      src: config.j2
      dest: /etc/app/config.ini
    notify: restart app                   # Notify handler if changed

handlers:
  - name: restart app
    service:
      name: app
      state: restarted                    # Restart service only if notified
```

## Tags

```yaml
tasks:
  - name: Install packages
    apt:
      name: nginx
      state: present
    tags: install                         # Task tagged as "install"
```

Then:

```sh
ansible-playbook site.yml --tags install      # Run only install-tagged tasks
ansible-playbook site.yml --skip-tags install # Run all except install-tagged tasks
```

## Roles

Directory structure:

```
roles/
  webserver/
    tasks/main.yml                          # Role tasks
    handlers/main.yml                       # Role handlers
    templates/                              # Jinja2 templates
    files/                                  # Static files
    vars/main.yml                           # Role-specific variables
    defaults/main.yml                       # Default variable values
```

Usage:

```yaml
- hosts: webservers
  roles:
    - webserver                             # Apply role "webserver"
```

Encrypted vars:

```yaml
db_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256...              # Example of encrypted variable stored with Vault
```

## Directory setup

### General structure

```sh
ansible-demo/
├── ansible.cfg               # Core configuration
├── requirements.yml          # Collections & role dependencies
├── inventory/
│   ├── hosts.yml             # YAML inventory (preferred)
│   └── group_vars/
│       └── all.yml           # Global variables
├── playbooks/
│   └── site.yml              # Main orchestration playbook
├── roles/
│   └── base/                 # Example role
│       ├── tasks/
│       │   └── main.yml
│       ├── handlers/
│       │   └── main.yml
│       ├── defaults/
│       │   └── main.yml
│       ├── vars/
│       │   └── main.yml
│       ├── files/
│       │   └── motd.txt
│       ├── templates/
│       │   └── nginx.conf.j2
│       ├── tests/
│       │   └── molecule.yml
│       └── README.md
├── tests/                    # Integration tests (Molecule + Testinfra)
│   └── molecule.yml
├── .ansible-lint              # Linting rules
├── .yamllint                  # YAML style rules
├── .gitignore
└── README.md
```

Collections do **not** need to exist beforehand in your repo.

### How it works:

* You declare dependencies in `requirements.yml`.
* When you run:

  ```bash
  ansible-galaxy install -r requirements.yml
  ```

  Ansible will download and install those collections into the path defined in `ansible.cfg` (`collections_paths`).

### Two cases:

1. **Using community/public collections**

   * Example: `community.general`, `ansible.posix`.
   * These are pulled automatically from Ansible Galaxy when you install requirements.

2. **Using private/internal collections**

   * If your org maintains its own, you can point to a Git repo or tarball in `requirements.yml`.
   * Example:

     ```yaml
     collections:
       - name: git+https://github.com/org/internal-collection.git
     ```
   * In this case, the repo must exist, but you don’t ship the collection itself in your starter repo — only the pointer in `requirements.yml`.

### Best practice:

* Keep only `requirements.yml` in your repo.
* Never commit collections/ or roles/ installed from Galaxy → they are reproducible from requirements.yml.
* Always bootstrap by running `ansible-galaxy install -r requirements.yml` after cloning.
* Never commit Vault passwords → only keep encrypted vault files in repo.
* Keep group_vars/ and host_vars/ in repo, but exclude sensitive vault override files if you don’t want them in Git.
* Molecule test artefacts (.molecule/, .cache/) are excluded.

## Testing Locally

Use **containers** or **local VMs** as disposable targets for validating playbooks that manage packages, users, files, and services.

* Not for building golden images (use Dockerfiles for that).
* Only for **running and testing playbooks**.
* Cheap, isolated, repeatable environments.

### 1. Running Playbooks Against Docker Containers

* Requirement: Install `community.docker` collection.
* Inventory (`inventory/hosts.ini`):

```ini
[docker]
mycontainer ansible_connection=docker
```

* Example playbook (`playbooks/test.yml`):

```yaml
- hosts: docker
  gather_facts: true
  tasks:
    - name: Install curl
      package:
        name: curl
        state: present

    - name: Add test user
      user:
        name: devuser
        state: present
```

* Run:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/test.yml
```

!!!warning
    Works only if container has a package manager (Debian/Ubuntu/CentOS images). Alpine requires `apk` module.

### 2. Running Playbooks Against Local VMs

* Local VMs (VirtualBox, VMware, Multipass, etc.) expose SSH.
* Example Vagrant inventory:

```ini
[vms]
vm1 ansible_host=127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_private_key_file=.vagrant/machines/vm1/virtualbox/private_key
```

* Example playbook:

```yaml
- hosts: vms
  tasks:
    - name: Ensure Git installed
      package:
        name: git
        state: present
```

### 3. Running Playbooks Against KIND Nodes

* KIND = Kubernetes IN Docker. Nodes are containers.
* Option 1: Use `ansible_connection=docker` directly:

```ini
[kind]
kind-control-plane ansible_connection=docker
```

* Option 2: Install SSH in nodes and use as normal hosts (heavier).

Playbook example:

```yaml
- hosts: kind
  tasks:
    - name: Install net-tools
      package:
        name: net-tools
        state: present
```

### 4. Why This Approach

* **Containers and VMs are disposable** → test without risk.
* **Playbooks are portable** → same YAML can run on dev containers, local VMs, or production servers.
* **Debugging faster** → catch logic errors (loops, conditionals, handlers) before applying to production.

For **testing Ansible playbooks** (installing packages, adding users, copying configs), you need containers that:

1. Have a **full OS base** (not minimal scratch/distroless).
2. Include or support a **package manager** (apt, yum/dnf, apk).
3. Behave similarly to real hosts you’ll manage in production.

### 🔹 Recommended Container Images for Ansible Testing

| OS Family     | Container Image                           | Package Manager | Notes                                                                                           |
| ------------- | ----------------------------------------- | --------------- | ----------------------------------------------------------------------------------------------- |
| Debian/Ubuntu | `ubuntu:22.04`, `debian:12`               | `apt`           | Most common; good for general playbook testing.                                                 |
| CentOS/RHEL   | `rockylinux:9`, `almalinux:9`             | `dnf/yum`       | Good replacement for CentOS (now EOL). Matches RHEL-family.                                     |
| Fedora        | `fedora:39`                               | `dnf`           | Bleeding-edge, useful if testing modern modules.                                                |
| Alpine Linux  | `alpine:3.19`                             | `apk`           | Very small, but requires `apk` module instead of `apt/yum`. Not ideal for all tests.            |
| openSUSE      | `opensuse/leap:15`                        | `zypper`        | Useful if targeting SUSE-based systems.                                                         |
| KIND Nodes    | `kindest/node:v1.28.0` (default for KIND) | none by default | Doesn’t ship with package manager; use `ansible_connection=docker` OR inject packages manually. |

### 🔹 Best Practice

* For **general testing**:

  * Use **Ubuntu** (`ubuntu:22.04`) and **Rocky Linux** (`rockylinux:9`).
  * Covers both Debian-family and RHEL-family modules.

* For **minimal footprint testing**:

  * Use **Alpine** with `apk` → but remember not all roles/modules support it.

* For **multi-distro validation**:

  * Run Molecule with a **matrix** (Ubuntu, Rocky, Alpine) to ensure roles are portable.

### 🔹 Example: Inventory with Multiple Test Containers

```ini
[ubuntu]
ubuntu_test ansible_connection=docker ansible_docker_extra_args="--name ubuntu_test ubuntu:22.04"

[rocky]
rocky_test ansible_connection=docker ansible_docker_extra_args="--name rocky_test rockylinux:9"

[alpine]
alpine_test ansible_connection=docker ansible_docker_extra_args="--name alpine_test alpine:3.19"
```

👉 If your goal is just **basic role/playbook testing for packages + users**, stick to:

* `ubuntu:22.04` (Debian-family)
* `rockylinux:9` (RHEL-family)

That covers 95% of real-world production environments.

## Example

1. Start containers manually (or with a helper script).
2. Add them to your **inventory** as hosts with `ansible_connection=docker`.
3. Point your **playbook** at them instead of VMs or servers.

### Example: Inventory with Docker containers

```ini
[docker]
ubuntu_test ansible_connection=docker
rocky_test ansible_connection=docker
```

* `ubuntu_test` → points to a running `ubuntu:22.04` container.
* `rocky_test` → points to a running `rockylinux:9` container.
* `ansible_connection=docker` tells Ansible to execute inside the container, no SSH required.

### Example: Playbook

```yaml
- name: Test playbook on containers
  hosts: docker
  gather_facts: true
  tasks:
    - name: Install curl
      package:
        name: curl
        state: present

    - name: Add user
      user:
        name: devuser
        state: present

    - name: Deploy MOTD
      copy:
        content: "Managed by Ansible in {{ inventory_hostname }}"
        dest: /etc/motd
```

Run:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/test.yml
```

### How you’d use it

1. Spin up containers:

   ```bash
   docker run -dit --name ubuntu_test ubuntu:22.04 bash
   docker run -dit --name rocky_test rockylinux:9 bash
   ```
2. Run playbook → Ansible runs inside the containers.
3. Inspect results:

   ```bash
   docker exec -it ubuntu_test cat /etc/motd
   docker exec -it rocky_test id devuser
   ```

### Why this works

* Ansible doesn’t care if the target is a container or a VM — as long as a connection plugin (`docker` or `ssh`) works.
* By **reusing your playbooks** against containers, you can validate logic quickly before applying to “real” VMs or servers.

## Semaphore

Semaphore allows us to use Ansible with a web UI. Not only that, but it is a Modern UI and powerful API for Ansible, Terraform/OpenTofu/Terragrunt, PowerShell and other DevOps tools.

Run semaphore with:

```sh
task dev
```

Username: admin
Password: admin123

Some boilerplate playbooks can be found [Here](https://github.com/ChristianLempa/boilerplates/tree/main/ansible)
