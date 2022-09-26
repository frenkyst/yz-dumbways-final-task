# Create your own repository for dumblix frontend & backend

## Step 1

Create repository

![image](https://user-images.githubusercontent.com/67664879/192183022-7526e007-5ebf-4692-81a3-fa9619c53248.png)

## Step 2

Create key for Github Authentication

![image](https://user-images.githubusercontent.com/67664879/192318434-e5d9762c-a32a-4b72-b0a5-38838f63d16e.png)


## Step 3

Add Private Key to SSH Config

```
nano ~/.ssh/config
```
```
host github.com
    IdentityFile ~/.ssh/github
```

![image](https://user-images.githubusercontent.com/67664879/192318642-df981ea5-058e-4d4b-a800-393d17e6e161.png)

## Step 4

Copy public key to Github

![image](https://user-images.githubusercontent.com/67664879/192319226-1dd9fa6c-ccfa-4ec3-b170-637279793a5e.png)


![image](https://user-images.githubusercontent.com/67664879/192319157-5a46046c-a673-43ae-97ee-d11cdb6d2617.png)



## Step 5

Test SSH Connection with this command

```
ssh -T git@github.com
```

![image](https://user-images.githubusercontent.com/67664879/192319702-e8090192-f161-40c0-8f12-8795327c6c5c.png)


## Step 6

Add Username and Email for Pull and Push

![image](https://user-images.githubusercontent.com/67664879/192188010-292945ce-77b6-47cc-bb82-e0852133b3a4.png)

# Clone Repository and Push to New Repository


## Step 1

Clone both frontend and backend repo

```
git clone https://github.com/dumbwaysdev/literature-frontend.git
git clone https://github.com/dumbwaysdev/literature-backend.git
```

![image](https://user-images.githubusercontent.com/67664879/192188227-6bf433a0-b24d-41fc-8e72-8bd178ce9c72.png)

## Step 2

Change remote url for each repository to your own

```
git remote set-url <remote-name> <remote-url>
```

![image](https://user-images.githubusercontent.com/67664879/192189969-0919fa94-3e40-4bfb-aade-9cca4e3ea5cf.png)

![image](https://user-images.githubusercontent.com/67664879/192190001-0707f0ca-328e-4557-a7f7-0bcea6e302e2.png)

## Step 3

Create 3 branch (Development, Staging, & Production) on each repository

```
git branch -c <origin branch> <destination branch>
```

![image](https://user-images.githubusercontent.com/67664879/192190583-4549a04c-89ba-49c4-84af-962b9846cac1.png)

## Step 4

Push all branch to repository

```
git push <remote name> <branch name>
```

![image](https://user-images.githubusercontent.com/67664879/192193000-1b75ab4f-5c98-43ec-adf7-008a87a1c928.png)
