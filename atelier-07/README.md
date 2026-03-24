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

> Nous pouvons tester d'executer deux fois successivement la commande de désinstallation. Le terminal nous retourne qu'il n'y a eu aucun changement étant donné que les packages ont déjà été désinstallés lors de la première execution de commande.

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
