## Step 1
make `jenkins-compose.yml` and insert this
```
version: '3.8'
services:
  
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins:lts-jdk11
    restart: always
    privileged: true
    user: root
    ports:
      - '8080:8080'
      - '50000:50000'
    expose:
      - '8080'
    volumes:
      - ~/jenkins:/var/jenkins_home


```
![image](https://user-images.githubusercontent.com/67664879/192600601-a035de13-cc4e-4136-87e4-8b5848147507.png)

## Step 2
Run last file with docker compose
```
docker compose -f jenkins-compose.yml up -d
```
![image](https://user-images.githubusercontent.com/67664879/192600898-fdfcf434-2f39-4796-beff-89fba385aac5.png)

## Step 3
Open docker logs and copy the authentication code
```
docker logs jenkins
```
![image](https://user-images.githubusercontent.com/67664879/192607216-e10243a9-550d-4bd2-8ff5-6cd578019e67.png)

## Step 4
Open jenkins url and insert authentication code
![image](https://user-images.githubusercontent.com/67664879/192617832-a9f0e897-b3fb-44d5-9b18-fe34f6b5911a.png)

## Step 5
Next step, we can choose plugins we want to install. In this case, i start with suggested plugin and ssh-agent
![image](https://user-images.githubusercontent.com/67664879/192618098-dc15adc4-1a62-4ea2-91b2-4f226f60038a.png)
After clicking install, jenkins will download and install all plugin you choose before
![image](https://user-images.githubusercontent.com/67664879/192618710-0289b1fd-9760-40f5-b078-a6f4419200da.png)

## Step 6
Create admin user as you want
![image](https://user-images.githubusercontent.com/67664879/192619089-b38093e5-689c-4fcc-ab19-5d37dc6b020e.png)

## Step 7
Insert url as you configure
![image](https://user-images.githubusercontent.com/67664879/192619199-aaf98630-6713-49ab-ac8d-4a21c34b28d0.png)

## Step 8
Now i will add ssh-key generated before. Follow this image to declare domain
![image](https://user-images.githubusercontent.com/67664879/192620832-f13a386f-c363-486a-bb6c-0dd91089fb50.png)
![image](https://user-images.githubusercontent.com/67664879/192620861-2d741239-833e-471e-af95-333f6d2dd6aa.png)
![image](https://user-images.githubusercontent.com/67664879/192621163-9d091a25-26e8-4e0e-8f0d-53c770134489.png)
![image](https://user-images.githubusercontent.com/67664879/192621193-0b6b1ed1-98a5-41fa-8b05-88c8aa899d0f.png)
![image](https://user-images.githubusercontent.com/67664879/192621658-0297a375-3489-47b5-a63c-1269eb08edc6.png)

## Step 9
Next we add the ssh-key
![image](https://user-images.githubusercontent.com/67664879/192621802-c5b56436-ade2-4a5a-b2a3-d5db1df4361d.png)
Use this parameter
```
- Kind: SSH Username with Private Key
- id: app-key
- username: dimasf
- private key: <enter your private key>
```
![image](https://user-images.githubusercontent.com/67664879/192622307-d4b756c6-4b22-45de-9471-e9ee0fa36925.png)


# Frontend Pipeline

## Step 1
Make `Jenkinsfile` in repo
```
def branch = "production"
def remoteurl = "https://github.com/yuuzukatsu/literature-frontend.git"
def remotename = "jenkins"
def workdir = "~/literature-frontend/"
def ip = "10.71.15.183"
def username = "dimasf"
def imagename = "yuuzukatsu/literature-frontend:latest"
def sshkeyid = "app-key"
def composefile = "frontend-compose.yml"

pipeline {
    agent any

    stages {
        stage('Pull From Frontend Repo') {
            steps {
                sshagent(credentials: ["${sshkeyid}"]) {
                    sh """
                        ssh -l ${username} ${ip} <<pwd
                        cd ${workdir}
                        git remote add ${remotename} ${remoteurl} || git remote set-url ${remotename} ${remoteurl}
                        git pull ${remotename} ${branch}
                        pwd
                    """
                }
            }
        }
            
        stage('Build Docker Image') {
            steps {
                sshagent(credentials: ["${sshkeyid}"]) {
                    sh """
                        ssh -l ${username} ${ip} <<pwd
                        cd ${workdir}
                        docker build -t ${imagename} .
                        pwd
                    """
                }
            }
        }
            
        stage('Deploy Image') {
            steps {
                sshagent(credentials: ["${sshkeyid}"]) {
                    sh """
                        ssh -l ${username} ${ip} <<pwd
                        cd ${workdir}
                        // docker compose -f ${composefile} down
                        docker compose -f ${composefile} up -d
                        pwd
                    """
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sshagent(credentials: ["${sshkeyid}"]) {
                    sh """
                        ssh -l ${username} ${ip} <<pwd
                        docker image push ${imagename}
                        docker image prune -f --all
                        pwd
                    """
                }
            }
        }


        stage('Send Success Notification') {
            steps {
                sh """
                    curl -X POST 'https://api.telegram.org/bot${env.telegramapi}/sendMessage' -d \
		    'chat_id=${env.telegramid}&text=Build ID #${env.BUILD_ID} Frontend Pipeline Successful!'
                """
            }
        }

    }
}

```
![image](https://user-images.githubusercontent.com/67664879/192656116-0a22d594-e57b-4820-9033-e756d59cf507.png)

## Step 2
Create new pipeline in Jenkins
![image](https://user-images.githubusercontent.com/67664879/192656182-0bc8257f-bbb1-456b-b45e-22a884473cf4.png)

## Step 3
Fill in like this
```
- Do not allow concurrent builds
- This project is parameterized
  - string telegramapi: <your telegram API>
  - string telegramid: <your chat id>
- GitHub hook trigger for GITScm polling
- Pipeline from SCM
  - SCM: Git
  - Repository URL: https://github.com/yuuzukatsu/literature-frontend.git
  - Branch Specifier: */production
  - Script path: Jenkinsfile
```

## Step 4
Add webhook
![image](https://user-images.githubusercontent.com/67664879/192656271-11156e4a-94c1-4d6a-8add-88ab4461c28f.png)


use this parameter
```
- Payload URL: https://yourjenkins.url/github-webhook/
- Content type: Application/Json
- Just the push event.
```
![image](https://user-images.githubusercontent.com/67664879/192656357-072e15bc-1247-41a5-80bc-f437b8901163.png)


# Backend Pipeline

## Step 1
Make `Jenkinsfile` in repo
```
def branch = "production"
def remoteurl = "https://github.com/yuuzukatsu/literature-backend.git"
def remotename = "jenkins"
def workdir = "~/literature-backend/"
def ip = "10.71.15.183"
def username = "dimasf"
def imagename = "yuuzukatsu/literature-backend:latest"
def sshkeyid = "app-key"
def composefile = "backend-compose.yml"

pipeline {
    agent any

    stages {
        stage('Pull From Backend Repo') {
            steps {
                sshagent(credentials: ["${sshkeyid}"]) {
                    sh """
                        ssh -l ${username} ${ip} <<pwd
                        cd ${workdir}
                        git remote add ${remotename} ${remoteurl} || git remote set-url ${remotename} ${remoteurl}
                        git pull ${remotename} ${branch}
                        pwd
                    """
                }
            }
        }
            
        stage('Build Docker Image') {
            steps {
                sshagent(credentials: ["${sshkeyid}"]) {
                    sh """
                        ssh -l ${username} ${ip} <<pwd
                        cd ${workdir}
                        docker build -t ${imagename} .
                        pwd
                    """
                }
            }
        }
            
        stage('Deploy Image') {
            steps {
                sshagent(credentials: ["${sshkeyid}"]) {
                    sh """
                        ssh -l ${username} ${ip} <<pwd
                        cd ${workdir}
                        // docker compose -f ${composefile} down
                        docker compose -f ${composefile} up -d
                        pwd
                    """
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sshagent(credentials: ["${sshkeyid}"]) {
                    sh """
                        ssh -l ${username} ${ip} <<pwd
                        docker image push ${imagename}
                        docker image prune -f --all
                        pwd
                    """
                }
            }
        }


        stage('Send Success Notification') {
            steps {
                sh """
                    curl -X POST 'https://api.telegram.org/bot${env.telegramapi}/sendMessage' -d \
		    'chat_id=${env.telegramid}&text=Build ID #${env.BUILD_ID} Backend Pipeline Successful!'
                """
            }
        }

    }
}

```
![image](https://user-images.githubusercontent.com/67664879/192647097-67e35887-01f9-4317-8412-11f1612c7d5a.png)

## Step 2
Create new pipeline in Jenkins
![image](https://user-images.githubusercontent.com/67664879/192647137-6227035f-2561-47ff-b4ee-66a5e42d2833.png)

## Step 3
Fill in like this
```
- Do not allow concurrent builds
- This project is parameterized
  - string telegramapi: <your telegram API>
  - string telegramid: <your chat id>
- GitHub hook trigger for GITScm polling
- Pipeline from SCM
  - SCM: Git
  - Repository URL: https://github.com/yuuzukatsu/literature-backend.git
  - Branch Specifier: */production
  - Script path: Jenkinsfile
```

## Step 4
Add webhook
![image](https://user-images.githubusercontent.com/67664879/192648011-cc9d2687-48a1-4852-9a0c-358b411600a9.png)

use this parameter
```
- Payload URL: https://yourjenkins.url/github-webhook/
- Content type: Application/Json
- Just the push event.
```
![image](https://user-images.githubusercontent.com/67664879/192648759-1b4d4c02-0917-4751-8db4-d1206a32fe53.png)
