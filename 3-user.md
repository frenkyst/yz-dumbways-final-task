For this task, im going to use Ansible because i have to make user on all server

## Step 1

Generate SSH Key for Ansible

```
ssh-keygen -C dimasf@dumbways -t ed25519 -f ansible-key 
```

![image](https://user-images.githubusercontent.com/67664879/192203742-b68a2727-b9f1-45d2-aeae-c6a5cae2eab6.png)

## Step 2

Make `inventory.yml` file

```
---
all:
  hosts:
    nginx:
      ansible_host: <ip server nginx>
    app:
      ansible_host: <ip server app>
    cicd:
      ansible_host: <ip server cicd>

```

## Step 3

Make `group_vars/all.yml`

```
ansible_connection: ssh
ansible_user: dimasf
ansible_become_pass: <sudo password>
ansible_ssh_private_key_file: <private key location>
ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
```

## Step 4

Generate password for new user using command
```
mkpasswd --method=SHA-256
```
![image](https://user-images.githubusercontent.com/67664879/192205311-c8a561e7-1479-4a22-8267-e5ad39abe826.png)

## Step 5

Make `0-create-user.yml` and insert this
```
- name: Create user for Ansible
  hosts: all
  vars:
    - username: dimasf
    - password: <password generated before>
    - publickey: <public key location>
  become: yes
  tasks:
    - name: Create user "{{username}}"
      ansible.builtin.user:
        name: "{{username}}"
        password: "{{password}}"
        groups: sudo
        append: yes
        shell: /bin/bash
    
    - name: Add public key to server
      ansible.posix.authorized_key:
        user: "{{username}}"
        key: "{{ lookup('file', '{{publickey}}') }}"

    - name: Disable SSH Password Auth
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: PasswordAuthentication yes
        line: PasswordAuthentication no

    - name: Enable Public Key Auth
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: \#PubkeyAuthentication yes
        line: PubkeyAuthentication yes
        
    - name: Enable Public Key Auth
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: \#PubkeyAuthentication yes
        line: PubkeyAuthentication yes

    - name: Reload SSH Config
      ansible.builtin.systemd:
        state: reloaded
        name: sshd.service
```

## Step 6

Run playbook with command
```
ansible-playbook -i inventory.yml 0-create-user.yml -k -u <initial ssh username>
```
