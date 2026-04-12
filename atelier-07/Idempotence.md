# Atelier 7 - Idempotence

## Objectif

- Installez successivement les paquets tree, git et nmap sur toutes les cibles.

- Désinstallez successivement ces trois paquets en utilisant le paramètre supplémentaire state=absent.

- Copiez le fichier /etc/fstab du Control Host vers tous les Target Hosts sous forme d'un fichier /tmp/test3.txt.

- Supprimez le fichier /tmp/test3.txt sur les Target Hosts en utilisant le module file avec le paramètre state=absent.

- Enfin, affichez l'espace utilisé par la partition principale sur tous les Target Hosts. Que remarquez-vous ?

---

## Challenge

> Commencez par lancer vos 4 VM et connectez-vous à votre VM ```Control Host```

### Installer / Désinstaller des packages sur les VM depuis le Control Host
> Installez successivement les 3 packages ```tree```, ```git``` et ```nmap``` avec une commande _ad hoc_ suivante éxecuté depuis la VM ```Control Host``` :
Inutile de se connecter aux 3 VM Targets une à une pour tout installer, Ansible propose son propre module pour installer des packages à distance sur d'autres VM :

```console
vagrant@ansible:~$ ansible all -m package -a "name=tree,git,nmap state=present"
```
> Le paramètre ```state=present``` permet ici de préciser que nous voulons "installer" les packages mentionnés. Les désinstaller aurait nécessité la valeur de ```state=absent```.
> Si on ne précise pas ce paramètre, il sera par défaut avec la valeur ```state=present```

```console
[vagrant@ansible ema]$ ansible all -m package -a "name=tree,git,nmap state=present"
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

> Nous pouvons tester d'executer deux fois successivement la commande de désinstallation afin de confirmer que la désinstallation s'est terminée avec succès.
> Le shell nous retourne qu'il n'y a eu aucun changement avec ```"changed: false``` étant donné que les packages ont bien été désinstallés lors de la première execution de commande. Il n'a donc plus rien a désinstaller.

```console
[vagrant@ansible]~$ ansible all -m package -a "name=tree,git,nmap state=absent"
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

### Copie de fichiers sur une liste de hosts

Créez à présent un fichier "test3.txt" et copiez-le sur toutes les VM en executant (depuis la VM de ```Control Host```) la commande ansible utilisant le mode "copy" :

```console
[vagrant@ansible]~$ ansible all -m copy -a "src=/etc/fstab dest=/tmp/test3.txt"
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
```

La fichier s'est bien copié sur nos trois VM. 
Nous pouvons le vérifier en réexcutant la commande de copy pour verifier qu'elle nous retourne bien ```"changed": false``` :

```console
[vagrant@ansible]~$ ansible all -m copy -a "src=/etc/fstab dest=/tmp/test3.txt"
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


> Nous pouvons à présent supprimer le fichier en utilisant le mode "file" :

```console
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
[vagrant@ansible]~$ ansible all -m command -a "df -h /"
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

