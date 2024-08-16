# Ansible Tutorial
This step-by-step tutorial will give you a basic understanding of how to use Ansible following [Best Practices](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html#directory-layout).

To keep the tutorial simple and easy to follow, it demonstrates how to manage the local host(127.0.0.1) as the target with Shellsible, rather than a remote host.

## Install Ansible(Example)

```
cd /vagrant
git clone https://github.com/ansible/ansible.git
cd /vagrant/ansible
sudo yum upgrade -y
sudo yum install -y python3.12
sudo yum install -y python3.12-pip
sudo yum install -y sshpass
python3.12 -m pip install --user -r ./requirements.txt
source ./hacking/env-setup
sudo ln -s /usr/bin/python3 /usr/bin/python
ansible --version
```

* The installation method used by 'env-setup' differs from the formal installation process.
* 'sshpass' is used to authorize SSH connections.

## Test SSH Login
Confirm whether you can log in to the local host as the root user.
```
ssh root@127.0.0.1
```

## Playbook(minimum)

### Make Palybook
```
cat << "EOF" > playbook.yml
---
- name: hello, world
  hosts:
    - 127.0.0.1
  vars:
    ansible_user: root
    ansible_password: vagrant

  tasks:
  - name: test
    debug:
      msg: "hello, world!"
EOF
```

### Execute
```
ansible-playbook playbook.yml
```

### Result
```
PLAY [hello, world] ************************************************************

TASK [Gathering Facts] *********************************************************
ok: [127.0.0.1]

TASK [test] ********************************************************************
ok: [127.0.0.1] => {
    "msg": "hello, world!"
}

PLAY RECAP *********************************************************************
127.0.0.1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

* Below is a demonstration of how to use Ansible following 'Best Practices.'

## Playbook(Best Practices)

### Make Directories
```
mkdir group_vars
mkdir host_vars
mkdir roles
mkdir roles/common
mkdir roles/common/tasks
mkdir roles/web
mkdir roles/web/tasks
mkdir roles/web/handlers
mkdir roles/web/templates
mkdir roles/web/files
mkdir roles/web/vars
mkdir roles/web/meta
```

As a result, you can see the following output.
```
.
├── group_vars
├── host_vars
├── roles
│     ├── common
│     │     ├── tasks
│     ├── web
│           ├── tasks
│           ├── handlers
│           ├── templates
│           ├── files
│           ├── vars
│           ├── meta
```


### Make Common Role
```
cat << EOF > roles/common/tasks/main.yml
---
- name: test
  debug:
    msg: "hello, common!"
EOF
```
* Sample for debug module.
* You can write any message for 'name: ' tag.
* You can define multiple modules inside 'main.yml'.
* '---' indicates that this file is written in YAML format.

### Make Web Role
```
cat << EOF > roles/web/tasks/main.yml
---
- name: test
  debug:
    msg: "hello, world!"
EOF
```

### Make 'webservers.yml'
```
cat << EOF > webservers.yml
---
- hosts: webservers
  roles:
    - common
    - web
EOF
```

- 'webservers.yml' is defined to have Ansible execute the common and web roles.

### Make 'site.yml'
```
cat << EOF > site.yml
---
- import_playbook: webservers.yml
EOF
```
- 'site.yml' is the top-level configuration file.

### Make Inventory
```
cat << EOF > inventory.yml
---
all:
  vars:
    ansible_user: root
  children:
    webservers:
      hosts:
        host1:
          ansible_host: 127.0.0.1
          ansible_password: vagrant
EOF
```
- The host, user, and password are defined.
- The host(host1) belonging to 'webservers' is defined.

### Directories
As a result, you can see the following output.

```
.
├── group_vars
├── host_vars
├── roles
│     ├── common
│     │     ├── tasks
│     │           ├── main.yml
│     ├── web
│           ├── tasks
│           │     ├── main.yml
│           ├── handlers
│           ├── templates
│           ├── files
│           ├── vars
│           ├── meta
├── webservers.yml
├── site.yml
├── inventory.yml
```

### Execute
```
ansible-playbook -i inventory.yml site.yml
```

### Result
```
PLAY [webservers] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [host1]

TASK [common : test] ***********************************************************
ok: [host1] => {
    "msg": "hello, common!"
}

TASK [web : test] **************************************************************
ok: [host1] => {
    "msg": "hello, world!"
}

PLAY RECAP *********************************************************************
host1                      : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Variables

### define Variables
```
cat << EOF > group_vars/all.yml
---
COMMON_MSG: hello, common!
EOF
```

* Every role can refer to the variables defined in 'group_vars/all.yml'.

```
cat << EOF > roles/web/vars/main.yml
---
WEB_MSG: hello, world!
EOF
```

* 'web' role can refer to the variables defined in 'roles/web/vars/main.yml'.

### Revise Playbook
```
cat << EOF > roles/common/tasks/main.yml
---
- name: test common
  debug:
    msg: "{{ COMMON_MSG }}"
EOF
```

```
cat << EOF > roles/web/tasks/main.yml
---
- name: test web
  debug:
    msg: "{{ WEB_MSG }}"
EOF
```

### Directories
As a result, you can see the following output.

```
.
├── group_vars
│     ├── all.yml
├── host_vars
├── roles
│     ├── common
│     │     ├── tasks
│     │           ├── main.yml
│     ├── web
│           ├── tasks
│           │     ├── main.yml
│           ├── handlers
│           ├── templates
│           ├── files
│           ├── vars
│           │     ├── main.yml
│           ├── meta
├── webservers.yml
├── site.yml
├── inventory.yml
```

### Execute
```
ansible-playbook -i inventory.yml site.yml
```

### Result
```
PLAY [webservers] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [host1]

TASK [common : test common] ****************************************************
ok: [host1] => {
    "msg": "hello, common!"
}

TASK [web : test web] **********************************************************
ok: [host1] => {
    "msg": "hello, world!"
}

PLAY RECAP *********************************************************************
host1                      : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## Loops

### define Variables
```
cat << EOF > roles/web/vars/main.yml
---
WEB_MSGS:
  - one
  - two
EOF
```

* define a list.

### Revise Playbook
```
cat << EOF > roles/web/tasks/main.yml
---
- name: test web
  debug:
    msg: "{{ item }}"
  loop: "{{ WEB_MSGS }}"
EOF
```

* 'item' refers to the value in a list.

### Execute
```
ansible-playbook -i inventory.yml site.yml
```

### Result
```
PLAY [webservers] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [host1]

TASK [common : test common] ****************************************************
ok: [host1] => {
    "msg": "hello, common!"
}

TASK [web : test web] **********************************************************
ok: [host1] => (item=one) => {
    "msg": "one"
}
ok: [host1] => (item=two) => {
    "msg": "two"
}

PLAY RECAP *********************************************************************
host1                      : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

## Template

### Create Template Fle

```
cat << EOF > roles/web/templates/sample.conf.j2
Template
{{ WEB_MSGS }}
EOF
```

### Revise Playbook
```
cat << EOF > roles/web/tasks/main.yml
---
- name: test template
  template:
    src: sample.conf.j2
    dest: /tmp/sample.conf
EOF
```

### Execute
```
ansible-playbook -i inventory.yml site.yml
```

### Result
```
PLAY [webservers] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [host1]

TASK [common : test common] ****************************************************
ok: [host1] => {
    "msg": "hello, common!"
}

TASK [web : test template] *****************************************************
changed: [host1]

PLAY RECAP *********************************************************************
host1                      : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

### Confirm Template Fle
```
cat /tmp/sample.conf
```

```
Template
['one', 'two']
```

## Meta

### Define Dependencies.
```
cat << EOF > roles/web/meta/main.yml
---
dependencies:
  - common
EOF
```

* 'web' role depends on 'common' role.

### Revise 'webservers.yml'
```
cat << EOF > webservers.yml
---
- hosts: webservers
  roles:
    - web
EOF
```
* delete 'common'.

### Execute
```
ansible-playbook -i inventory.yml site.yml
```

### Result
```
PLAY [webservers] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [host1]

TASK [common : test common] ****************************************************
ok: [host1] => {
    "msg": "hello, common!"
}

TASK [web : test template] *****************************************************
ok: [host1]

PLAY RECAP *********************************************************************
host1                      : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

* 'common' role is also executed.

## Handler

### Create Handler
```
cat << EOF > roles/web/handlers/main.yml
---
- name: test handler
  debug:
    msg: "test handler executed"
  listen:
    - test_handler
EOF
```

### Revise Playbook
```
cat << EOF > roles/web/tasks/main.yml
---
- name: test handlers1
  command: /bin/true
  notify:
    - test_handler
- name: test handlers2
  command: /bin/true
  notify:
    - test_handler
EOF
```

* Notify 'test_handler' twice.

### Execute
```
ansible-playbook -i inventory.yml site.yml
```

### Result
```
PLAY [webservers] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [host1]

TASK [common : test common] ****************************************************
ok: [host1] => {
    "msg": "hello, common!"
}

TASK [web : test handlers1] ****************************************************
changed: [host1]

TASK [web : test handlers2] ****************************************************
changed: [host1]

RUNNING HANDLER [web : test handler] *******************************************
ok: [host1] => {
    "msg": "test handler executed"
}

PLAY RECAP *********************************************************************
host1                      : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

* 'test_handler' is executed only once.

## Script Module

### Create Shell Script
```
cat << "EOF" > roles/web/files/sample.sh
#!/bin/bash
cp -p "$1" "$1.bak"
EOF
```

### Revise Playbook
```
cat << EOF > roles/web/tasks/main.yml
---
- name: test script
  script: sample.sh sample.conf
  args:
    chdir: /tmp
EOF
```

### Execute
```
ansible-playbook -i inventory.yml site.yml
```

### Result
```
PLAY [webservers] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [host1]

TASK [common : test common] ****************************************************
ok: [host1] => {
    "msg": "hello, common!"
}

TASK [web : test script] *******************************************************
changed: [host1]

PLAY RECAP *********************************************************************
host1                      : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

### Confirm the result of 'sample.sh'
```
ls -lt /tmp
```

```
-rw-r--r-- 1 root    root     24 Aug 16 07:37 sample.conf
-rw-r--r-- 1 root    root     24 Aug 16 07:37 sample.conf.bak
```

## Shell Module

### Create Dummy Logs
```
touch /tmp/sample1_$(date +'%Y%m%d').log
touch /tmp/sample2_$(date +'%Y%m%d').log
```

### Revise Playbook
```
cat << "EOF" > roles/web/tasks/main.yml
---
- name: test ls
  shell: ls -1 /tmp/sample*.log
  register: result
- name: echo result
  debug:
    msg: "{{ result.stdout_lines }}"
EOF
```

### Execute
```
ansible-playbook -i inventory.yml site.yml
```

### Result
```
PLAY [webservers] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [host1]

TASK [common : test common] ****************************************************
ok: [host1] => {
    "msg": "hello, common!"
}

TASK [web : test ls] ***********************************************************
changed: [host1]

TASK [web : echo result] *******************************************************
ok: [host1] => {
    "msg": [
        "/tmp/sample1_20240816.log",
        "/tmp/sample2_20240816.log"
    ]
}

PLAY RECAP *********************************************************************
host1                      : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

## Fetch Module

### Revise Playbook
```
cat << "EOF" > roles/web/tasks/main.yml
---
- name: test ls
  shell: ls -1 /tmp/sample*.log
  register: result
- name: test fetch
  fetch:
    src: "{{ item }}"
    dest: /tmp/result/
    flat: yes
  loop: "{{ result.stdout_lines }}"
EOF
```

### Execute
```
ansible-playbook -i inventory.yml site.yml
```

### Result
```
PLAY [webservers] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [host1]

TASK [common : test common] ****************************************************
ok: [host1] => {
    "msg": "hello, common!"
}

TASK [web : test ls] ***********************************************************
changed: [host1]

TASK [web : test fetch] ********************************************************
changed: [host1] => (item=/tmp/sample1_20240816.log)
changed: [host1] => (item=/tmp/sample2_20240816.log)

PLAY RECAP *********************************************************************
host1                      : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

```
ls -lt /tmp/result
```

```
-rw-rw-r-- 1 vagrant vagrant 0 Aug 16 08:25 sample2_20240816.log
-rw-rw-r-- 1 vagrant vagrant 0 Aug 16 08:25 sample1_20240816.log
```

## Encrypt Root Password

### Create Password File
```
cat << EOF > extra_vars.yml
---
PASSWORD: vagrant
EOF
```

### Encrypt Password File
```
ansible-vault encrypt extra_vars.yml
```

```
New Vault password: 
Confirm New Vault password: 
Encryption successful
```

* Input 'test' twice.

### Revise Inventory
```
cat << EOF > inventory.yml
---
all:
  vars:
    ansible_user: root
  children:
    webservers:
      hosts:
        host1:
          ansible_host: 127.0.0.1
          ansible_password: "{{ PASSWORD }}"
EOF
```

### Execute
```
ansible-playbook -i inventory.yml site.yml --extra-vars @extra_vars.yml
```

### Result(Failed)
```
ERROR! Attempting to decrypt but no vault secrets found
```

### Execute
```
echo test > password.txt
```
```
ansible-playbook -i inventory.yml site.yml --extra-vars @extra_vars.yml --vault-password-file password.txt
```

### Result
```
TASK [Gathering Facts] *********************************************************
ok: [host1]

TASK [common : test common] ****************************************************
ok: [host1] => {
    "msg": "hello, common!"
}

TASK [web : test ls] ***********************************************************
changed: [host1]

TASK [web : test fetch] ********************************************************
ok: [host1] => (item=/tmp/sample1_20240816.log)
ok: [host1] => (item=/tmp/sample2_20240816.log)

PLAY RECAP *********************************************************************
host1                      : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

## Directories
As a result, you can see the following output.

```
.
├── group_vars
│     ├── all.yml
├── host_vars
├── roles
│     ├── common
│     │     ├── tasks
│     │           ├── main.yml
│     ├── web
│           ├── tasks
│           │     ├── main.yml
│           ├── handlers
│           │     ├── main.yml
│           ├── templates
│           │     ├── sample.conf.j2
│           ├── files
│           │     ├── sample.sh
│           ├── vars
│           │     ├── main.yml
│           ├── meta
│           │     ├── main.yml
├── webservers.yml
├── site.yml
├── inventory.yml
├── extra_vars.yml
```
