Create file `~/.ssh/config` and insert this

```
host nginx
    HostName 103.172.204.181
    User yuuzukatsu
    IdentityFile /home/yuuzukatsu/.ssh/yzkey
    ForwardAgent yes
    AddKeysToAgent yes
    RemoteForward 52698 127.0.0.1:52698

host app
    # HostName 103.179.57.51
    Hostname 10.71.15.183
    User yuuzukatsu
    IdentityFile /home/yuuzukatsu/.ssh/yzkey
    ForwardAgent yes
    AddKeysToAgent yes
    RemoteForward 52698 127.0.0.1:52698
    ProxyCommand ssh nginx -W %h:%p

host cicd
    # HostName 103.174.114.52
    HostName 10.71.15.234
    User yuuzukatsu
    IdentityFile /home/yuuzukatsu/.ssh/yzkey
    ForwardAgent yes
    AddKeysToAgent yes
    RemoteForward 52698 127.0.0.1:52698
    ProxyCommand ssh nginx -W %h:%p
```
