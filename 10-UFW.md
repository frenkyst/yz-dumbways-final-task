```
- name: Setup UFW with Ansible
  hosts: all
  become: true
  vars:
    nginx_port: 
      - 80
      - 443
      - 9100
      - 9323
      - 52698
    app_port:
      - 3000:3010
      - 5000:5010
      - 5432
      - 9100
      - 9323
      - 52698
    cicd_port:
      - 3000
      - 8080
      - 9090
      - 9100
      - 9323
      - 52698

  tasks:

    # - name: Reset UFW 
    #   community.general.ufw:
    #     state: reset

    - name: Enable SSH Access
      community.general.ufw:
        rule: allow
        port: 22
    
    - name: Enable Defined Port on Nginx Server
      delegate_to: nginx
      community.general.ufw:
        rule: allow
        port: "{{item}}"
      loop: "{{nginx_port}}"
    
    - name: Enable Defined Port on App Server
      delegate_to: app
      community.general.ufw:
        rule: allow
        port: "{{item}}"
        proto: tcp
      loop: "{{app_port}}"
    
    - name: Enable Defined Port on CICD Server
      delegate_to: cicd
      community.general.ufw:
        rule: allow
        port: "{{item}}"
      loop: "{{cicd_port}}"

    - name: Reload and Enable UFW
      community.general.ufw:
        state: "{{item}}"
      loop:
        - reloaded
        - enabled
```
