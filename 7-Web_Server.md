# Setup NGINX
on Installation and Configuration, im using Ansible because there's a lot of config file to make and its more convinient to edit them locally

Make 5 Config file

- `frontend.conf`
```
upstream frontend { 
    server 10.71.15.183:3000;
    server 10.71.15.183:3001;
}
server { 
    server_name dimasf.studentdumbways.my.id; 
  
    location / { 
             proxy_pass http://frontend;
    }
}
```

- `backend.conf`
```
upstream backend { 
    server 10.71.15.183:5000;
    server 10.71.15.183:5001;
}
server { 
    server_name api.dimasf.studentdumbways.my.id; 
  
    location / { 
             proxy_pass http://backend;
    }
}
```

- `monitoring.conf`
```
server {
  server_name monitoring.dimasf.studentdumbways.my.id;
  location / {
    proxy_pass http://10.71.15.234:3000;
    proxy_set_header Host      $host;
    proxy_set_header X-Real-IP $remote_addr;
  }
}
```

- `prometheus.conf`
```
server {
  server_name prometheus.dimasf.studentdumbways.my.id;
  location / {
    proxy_pass http://10.71.15.234:9090;
  }
}
```
- `jenkins.conf`
```
server {
  server_name jenkins.dimasf.studentdumbways.my.id;
  location / {
    proxy_pass http://10.71.15.234:8080;
  }
}
```

ansible playbook `3-Setup-Nginx.yml`
```
- name: Install Nginx with Ansible
  hosts: nginx
  vars:
    - localdir: /home/yuuzukatsu/ansible-final/nginx
    - remotedir: /etc/nginx
  become: yes

  tasks:
    - name: Install Nginx
      ansible.builtin.apt:
        pkg: 
          - nginx
        update_cache: true

    - name: Copy All Nginx Config
      ansible.builtin.copy:
        src: "{{localdir}}/"
        dest: "{{remotedir}}/sites-available/"

    - name: Add filename to Variable rplist
      ansible.builtin.find:
        path: "{{remotedir}}/sites-available"
        file_type: file
      register: rplist

    - name: Enable Sites With Sysmlink
      ansible.builtin.file:
        src: "{{item.path}}"
        dest: "{{remotedir}}/sites-enabled/{{item.path | basename}}"
        state: link
      with_items:
        - "{{rplist.files}}"

    - name: Reload Nginx
      ansible.builtin.systemd:
        state: reloaded
        name: nginx.service
        enabled: yes

```

# Setup SSL


## Step 1
Create API key on cloudflare <https://dash.cloudflare.com/profile/api-tokens>
![image](https://user-images.githubusercontent.com/67664879/192566344-f2053da6-03b6-4e2c-8168-f14ffe546ee3.png)
![image](https://user-images.githubusercontent.com/67664879/192566397-a6432523-3e93-41ce-82aa-763379f0ecd3.png)
![image](https://user-images.githubusercontent.com/67664879/192566547-7c81f95f-9422-4ba6-83ac-beea85ef655e.png)

## Step 2
Create new file in `~/.secret/cloudflare.ini`, and fill it with below
```
dns_cloudflare_api_token = <YOUR API TOKEN>
```
![image](https://user-images.githubusercontent.com/67664879/192567659-f6aab7a3-a249-49ab-af7f-579fdd435284.png)

## Step 3
For extra security, make file and directory read/write only for owner
```
chmod 600 .secret/cloudflare.ini
```

## Step 4
Follow <https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal> for certbot installation
Use `cloudflare` plugin
```
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo snap set certbot trust-plugin-with-root=ok
sudo snap install certbot-dns-cloudflare
```

## Step 5
Use this command to generate and auto install certificate
```
sudo certbot \
-i nginx \
--dns-cloudflare --dns-cloudflare-credentials ~/.secret/cloudflare.ini \
-d dimasf.studentdumbways.my.id \
-d *.dimasf.studentdumbways.my.id \
-m your@email.com \
--agree-tos -n
```
if certificate installation shows error, try this command (shown on error log)
```
sudo certbot install --cert-name dimasf.studentdumbways.my.id
```

![image](https://user-images.githubusercontent.com/67664879/192576103-b508d003-675d-448b-929a-c1de4b60c0fe.png)

Test page
![image](https://user-images.githubusercontent.com/67664879/192581868-27ffe55c-024b-4605-b3bc-11a262b40066.png)

