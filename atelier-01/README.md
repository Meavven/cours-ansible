# Atelier 1

## Challenge n°1

Démarrez la VM ubuntu depuis le répertoire atelier-01.

```console
[vagrant@ubuntu:atelier-1] vagrant up ubuntu
```

Connectez-vous à cette VM.

```console
[vagrant@ubuntu:atelier-1] vagrant ssh ubuntu
```

Rafraîchissez les informations sur les paquets.

```console
vagrant@ubuntu~$ sudo apt update
```

Recherchez le paquet ansible avec les options qui vont bien.

```console
vagrant@ubuntu~$ apt-cache search --names-only ansible

    ansible - Configuration management, deployment, and task execution system
    ansible-core - Configuration management, deployment, and task execution system
    ansible-lint - lint tool for Ansible playbooks
    ansible-mitogen - Fast connection strategy for Ansible
    ...
```

Installez le paquet officiel fourni par la distribution.

```console
vagrant@ubuntu~$ sudo apt install ansible -y
```

Vérifiez si l'installation s'est bien déroulée et notez la version d'Ansible

```console
vagrant@ubuntu~$ ansible --version

    ansible 2.10.8
    ...
```

Déconnectez-vous et supprimez la VM.

```console
vagrant@ubuntu~$ exit
[user@sandbox:atelier-1] -f ubuntu
```


## Challenge n°2

Ajout de PPA Ansible (Personal Package Archive)

```console
vagrant@ubuntu~$ sudo apt-add-repository --yes --update ppa:ansible/ansible
    ...
    ansible - Ansible collections for ansible-core
    ansible-core - Ansible IT Automation
    ansible-test - Ansible IT Automation
```

```console
vagrant@ubuntu~$ sudo apt install ansible
```
