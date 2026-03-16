# Atelier 6

## Challenge

Mise à niveau de l'atelier-06 avec la pré-configuration de l'atelier-03 :

- lancement des quatres VM avec vagrant
- Mise à jour du fichier hosts de la VM "control" avec les hostname des trois VM targets
- Installation de ansible sur la VM "control"
- Ajout des clefs SSH des VM pour pouvoir ping les trois targets à partir de la VM "control" avec le mode ping de ansible

Verification de la pré-configuration avec la commande suivante sollicitant le mode _ping_ de ansible :
```console
vagrant@control:~$ ansible all -i target01,target02,target3 -m ping
```


![image](./atelier06-1.png)


Créer un dossier monprojet dans le répertoire du home avec dedans un fichier ansible.cfg
Verifier que le fichier de configuration est bien pris en compte avec la commande suivante :

```console
vagrant@control:~$ ansible --version
```

La commande nous retourne le bon chemin vers le nouveau fichier de configuration ansible.cfg créé à l'instant : _/home/vagrant/monprojet/ansible.cgf_

![image](./atelier06-2.png)
