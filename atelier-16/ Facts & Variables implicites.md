# Atelier 16

## Objectif

Écrivez trois playbooks pour afficher des informations sur chacun des Target Hosts :

- pkg-info.yml pour afficher le gestionnaire de paquets utilisé

- python-info.yml pour afficher la version de Python installée

- dns-info.yml pour afficher le(s) serveur(s) DNS utilisé(s)

---

## Challenge

> Lancez les 4 VM et connectez-vous à la VM ```Control Host```

### Création du premier playbook ```pkg-info.yml```

> Ansible détecte automatiquement quel outil est utilisé pour gérer les paquets (apt, dnf, yum, zypper, etc.) via le fact ansible_pkg_mgr.

pkg-info.yml affichera le gestionnaire de paquets utilisé

```yaml
--- # pkg-info.yml
- name: Informations sur le gestionnaire de paquets
  hosts:=
[vagrant@ansible playbooks]$ ansible-playbook pkg-info.yml 

PLAY [Informations sur le gestionnaire de paquets] *******************************************************

TASK [Gathering Facts] ***********************************************************************************
ok: [rocky]
ok: [suse]
ok: [debian]

TASK [Afficher le gestionnaire de paquets] ***************************************************************
ok: [rocky] => {
    "msg": "L'hôte rocky utilise : dnf"
}
ok: [debian] => {
    "msg": "L'hôte debian utilise : apt"
}
ok: [suse] => {
    "msg": "L'hôte suse utilise : zypper"
}

PLAY RECAP ***********************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

---
### Création du second playbook ```python-info.yml```

> Ce playbook affichera la version de Python installée

```yaml
--- # python-info.yml
- hosts: all
  gather_facts: true

  tasks:
    - name: Afficher la version complète
      debug:
        var: ansible_python_version
...
```

> Résultat du lancement du playbook _python-info.yml_ :

```console
[vagrant@ansible playbooks]$ ansible-playbook python-info.yml 

PLAY [all] ***********************************************************************************************

TASK [Gathering Facts] ***********************************************************************************
ok: [suse]
ok: [rocky]
ok: [debian]

TASK [Afficher la version complète] **********************************************************************
ok: [rocky] => {
    "ansible_python_version": "3.9.21"
}
ok: [debian] => {
    "ansible_python_version": "3.11.2"
}
ok: [suse] => {
    "ansible_python_version": "3.6.15"
}

PLAY RECAP ***********************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

> Ce playbook affichera le(s) serveur(s) DNS utilisé(s)

```yaml
--- # dns-info.yml

- hosts: all
  gather_facts: true

  tasks:
    - name: Afficher les serveurs de noms (DNS)
      debug:
        msg: "Les serveurs DNS de {{ inventory_hostname }} sont : {{ ansible_dns.nameservers | join(', ') }}"
```

> Résultat du lancement du playbook dns-info.yml :

```console
[vagrant@ansible playbooks]$ ansible-playbook dns-info.yml 

PLAY [all] ***********************************************************************************************

TASK [Gathering Facts] ***********************************************************************************
ok: [suse]
ok: [rocky]
ok: [debian]

TASK [Afficher les serveurs de noms (DNS)] ***************************************************************
ok: [suse] => {
    "msg": "Les serveurs DNS de suse sont : 10.0.2.3"
}
ok: [rocky] => {
    "msg": "Les serveurs DNS de rocky sont : 10.0.2.3"
}
ok: [debian] => {
    "msg": "Les serveurs DNS de debian sont : 10.0.2.3"
}

PLAY RECAP ***********************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
