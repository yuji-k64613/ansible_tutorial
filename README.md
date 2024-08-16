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

