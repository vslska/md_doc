Техничный (SSH Config)

 Чтобы не писать дебильные ипишники и грязь не делать в /etc/hosts прописывая туда ипи и имена, батя давай иди в ssh config

 `~/.ssh/config` на ноуте:
 
```ini
 Host node1
    HostName 192.168.122.10
    User fedora
    IdentityFile ~/.ssh/id_rsa

Host node2
    HostName 192.168.122.11
    User fedora

```

