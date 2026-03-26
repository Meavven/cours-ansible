# Atelier 14

## Objectif

- Écrivez un playbook myvars1.yml qui affiche respectivement votre voiture et votre moto préférée en utilisant le module debug et deux variables mycar et mybike définies en tant que play vars.

- En utilisant les extra vars, remplacez successivement l'une et l'autre marque - puis les deux à la fois - avant d'exécuter le play.

- Écrivez un playbook myvars2.yml qui fait essentiellement la même chose que myvars1.yml, mais en utilisant une tâche avec set_fact pour définir les deux variables.

- Là aussi, essayez de remplacer les deux variables en utilisant des extra vars avant l'exécution du play.

- Écrivez un playbook myvars3.yml qui affiche le contenu des deux variables mycar et mybike mais sans les définir. Avant d'exécuter le playbook, définissez VW et BMW comme valeurs par défaut pour mycar et mybike pour tous les hôtes, en utilisant l'endroit approprié.

- Effectuez le nécessaire pour remplacer VW et BMW par Mercedes et Honda sur l'hôte target02.

- Écrivez un playbook display_user.yml qui affiche un utilisateur et son mot de passe correspondant à l'aide des variables user et password. Ces deux variables devront être saisies de manière interactive pendant l'exécution du playbook. Les valeurs par défaut seront microlinux pour user et yatahongaga pour password. Le mot de passe ne devra pas s'afficher pendant la saisie.

---

## Challenge

> Lancez les 4 VM et connectez-vous à la VM ```Control Host```

#### Playbook myvars1 
Il utilisera les extra vars pour remplacer successivement l'une et l'autre marque - puis les deux à la fois

_myvars1.yaml_

```yaml
--- # myvars1.yml
- name: Affichage des véhicules (Play Vars)
  hosts: all
  gather_facts: false
  vars:
    mycar: Peugeot
    mybike: Yamaha

  tasks:
    - name: Afficher les véhicules
      debug:
        msg: "Ma voiture : {{ mycar }}, Ma moto : {{ mybike }}"
...
```

> Résultat du lancement du playbook myvars1

```console
[vagrant@control playbooks]$ ansible-playbook myvars1.yml 

PLAY [Affichage des véhicules (Play Vars)] ***************************************************************

TASK [Afficher les véhicules] ****************************************************************************
ok: [target01] => {
    "msg": "Ma voiture : Peugeot, Ma moto : Yamaha"
}
ok: [target02] => {
    "msg": "Ma voiture : Peugeot, Ma moto : Yamaha"
}
ok: [target03] => {
    "msg": "Ma voiture : Peugeot, Ma moto : Yamaha"
}

PLAY RECAP ***********************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@control playbooks]$ ansible-playbook myvars1.yml -e mycar=Ferrari

PLAY [Affichage des véhicules (Play Vars)] ***************************************************************

TASK [Afficher les véhicules] ****************************************************************************
ok: [target01] => {
    "msg": "Ma voiture : Ferrari, Ma moto : Yamaha"
}
ok: [target03] => {
    "msg": "Ma voiture : Ferrari, Ma moto : Yamaha"
}
ok: [target02] => {
    "msg": "Ma voiture : Ferrari, Ma moto : Yamaha"
}

PLAY RECAP ***********************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@control playbooks]$ ansible-playbook myvars1.yml -e mycar=Tesla -e mybike=Ducati

PLAY [Affichage des véhicules (Play Vars)] ***************************************************************

TASK [Afficher les véhicules] ****************************************************************************
ok: [target01] => {
    "msg": "Ma voiture : Tesla, Ma moto : Ducati"
}
ok: [target02] => {
    "msg": "Ma voiture : Tesla, Ma moto : Ducati"
}
ok: [target03] => {
    "msg": "Ma voiture : Tesla, Ma moto : Ducati"
}

PLAY RECAP ***********************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

#### Playbook myvars2.yml
Il fera essentiellement la même chose que myvars1.yml, mais en utilisant une tâche avec set_fact pour définir les deux variables.

_myvars2.yaml_

```yaml
-- # myvars2.yml
- name: Affichage des véhicules (set_fact)
  hosts: all
  gather_facts: false

  tasks:
    - name: Définir les variables dynamiquement
      set_fact:
        mycar: Renault
        mybike: Kawasaki

    - name: Afficher les véhicules
      debug:
	msg: "Ma voiture : {{ mycar }}, Ma moto : {{ mybike }}"
...
```

> Résultat du lancement du playbook myvars2 :

```console
[vagrant@control playbooks]$ ansible-playbook myvars2.yml 

PLAY [Affichage des véhicules (set_fact)] ****************************************************************

TASK [Définir les variables dynamiquement] ***************************************************************
ok: [target01]
ok: [target02]
ok: [target03]

TASK [Afficher les véhicules] ****************************************************************************
ok: [target01] => {
    "msg": "Ma voiture : Renault, Ma moto : Kawasaki"
}
ok: [target03] => {
    "msg": "Ma voiture : Renault, Ma moto : Kawasaki"
}
ok: [target02] => {
    "msg": "Ma voiture : Renault, Ma moto : Kawasaki"
}

