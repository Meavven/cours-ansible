# Atelier 7

## Challenge

Lancez les 4 VM et connectez-vous à la VM control

```console
[vagrant@ubuntu:atelier-3] vagrant up
[vagrant@ubuntu:atelier-3] vagrant ssh control
vagrant@control:~$ 
```

Installer les 3 packages consécutivement avec une commande ```ad hoc``` depuis la VM ```Control Host``` :

```console
vagrant@control:~$ ansible all -m -package -a "name=tree"
vagrant@control:~$ ansible all -m -package -a "name=git"
vagrant@control:~$ ansible all -m -package -a "name=nmap"
```
