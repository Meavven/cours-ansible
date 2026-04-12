# Atelier 6 - Configuration de Base

## Objectif

- Éditez /etc/hosts de manière à ce que les Target Hosts soient joignables par leur nom d'hôte simple.

- Configurez l'authentification par clé SSH avec les trois Target Hosts.

- Installez Ansible.

- Envoyez un premier ping Ansible sans configuration.

- Créez un répertoire de projet ~/monprojet.

- Créez un fichier vide ansible.cfg dans ce répertoire.

- Vérifiez si ce fichier est bien pris en compte par Ansible.

- Spécifiez un inventaire nommé hosts.

- Activez la journalisation dans ~/journal/ansible.log.

- Testez la journalisation.

- Créez un groupe [testlab] avec vos trois Target Hosts.

- Définissez explicitement l'utilisateur vagrant pour la connexion à vos cibles.

- Envoyez un ping Ansible vers le groupe de machines [all].

- Définissez l'élévation des droits pour l'utilisateur vagrant sur les Target Hosts.

- Affichez la première ligne du fichier /etc/shadow sur tous les Target Hosts.

- Quittez le Control Host et supprimez toutes les VM de l'atelier.  
  
---

## Challenge

### Pré-configuration

Commencez par pré-configurer votre environnement en vous basant sur les quatres points suivant de l'atelier-03 :

  > - lancement des quatres VM avec vagrant  
  > -  Mise à jour du fichier hosts de la VM ```Control Host``` avec les hostname des trois VM targets  
  > - Installation de ansible sur la VM ```Control Host```  
  > - Ajout des clefs SSH des VM pour pouvoir ping les trois targets à partir de la VM ```Control Host``` avec le mode ping de ansible  

Connectez-vous à la VM ```control``` qui gère Ansible et verifiez la pré-configuration avec la commande suivante sollicitant le mode _ping_ de ansible :

```console
[vagrant@ubuntu:atelier-7] vagrant up
[vagrant@ubuntu:atelier-7] vagrant ssh control
vagrant@ansible:~$ 
```
```console
vagrant@control:~$ ansible all -i target01,target02,target3 -m ping
```


![image](./atelier06-1.png)


### Création de ansible.cfg

> Créez un dossier ```~/monprojet``` dans le répertoire du home avec dedans un fichier ```ansible.cfg```  

> Verifier que c'est bien votre nouveau fichier de configuration ```ansible.cfg``` de votre repertoire qui est pris en compte en vous servant de la commande suivante qui affichera le chemin du fichier ansible.cfg sur lequel ansible se réfère :  

```console
vagrant@control:~$ ansible --version
```

> La commande nous retourne le bon chemin vers le nouveau fichier de configuration ansible.cfg créé à l'instant :   
```config file = /home/vagrant/monprojet/ansible.cgf```

![image](./atelier06-2.png)


### Pré-configuration de l'inventory

> Spécifiez dans votre fichier "_ansible.cfg_" un chemin vers un futur fichier inventory  

```txt
[defaults]
inventory = ./hosts
```
### Activation des logs

> Ajoutez à présent dans "_ansible.cfg_" un chemin vers un futur fichier de log  

```txt
[defaults]
...
log_path = ~/journal/ansible.log
```

> Créez le chemin ```~/journal/``` qui contiendra le fichier de log. Le fichier se créera de lui-même lorsqu'un log sera généré  

```console
vagrant@control:~$ mkdir ~/journal
```

Afin de tester la création automatique du fichier de log, générez un simple log en éxecutant un ping d'une des VM avec le module "ping" de Ansible vue précédemment  


### Configuration de l'inventory

> Créez et configurez un nouveau fichier inventory nommé "_hosts_" avec la configuration plus bas qui remplira les tâches suivantes :
    - Ajout d'un groupe "_testlab_" contenant nos trois VM target
    - Spécification du nom d'utilisateur _vagrant_ des VM target pour une connexion
    - Spécification d'un interpreteur (python3)

__Configuration__ du fichier ```hosts``` :

```txt
[testlab]
target01
target02
target03

[testlab:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=vagrant
```

> Vérifions à présent le bon fonctionnement de la configuration avec un ping de nos VM via l'inventaire qui contient les informations nécessaires à leurs connexions (fichier inventaire ici nommé "hosts") :  

```console
vagrant@control:~$ ansible hosts -m ping
```

Vous devriez obtenir un résultat similaire à ci-dessous 

```txt
target02 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
target01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
target03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

> Une verification optionnelle est possible pour confirmer que les trois hosts sont bien dans le registre de l'inventaire avec la commande suivante :  

```console
vagrant@control:~$ ansible all --list-hosts
```

> Si tout est opérationnel, la commande doit vous retourner la liste de vos trois VM Targets.
```txt
  hosts (3):
    target01
    target02
    target03
```

### Elévation des privilèges de "vagrant"

L'objectif à présent est d'élever les privilèges de l'utilisateur ```vagrant``` des VM Targets depuis la VM "control" afin qu'elle puisse élargir ses droits d'execution de commandes sur les VM targets

> Ajoutez le paramètre "_ansible_become=yes_" dans les variables du fichier inventory "_hosts_" :

```txt
[testlab:vars]
...
ansible_become=yes
```

Vous pouvez verifier l'élevation des privilèges en executant une commande pour afficher le fichier ```/etc/shadow``` des VM target

```console
vagrant@control:~$ ansible all -a "head -n 1 /etc/shadow"
```

![image](./atelier06-3.png)

Notre élévation de privilèges est désormais fonctionnelle.