PLAY RECAP ***********************************************************************************************
target01                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@control playbooks]$ ansible-playbook myvars2.yml -e mycar=Ferrari

PLAY [Affichage des véhicules (set_fact)] ****************************************************************

TASK [Définir les variables dynamiquement] ***************************************************************
ok: [target01]
ok: [target02]
ok: [target03]

TASK [Afficher les véhicules] ****************************************************************************
ok: [target01] => {
    "msg": "Ma voiture : Ferrari, Ma moto : Kawasaki"
}
ok: [target03] => {
    "msg": "Ma voiture : Ferrari, Ma moto : Kawasaki"
}
ok: [target02] => {
    "msg": "Ma voiture : Ferrari, Ma moto : Kawasaki"
}

PLAY RECAP ***********************************************************************************************
target01                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@control playbooks]$ ansible-playbook myvars2.yml -e mycar=Tesla -e mybike=Ducati

PLAY [Affichage des véhicules (set_fact)] ****************************************************************

TASK [Définir les variables dynamiquement] ***************************************************************
ok: [target01]
ok: [target02]
ok: [target03]

TASK [Afficher les véhicules] ****************************************************************************
ok: [target01] => {
    "msg": "Ma voiture : Tesla, Ma moto : Ducati"
}
ok: [target02] => {
    "msg": "Ma voiture : Tesla, Ma moto : Ducati"
}
ok: [target03] => {
    "msg": "Ma voiture : Tesla, Ma moto : Ducati"
}

PLAY RECAP ***********************************************************************************************
target01                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

#### Playbook myvars3.yml 
Il affichera le contenu des deux variables mycar et mybike mais sans les définir, définissant avant d'être exécuté les variables VW et BMW comme valeurs par défaut pour mycar et mybike pour tous les hôtes, en utilisant l'endroit approprié.

_myvars3.yaml_

```yaml
-- # myvars3.yml
- name: Affichage des véhicules (External Vars)
  hosts: all
  gather_facts: false

  tasks:
    - name: Afficher les véhicules
      debug:
    msg: "Ma voiture : {{ mycar }}, Ma moto : {{ mybike }}"
...
```


> Remplacement des variables VW et BMW par Mercedes et Honda sur l'hôte target02.

```console
[vagrant@control playbooks]$ nano myvars3.yml
[vagrant@control playbooks]$ mkdir group_vars
[vagrant@control playbooks]$ cat <<EOF > group_vars/all.yml
mycar: VW
mybike: BMW
EOF
[vagrant@control playbooks]$ mkdir host_vars
[vagrant@control playbooks]$ cat <<EOF > host_vars/target02.yml
mycar: Mercedes
mybike: Honda
EOF
```

> Résultat du lancement du playbook myvars3 :

```console

[vagrant@control playbooks]$ ansible-playbook myvars3.yml 

PLAY [Affichage des véhicules (External Vars)] ***********************************************************

TASK [Afficher les véhicules] ****************************************************************************
ok: [target01] => {
    "msg": "Ma voiture : VW, Ma moto : BMW"
}
ok: [target02] => {
    "msg": "Ma voiture : Mercedes, Ma moto : Honda"
}
ok: [target03] => {
    "msg": "Ma voiture : VW, Ma moto : BMW"
}

PLAY RECAP ***********************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@control playbooks]$
```


#### playbook display_user.yml
Il affichera un utilisateur et son mot de passe correspondant à l'aide des variables user et password. Ces deux variables seront saisies de manière interactive pendant l'exécution du playbook. Les valeurs par défaut seront microlinux pour user et yatahongaga pour password. Le mot de passe ne sera pas s'affichéc pendant la saisie.

_display_users.yaml_

```yaml
--- # display_users.yml
- hosts: all
  gather_facts: false

  vars_prompt:
    - name: user
      prompt: "Utilisateur"
      default: "microlinux"
      private: false
    - name: password
      prompt: "Mot de passe"
      default: "yatahongaga"
      private: true 

  tasks:
    - name: Afficher les identifiants
      debug:
        msg: "L'utilisateur est {{ user }} et son mot de passe est {{ password }}"
... 
```



> Résultat du lancement du playbook myvars3 :

```console
[vagrant@control playbooks]$ ansible-playbook display_users.yml 
Utilisateur [microlinux]: Daniel
Mot de passe [yatahongaga]: 

PLAY [all] ***********************************************************************************************

TASK [Afficher les identifiants] *************************************************************************
ok: [target01] => {
    "msg": "L'utilisateur est Daniel et son mot de passe est EMA"
}
ok: [target02] => {
    "msg": "L'utilisateur est Daniel et son mot de passe est EMA"
}
ok: [target03] => {
    "msg": "L'utilisateur est Daniel et son mot de passe est EMA"
}

PLAY RECAP ***********************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

