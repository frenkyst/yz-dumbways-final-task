# Create your own repository for dumblix frontend & backend

## Step 1

Create repository

![image](https://user-images.githubusercontent.com/67664879/192183022-7526e007-5ebf-4692-81a3-fa9619c53248.png)

## Step 2

Create key for Github Authentication

![image](https://user-images.githubusercontent.com/67664879/192183283-3aedcee4-6afc-4aba-8505-905c8da2ab8a.png)

## Step 3

Add Private Key to SSH Config

```
nano ~/.ssh/config
```
```
host github.com
    IdentityFile ~/.ssh/github
```

![image](https://user-images.githubusercontent.com/67664879/192184921-a81ac38a-4fa2-4d39-b906-ef6b79bad9b0.png)

## Step 4

Copy public key to Github

![image](https://user-images.githubusercontent.com/67664879/192185249-54097c5d-6123-42e1-9130-7384bb6e2988.png)

![image](https://user-images.githubusercontent.com/67664879/192188534-2e565858-3164-487d-baca-0553e73a03c0.png)


## Step 5

Test SSH Connection with this command

```
ssh -T git@github.com
```

![image](https://user-images.githubusercontent.com/67664879/192185495-be172ac6-7f5f-481a-aa9c-642189d0a892.png)

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
