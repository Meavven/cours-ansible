# Atelier 12

## Objectif

Écrivez un playbook chrony.yml qui assure la synchronisation NTP de tous vos Target Hosts :

- Installez le paquet chrony.

- Activez et démarrez le service chronyd correspondant.

- Jetez un œil sur le fichier de configuration /etc/chrony.conf fourni par défaut.

- Installez une configuration personnalisée (cf. ci-dessous).

- Prenez en compte cette nouvelle configuration.

- Vérifiez la syntaxe correcte de votre playbook chrony.yml.

- Vérifiez l'idempotence de votre playbook.

Voici la configuration à installer :

_/etc/chrony.conf_

```sh
# /etc/chrony.conf
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```

---

## Challenge

> Lancez les 4 VM et connectez-vous à la VM ```Control Host```

```console
[vagrant@ubuntu:atelier-7] vagrant up
[vagrant@ubuntu:atelier-7] vagrant ssh ansible
vagrant@ansible:~$ 
```

### Créer un playbook pour installer chrony

_playbook-chrony.yaml_

```yaml
  GNU nano 5.6.1                                   chrony.yml                                             
--- # chrony.yml
- hosts: all
  become: true

  tasks:
    - name: Mise a jour du cache des paquets
      dnf:
    update_cache: yes

    - name: Installation du paquet chrony
      dnf:
    name: chrony
        state: present

    - name: Mise en place de la configuration personnalisée
      copy:
    dest: /etc/chrony.conf
        owner: root
        group: root
        mode: '0644'
        content: |
          # /etc/chrony.conf
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst

          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Redemarrer chronyd

    - name: le service chronyd est demarre et active
      service:
    name: chronyd
        state: started
        enabled: true

  handlers:
    - name: Redemarrer chronyd
      service:
    name: chronyd
        state: restarted
```

Résultat du lancement :

![image](./atelier12-1.png)
