Before starting database, i will install docker first using this ansible playbook

```
- name: Install Docker with Ansible
  hosts: app,cicd
  vars:
    - username: <username for docker>
  become: yes

  tasks:
    - name: Prepare Apt
      ansible.builtin.apt:
        pkg: 
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
        update_cache: true    

    - name: Add Docker Official GPG Key
      ansible.builtin.apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker Repo
      ansible.builtin.apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu focal stable
        state: present

    - name: Update Apt and Install Docker + Dependencies
      ansible.builtin.apt:
        pkg: 
          - docker-ce 
          - docker-ce-cli 
          - containerd.io 
          - docker-compose-plugin
        update_cache: true
    
    - name: Add user to Docker Group
      ansible.builtin.user:
        name: "{{username}}"
        groups: docker
        append: yes

    - name: Install Pip
      ansible.builtin.apt:
        pkg: 
          - python3
          - python3-pip
        update_cache: true

    - name: Install Docker + Docker-Compose Module via Pip
      ansible.builtin.pip:
        name: 
          - docker
          - docker-compose

```

To run database, im using compose named `database-compose.yml`

```
version: '3.8'
services:
  
  database:
    container_name: database
    image: postgres:latest
    restart: unless-stopped
    ports:
      - '5432:5432'
    expose:
      - '5432'
    environment:
      POSTGRES_PASSWORD: <init password>
      POSTGRES_USER: <init username>
      POSTGRES_DB: <init database>
    volumes:
      - ~/postgre/:/var/lib/postgresql/data
```


And im using this playbook to run docker compose on server
```
- name: Install Docker PostgreSQL with Ansible
  hosts: app
  vars:
    docker:
      local: /home/yuuzukatsu/ansible-final/docker/
      remote: /home/dimasf/
      name: database-compose.yml

  tasks:
    - name: Send database-compose.yml to server
      ansible.builtin.copy:
        src: "{{docker.local}}/{{docker.name}}"
        dest: "{{docker.remote}}/{{docker.name}}"

    - name: Run Docker Compose
      community.docker.docker_compose:
        project_name: Database
        project_src: "{{docker.remote}}"
        files: 
          - "{{docker.name}}"
        restarted: yes
      
```
