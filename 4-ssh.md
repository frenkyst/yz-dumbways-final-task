Create file `~/.ssh/config` and insert this

```
host app
    HostName <app server ip>
    User <username>
    IdentityFile /home/yuuzukatsu/.ssh/yzkey
    ForwardAgent yes
    AddKeysToAgent yes

host nginx
    HostName <nginx server ip>
    User <username>
    IdentityFile /home/yuuzukatsu/.ssh/yzkey
    ForwardAgent yes
    AddKeysToAgent yes

host cicd
    HostName <cicd server ip>
    User <username>
    IdentityFile /home/yuuzukatsu/.ssh/yzkey
    ForwardAgent yes
    AddKeysToAgent yes
```
