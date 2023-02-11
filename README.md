## Install

```shell
sudo apt update
sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
sudo apt install sshpass -y
ansible --version | grep "python version"
```

> ssh access
```shell
username=remi
ssh-keygen -t rsa -b 2048
ssh-copy-id $username@10.182.65.14
```

```shell
ansible-playbook -i hosts apt-update-upgrade.yml
```

## Ad-hoc Commands

```shell
ansible [target pattern] -m [module name] -a "[module arguments]"
# -m -> module name
# -a -> module arguments
```

> Gathering facts
```shell
ansible localhost -m setup
```

> command
- The default module 
- 'command' accepts free rom options
```shell
ansible -m command -a "ip a" all
ansible -m command -a "hostname" all
ansible            -a "hostname" all
ansible            -a "/bin/echo hello" all 
```

> ping
```shell
ansible all -m ping
ansible webservers -m ping
ansible all -m ping -u bruce # as bruce
ansible all -m ping -u bruce --become # as bruce, sudoing to root (sudo is default method)
ansible all -m ping -u bruce --become --become-user batman # as bruce, sudoing to batman
```

> copy
```shell
ansible atlanta -m copy -a "src=/etc/hosts dest=/tmp/hosts"
```

> service
```shell
ansible webservers -m service -a "name=httpd state=started"
```

## Tasks
- The task is the basic building block in ansible
- There is not a single rule on how the tasks should be written down, it all depends on the modules you'd like to use and what's the intended use-case. 


### Modules

Instead of making Ansible run loads of bash commands, what you should be doing is using modules

> `copy`   copy file or content 

> `get_url`   download file 

> `file`   manage file/directories 

> `yum`   manage package 

> `service`   manage services 

> `firewalld`   firewall service 

> `lineinfile`   add a line to dest file 

> `template`   to template file with variables 

> `debug`  to debug and display 

> `add_host`  add host to inventory while play|

> `wait_for`  use for flow control  

> `apt`   manage apt-packages 

> `shell`  execute shell commands on targets 

### Handlers

When we want to perform an action under certain cirmumstances

> `notify`  to notify the handler 

> `handlers`  define handler 

### Task Control and Loops

> `with_items`  then “item” inside action 

> `with_nested`  for nested loops

> `with_file` 

> `with_fileglob`   

> `with_sequence` 

> `with_random_choice` 

> `when`  meet a condition 
 
## Playbook

- A playbook is a [list] of plays we are pushing to our hosts described in the Inventory. 
- The play is a [list] of related tasks.
- Each play in a playbook must begin with the two main items: `hosts` and `tasks`, any additional tags (name, vars, become, ...) are optional

```shell
ansible-playbook –i <INVENTORY> <YAML>
ansible-playbook <YAML> # Run on all hosts defined 
ansible-playbook <YAML> -f 10 # Fork - Run 10 hosts parallel 
ansible-playbook <YAML> --verbose # Verbose on successful tasks 
ansible-playbook <YAML> -C # Test run 
ansible-playbook <YAML> -C -D # Dry run 
ansible-playbook <YAML> -l <host> # Limit to run on single host 
```

> Simple Playbook 
```yaml
---
- name: "Simple Playbook"
  hosts: webservers
  become: yes
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    apt: name=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running (and enable it at boot)
    service: name=httpd state=started enabled=yes
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```

> Set basic network settings

```yaml
---
- name: "Set basic network settings"
  hosts: all
  become: yes
  tasks:
    - name: "Set hostname to AnsibleFTW"
      hostname:
        name: ansibleftw
    - name: "Set DNS servers"
      net_system:
        name_servers:
          - 8.8.8.8
          - 8.8.4.4
```

> File operations

```yaml
---
- name: "File Operations"
  hosts: all
  tasks:
    - name: "Make sure the file exist with touch command"
      file:
        path: /tmp/testAnsibleFile
        state: present
    - name: "Use echo command to add data to the file"
      shell: "echo 'DATA DATA DATA DATA' >> /tmp/testAnsibleFile"
    - name: "Create a directory to host the file in"
      file:
        path: /tmp/testAnsibleDir
        state: directory
    - name: "Copy the file into our new directory"
      copy:
        remote_src: yes
        src: /tmp/testAnsibleFile
        dest: /tmp/testAnsibleDir/copyOfAnsibleTestFile
```

