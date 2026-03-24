# Atelier 7

## Challenge

> Lancez les 4 VM et connectez-vous à la VM ```Control Host```

```console
[vagrant@ubuntu:atelier-7] vagrant up
[vagrant@ubuntu:atelier-7] vagrant ssh control
vagrant@control:~$ 
```

### Installer / Désinstaller des packages sur les VM depuis le Target Host
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

Nous pouvons finir en collectant l'information de l'espace utilisés sur les partitions de toutes nos VM avec le mode "command" utilisé au début du challenge :

```console
[vagrant@ansible ema]$ ansible all -m command -a "df -h /"
debian | CHANGED | rc=0 >>
Filesystem                       Size  Used Avail Use% Mounted on
/dev/mapper/debian--12--vg-root   62G  1.7G   57G   3% /
rocky | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        61G  2.1G   59G   4% /
suse | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        64G  2.4G   58G   4% /
[vagrant@ansible ema]$ ansible all -m command -a "df -h /"
debian | CHANGED | rc=0 >>
Filesystem                       Size  Used Avail Use% Mounted on
/dev/mapper/debian--12--vg-root   62G  1.7G   57G   3% /
suse | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        64G  2.4G   58G   4% /
rocky | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        61G  2.1G   59G   4% /
```

Pour conclure ce challenge, on remarque que, malgré que ce soit la même commande qui est executé successivement, ansible retourne qu'il y a un changement à chaque fois car il s'agit d'une commande bash. Ansible ne peut pas savoir s'il y a un changement contrairement à tout à l'heure où il appliquait des modifications de lui-même. Il pouvait donc facilement savoir s'il executait deux fois la même modification (en ajoutant/supprimer un fichier, installant/désinstallant un packages, etc.).

