# Atelier 1

## Objectif

#### Challenge n° 1

- Démarrez la VM ubuntu depuis le répertoire atelier-01.

- Connectez-vous à cette VM.

- Rafraîchissez les informations sur les paquets.

- Recherchez le paquet ansible avec les options qui vont bien.

- Installez le paquet officiel fourni par la distribution.

- Vérifiez si l'installation s'est bien déroulée.

- Notez la version d'Ansible.

- Déconnectez-vous et supprimez la VM.

#### Challenge n° 2

- Répétez le challenge précédent en configurant un dépôt PPA (Personal Package Archive) pour Ansible :

```console
$ sudo apt-add-repository ppa:ansible/ansible
```

- Notez la version fournie par ce dépôt tiers et comparez avec la version officielle du challenge précédent.

#### Challenge n° 3

- Lancez une VM Rocky Linux et installez Ansible en utilisant PIP et Virtualenv.

-------------------------------------------------------------------------------------


## Challenge n°1

Démarrez la VM ubuntu depuis le répertoire atelier-01.

```console
[user@sandbox:atelier-1] vagrant up ubuntu
```
![image](./atelier01-chall1-1.png)

Connectez-vous à cette VM.

```console
[user@sandbox:atelier-1] vagrant ssh ubuntu
```
![image](./atelier01-chall1-2.png)


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
![image](./atelier01-chall1-3.png)


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
![image](./atelier01-chall1-4.png)

Déconnectez-vous et supprimez la VM.

```console
vagrant@ubuntu~$ exit
[user@sandbox:atelier-1] -f ubuntu
```


## Challenge n°2

Reprenez le début de la procédure du challenge n°1 et poir recréez une VM et mettre à jour les repository.

Ajout de PPA Ansible (Personal Package Archive)

```console
vagrant@ubuntu~$ sudo apt-add-repository --yes --update ppa:ansible/ansible
```
![image](./atelier01-chall2-1.png)


```console
vagrant@ubuntu~$ apt-cache search --names-only ansible
    ...
    ansible - Ansible collections for ansible-core
    ansible-core - Ansible IT Automation
    ansible-test - Ansible IT Automation
```
![image](./atelier01-chall2-2.png)


```console
vagrant@ubuntu~$ sudo apt install ansible
```

```console
vagrant@ubuntu~$ ansible --version

    ansible [core 2.17.14]
    ...
```

On peut remarquer que la version de ansible ici est moins récente que celle du package du repository standard utilisé dans le challenge précédent.

Déconnectez-vous et supprimez de nouveau la VM.

```console
vagrant@ubuntu~$ exit
[user@sandbox:atelier-1] -f ubuntu
```


## Challenge n°3

Lancez avant tout, lancez une VM rocky et connectez-vous

```console
[user@sandbox:atelier-1] vagrant up rocky
[user@sandbox:atelier-1] vagrant ssh rocky
```
![image](./atelier01-chall3-1.png)


Puis installez le package python3-pip et python3 qui inclus le package nécessaires de venv

```console
vagrant@rocky~$ sudo dnf install -y python3 python3-pip
```

Initialisez l'environnement de venv et lancez-le

```console
vagrant@rocky~$ python3 -m venv ~/.venv/ansible
vagrant@rocky~$ source ~/.venv/ansible/bin/activate
```
![image](./atelier01-chall3-2.png)


A présent, mettez à jour pip

```console
(ansible) $ pip install --upgrade pip
```

Ensuite installez ansible avec pip et verifiez l'installation

```console
(ansible) $ pip install ansible
```

```console
(ansible) $ ansible --version
```
![image](./atelier01-chall3-3.png)


L'environnement virtuel peut être désactivé avec la commande "```$ deactivate```"