> Repos and Packages
```yaml
---
- name: "Repos and Packages"
  hosts: all
  tasks:
    - name: "Create new repo"
      yum_repository:
        name: epel
        description: EPEL YUM repo
        baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/
    - name: "Install googler package from EPEL repo"
      yum:
        name: googler
        enablerepo: epel
        state: present
```


## Roles

main file in sub-directories should be main.yml

Role variable can define under **roles** directive

**Role Directories**

| Item  | Description |
| ------------- | ------------- |
| defaults | default value of role variables |
| files | static files referenced by role tasks |
| handlers | role’s handlers |
| meta | role info like Author, Licence, Platform etc |
| tasks | role’s task defenition |
| templates | jinja2 templates |
| tests | test inventory and test.yml |
| vars | role’s variable values |
| pre_tasks | tasks before role |
| post_tasks | tasks after role |

## Inventory (static or dynamic)

```shell
ansible --list-hosts all
ansible --list-hosts "*"
ansible --list-hosts agents
ansible --list-hosts \!agents # everything other than agents
ansible --list-hosts "ag*"
ansible --list-hosts agents[0] # index selector
ansible -i inventory --list-hosts all
```

defalut inventory
```shell
cat /etc/ansible/hosts
```
hosts file
```yaml
[agents]
10.99.0.1 
10.99.0.2 
10.99.0.3 
10.99.0.4 

[servers]
10.99.0.50

[controllers]
10.99.0.60 ansible_connection=local

[all:vars]
ansible_connection=ssh
ansible_user=remi
ansible_ssh_pass=remi1234
ansible_sudo_pass=remi1234
```


```shell
[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com

[webservers]
www[01:50].example.com

badwolf.example.com:5309

jumper ansible_port=5555 ansible_host=192.0.2.50


[databases]
db-[a:f].example.com

[all:vars]
ansible_user=vagrant	
ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key

```

## Variables and Facts

- Ansible Facts
collecting the host specific data

```yaml
{{ ansible_facts["eth0"]["ipv4"]["address"] }}
or
{{ ansible_facts.eth0.ipv4.address }}
```
- Inventory variables

```yaml
[agents]
10.99.0.1 
10.99.0.2 
10.99.0.3 
10.99.0.4 

[servers]
10.99.0.50
10.99.0.60

[all:vars]
ansible_connection=ssh
ansible_user=remi
ansible_ssh_pass=remi1234
ansible_become_password=remi1234
ansible_python_interpreter=/usr/bin/python3
```

- Playbook variables

```yaml
`vars` 
`vars_files`
```

```yaml
- hosts: all
  vars:
    ansible_connection: network_cli
  vars_files:
    - FILENAME
  tasks:
    - name: Execute show run on the switch
      arubaoss_command:
        commands: "{{ansible_connection}}"
  become: yes
  become_user: apache
```

```yaml
---
- name: "Install pkgs on specific OS only"
  hosts: all
  become: yes
  tasks:
    - name: "install http pkgs on Red Hat"
      yum:
        name: packages
        state: present
      vars:
        packages:
          - httpd
          - httpd-tools
      when: ansible_facts['os_family'] == "Red Hat"
    - name: "install vim on CentOS"
      yum:
        name: packages
        state: present
      vars:
        packages:
          - vim
          - php
          - mysql
      when: ansible_facts['os_family'] == "CentOS"
```    

- group_vars
directory for group variable files 

- hosts_vars
directory for host variable files  

- Special variables
  - Magic
  - Facts
  - Connection variables

| Item  | Description |
| ------------- | ------------- |
| `register` | registered variables |
| `include_vars` | module |
| `include_tasks: stuff.yml` | include a sub task file |



## Parallelism

| Item  | Description |
| ------------- | ------------- |
| `forks` | number of forks or parallel machines|
| `--forks` | when using ansible-playbook |
| `serial` | control number parallel machines |
| `async: 3600` | wait 3600 seconds to complete the task |
| `poll: 10` | check every 10 seconds if task completed |
| `wait_for` |module to wait and check  if specific condition met |
| `async_status` | module to check an async task status |





## Configuration File

At minimum, Ansible uses two files - both available in a template form under `/etc/ansible` - the configuration file (asnible.cfg) and an inventory file (hosts). 

```shell
cat /etc/ansible/ansible.cfg # Default file
```
or you can create a 'ansible.cfg' at your local directory

