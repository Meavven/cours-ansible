# Atelier 7

## Challenge

> Lancez les 4 VM et connectez-vous à la VM ```Control Host```

```console
[vagrant@ubuntu:atelier-7] vagrant up
[vagrant@ubuntu:atelier-7] vagrant ssh control
vagrant@control:~$ 
```

> Installez successivement les 3 packages avec une commande _ad hoc_ depuis la VM ```Control Host``` :

```console
vagrant@control:~$ ansible all -m package -a "name=tree,git,nmap state=present)"
```
> Le paramètre ```state=present``` permet ici de préciser que nous voulons "installer" les packages mentionnés. Les désinstaller aurait nécessité la valeur de ```state=absent```.
> Si on ne précise pas ce paramètre, il sera par défaut avec la valeur ```state=present```
> 
```console
[vagrant@ansible ema]$ ansible all -m package -a "name=tree,git,nmap"
suse | SUCCESS => {
    "changed": false,
    "name": [
        "tree",
        "git",
        "nmap"
    ],
    "rc": 0,
    "state": "present",
    "update_cache": false
}
debian | SUCCESS => {
    "cache_update_time": 1761245363,
    "cache_updated": false,
    "changed": false
}
rocky | SUCCESS => {
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
```

Les trois packages se sont bien installés. On peut désormais les désinstaller.

> Nous pouvons tester d'executer deux fois successivement la commande de désinstallation afin de confirmer que la désinstallati on s'est terminée avec succès.
> Le terminal nous retourne qu'il n'y a eu aucun changement étant donné que les packages ont bien été désinstallés lors de la première execution de commande.

```console
[vagrant@ansible ema]$ ansible all -m package -a "name=tree,git,nmap state=absent"
debian | SUCCESS => {
    "changed": false
}
suse | SUCCESS => {
    "changed": false,
    "name": [
        "tree",
        "git",
        "nmap"
    ],
    "rc": 0,
    "state": "absent",
    "update_cache": false
}
rocky | SUCCESS => {
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
```

Les trois packages se sont bel et bien déinstallation. On le confirme par l'information de "changed: false". Aucun changement n'a été executé.

### Copie de fichier sur une liste de hosts

Créez à présent un fichier "test3.txt" et copiez-le sur toutes les VM en executant (depuis la VM ```Control Host```) la commande ansible utilisant le mode "copy" :

```console
[vagrant@ansible ema]$ ansible all -m copy -a "src=/etc/fstab dest=/tmp/test3.txt"
suse | SUCCESS => {
    "changed": false,
    "checksum": "0fe1d6fcaf1695fb3ef9d8a42d45d04e5e0c11c2",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "path": "/tmp/test3.txt",
    "size": 679,
    "state": "file",
    "uid": 0
}
debian | SUCCESS => {
    "changed": false,
    "checksum": "0fe1d6fcaf1695fb3ef9d8a42d45d04e5e0c11c2",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "path": "/tmp/test3.txt",
    "size": 679,
    "state": "file",
    "uid": 0
}
rocky | SUCCESS => {
    "changed": false,
    "checksum": "0fe1d6fcaf1695fb3ef9d8a42d45d04e5e0c11c2",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "path": "/tmp/test3.txt",
    "secontext": "unconfined_u:object_r:user_home_t:s0",
    "size": 679,
    "state": "file",
    "uid": 0
}
```

La fichier s'est bien copié sur nos trois VM. 
> Nous pouvons le supprimer en utilisant le mode "file" :

```console
[vagrant@ansible ema]$ ansible all -m copy -a "src=/etc/fstab dest=/tmp/test3.txt"
debian | CHANGED => {
    "changed": true,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
rocky | CHANGED => {
    "changed": true,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
suse | CHANGED => {
    "changed": true,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
[vagrant@ansible ema]$ ansible all -m file -a "path=/tmp/test3.txt state=absent"
debian | SUCCESS => {
    "changed": false,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
suse | SUCCESS => {
    "changed": false,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
rocky | SUCCESS => {
    "changed": false,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
```
