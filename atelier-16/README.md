# Atelier 16

## Objectif

- Écrivez successivement deux playbooks pour mettre en place la synchronisation NTP avec Chrony sur vos quatre Target Hosts sous Debian, Rocky Linux, SUSE Linux et Ubuntu. Il vous faudra identifier le nom du paquet, le service correspondant et le chemin vers le fichier de configuration par défaut pour chacune des distributions.

Voici la configuration qu'il faudra installer sur chacune des quatre cibles :

\# _chrony.conf_ :
```sh
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```

- Le premier playbook chrony-01.yml utilisera les modules de gestion de paquets natifs apt, dnf et zypper et s'inspirera de la méthode « gros sabots » utilisée plus haut dans cet atelier.

- Le deuxième playbook chrony-02.yml définira trois variables chrony_package, chrony_service et chrony_confdir et utilisera le module de gestion de paquets générique package.

---

## Challenge

> Lancez les 4 VM et connectez-vous à la VM ```Control Host```

### Création du premier playbook ```chrony-01.yml```

> Ansible détecte automatiquement quel outil est utilisé pour gérer les paquets (apt, dnf, yum, zypper, etc.) via le fact ansible_pkg_mgr.
> Le playbook chrony-01.yml utilisera les modules de gestion de paquets natifs apt, dnf et zypper et s'inspirera de la méthode « gros sabots »

_chrony-01.yml_

```yaml
--- # pkg-info.yml
- name: Informations sur le gestionnaire de paquets
  hosts: all
  gather_facts: true

  tasks:
    - name: Afficher le gestionnaire de paquets
      debug:
        msg: "L'hôte {{ inventory_hostname }} utilise : {{ ansible_pkg_mgr }}"
```

> Résultat du lancement du playbook _chrony-01.yml_ :

```console
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
### Création du second playbook ```chrony-02.yml```
