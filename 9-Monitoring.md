# NODE EXPORTER DEPLOYMENT
In this step, i will use ansible again

`exporter-compose.yml`
```
version: '3.8'
services:
  
  node-exporter:
    container_name: node-exporter
    image: prom/node-exporter:latest
    restart: always
    ports:
      - '9100:9100'
    expose:
      - '9100'
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
```

playbook `5-setup-exporter.yml`
```
- name: Install Node Exporter on Docker with Ansible
  hosts: all
  vars:
    localdir: /home/yuuzukatsu/ansible-final
    remotedir: /home/dimasf

  tasks:

    - name: Enable Docker Monitoring
      become: true
      ansible.builtin.lineinfile:
        dest: /etc/docker/daemon.json
        line: |
          {
            "metrics-addr" : "127.0.0.1:9323",
            "experimental" : true
          }
        create: true
        insertbefore: EOF

    - name: Restart Docker Daemon
      become: true
      ansible.builtin.systemd:
        name: docker
        state: restarted

    - name: Send exporter-compose.yml to server
      ansible.builtin.copy:
        src: '{{localdir}}/docker/exporter-compose.yml'
        dest: '{{remotedir}}/exporter-compose.yml'
          
    - name: Run Docker Compose
      community.docker.docker_compose:
        project_name: Exporter
        project_src: '{{remotedir}}'
        files: 
          - exporter-compose.yml
        restarted: yes
```

Run playbook with command
```
ansible-playbook -i inventory.yml 5-setup-exporter.yml
```

# PROMETHEUS + GRAFANA DEPLOYMENT

In this deployment, im using ansible again

## Step 1
create `prometheus.yml`
```
global:
  scrape_interval: 15s 

scrape_configs:

  - job_name: 'app-server-docker'
    static_configs:
      - targets: ['10.71.15.183:9323']

  - job_name: 'app-server-exporter'
    static_configs:
      - targets: ['10.71.15.183:9100']

  - job_name: 'nginx-server-docker'
    static_configs:
      - targets: ['10.71.15.19:9323']

  - job_name: 'nginx-server-exporter'
    static_configs:
      - targets: ['10.71.15.19:9100']

  - job_name: 'monitoring-server-docker'
    static_configs:
      - targets: ['10.71.15.234:9323']

  - job_name: 'monitoring-server-exporter'
    static_configs:
      - targets: ['10.71.15.234:9100']
```

## Step 2
I will make basic auth for prometheus. Create this script in Python
`bcrypt-pass.py`
```
import getpass
import bcrypt

password = getpass.getpass("password: ")
hashed_password = bcrypt.hashpw(password.encode("utf-8"), bcrypt.gensalt())
print(hashed_password.decode())
```
![image](https://user-images.githubusercontent.com/67664879/192610633-0b973cc5-9c64-4f58-a628-997ce58edd61.png)

## Step 3
Run the script with this command
```
python3 bcrypt-pass.py 
```
Insert password and save the output string
![image](https://user-images.githubusercontent.com/67664879/192610928-4a13da5d-5a5f-41cb-8370-57800ad38baf.png)

## Step 4
Create `web.yml`
```
basic_auth_users:
    admin: <your hashed password>
```

## Step 5
Create `monitoring-compose.yml`
```
version: '3.8'
services:
  
  prometheus:
    container_name: prometheus
    image: prom/prometheus:latest
    restart: always
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./web.yml:/etc/prometheus/web.yml
      - prometheus-data:/prometheus
    ports:
      - '9090:9090'
    expose:
      - '9090'
    command:
      - '--web.config.file=/etc/prometheus/web.yml'

  grafana:
    container_name: grafana
    image: grafana/grafana:latest
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - '3000:3000'
    expose:
      - '3000'
    
volumes:
  prometheus-data: {}
  grafana-data: {}
```

## Step 4
Create playbook and name it `6-setup-monitoring.yml`
```
- name: Install Grafana Monitoring + Prometheus on Docker with Ansible
  hosts: cicd
  vars:
    localdir: /home/yuuzukatsu/ansible-final
    remotedir: /home/dimasf

  tasks:
    - name: Send prometheus.yml, web.yml and monitoring-compose.yml to server
      ansible.builtin.copy:
        src: '{{localdir}}/docker/{{item}}'
        dest: '{{remotedir}}/{{item}}'
      loop: 
        - prometheus.yml
        - web.yml
        - monitoring-compose.yml

    - name: Run Docker Compose
      community.docker.docker_compose:
        project_name: Monitoring
        project_src: '{{remotedir}}'
        files: 
          - monitoring-compose.yml
        restarted: yes
```

## Step 5
Run playbook with command
```
ansible-playbook -i inventory.yml 6-setup-monitoring.yml
```

## Step 6
Open prometheus url and there will be user password prompt
![image](https://user-images.githubusercontent.com/67664879/192615357-0b9f65af-1231-4507-be09-7e96bb9e9e94.png)
