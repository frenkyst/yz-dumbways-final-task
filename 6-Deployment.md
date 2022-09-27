## Step 1
Clone both frontend and backend repository
```
https://github.com/yuuzukatsu/literature-frontend.git
https://github.com/yuuzukatsu/literature-backend.git
```
![image](https://user-images.githubusercontent.com/67664879/192433863-a9f31b93-6025-41bf-a012-cc63fd8c677d.png)

## Step 2
Move to production branch
```
git switch production
```
![image](https://user-images.githubusercontent.com/67664879/192434031-1d4ac181-4bfd-4b03-91c6-b5cb75c5aa43.png)

## Step 3
Copy `.gitignore` and name it `.dockerignore`. do this in both repository
```
cp .gitignore .dockerignore
```
![image](https://user-images.githubusercontent.com/67664879/192434607-8389bac0-ac9a-4d54-9cbd-aa08a5af61b4.png)

## Step 4
Login to docker hub with this command
```
docker login
```
![image](https://user-images.githubusercontent.com/67664879/192434854-e1b26c1f-1e2c-4f39-90d6-22c9964235e5.png)


# BACKEND DEPLOYMENT
On this deployment, i will run the app in Production environtment

## Step 1

Make `Dockerfile` in repo folder
```
FROM node:15
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 5000
CMD ["node", "server.js"]
```


## Step 2
Build image
```
docker image build -f Dockerfile -t yuuzukatsu/literature-backend:latest .
```
![image](https://user-images.githubusercontent.com/67664879/192471148-b301c7b9-9c12-4180-ab43-beb02bc1bd6c.png)

## Step 3
Make `backend-compose.yml`
```
version: '3.8'
services:
  
  backend-1:
    container_name: literature-backend-1
    image: yuuzukatsu/literature-backend:latest
    restart: unless-stopped
    ports:
      - '5000:5000'
    environment:
      DATABASE_URL: postgresql://<dbuser>:<dbpassword>@database:5432/<dbname>
      NODE_ENV: production
    networks:
      - literature-network
    command: >
      sh -c "npx sequelize-cli db:migrate --env production &&
             node server.js"

networks:
  literature-network:
    name: literature-network
    external: true
```

## Step 4
Run compose with command
```
docker compose -f backend-compose.yml up -d
```
![image](https://user-images.githubusercontent.com/67664879/192480653-7de17bd6-6875-4cc0-92f4-abd10767c498.png)

# FRONTEND DEPLOYMENT
Same as backend, i will use Production Environtment. Because 

## Step 1
Open `src/config/config.js` and edit `baseURL` line to your publicly accesable url
![image](https://user-images.githubusercontent.com/67664879/192482770-36558210-e65d-492e-ba81-6356ba04a7a3.png)


## Step 2
Open `.dockerignore` you copy before. And delete or comment line `/build`
![image](https://user-images.githubusercontent.com/67664879/192485328-9fd00f40-d443-4984-bb36-087c2c3c1dc1.png)

## Step 3
Because we need to build the app first to be used in production, we need to install npm first. Im using `nvm` method
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
exec bash
nvm install 15
```
![image](https://user-images.githubusercontent.com/67664879/192485692-c39ea937-3401-4a19-95fd-a16512c15673.png)
![image](https://user-images.githubusercontent.com/67664879/192485767-1f3c5bac-7ef0-438f-a8a1-42cdd3b24df5.png)

## Step 4
Because this is react app, we can build the app with command specified in `package.json` file
```
npm install
npm run build
```
![image](https://user-images.githubusercontent.com/67664879/192486443-75410eb4-9af7-4436-9b20-28e2539bdb0c.png)

## Step 5
Make `Dockerfile` in repo folder
```
FROM node:15
WORKDIR /app
COPY build .
RUN npm install -g serve
EXPOSE 3000
CMD [ "serve", "-s", "." ]
```
![image](https://user-images.githubusercontent.com/67664879/192497596-3f2765ea-b771-4434-9c34-4a4b36297e0d.png)

## Step 6
Create image with command
```
docker image build -f Dockerfile -t yuuzukatsu/literature-frontend:latest .
```
![image](https://user-images.githubusercontent.com/67664879/192498420-e2a81f4f-713c-4f5f-b278-506eb2a8a3b5.png)

## Step 7
Make `frontend-compose.yml` and insert this
```
![image](https://user-images.githubusercontent.com/67664879/192499708-6823eb85-7659-4c93-9af6-cec7b30729e0.png)

```

## Step 8
Run compose with command
```
docker compose -f frontend-compose.yml up -d
```
![image](https://user-images.githubusercontent.com/67664879/192500192-c9d514b5-4da0-4f1d-a591-11d125beba88.png)

