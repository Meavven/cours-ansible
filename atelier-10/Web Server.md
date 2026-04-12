# Atelier 10 - Web Server

## Objectif

Écrivez trois playbooks :

- Un premier playbook apache-debian.yml qui installe Apache sur l'hôte debian avec une page personnalisée Apache web server running on Debian Linux.

- Un deuxième playbook apache-rocky.yml qui installe Apache sur l'hôte rocky avec une page personnalisée Apache web server running on Rocky Linux.

- Un troisième playbook apache-suse.yml qui installe Apache sur l'hôte suse avec une page personnalisée Apache web server running on SUSE Linux.

---

## Challenge

> Lancez avant tous vos 4 VM et connectez-vous à la VM de ```Control Host```

> Voici la création de nos trois playbook pour les VM Debian, Rocky et SUSE que nous allons voir en détails :

_playbook-apache-debian.yaml_
```console
---  # playbook-apache-debian.yaml

- hosts: debian

  tasks:

    - name: Update package information
      apt:
        update_cache: true
        cache_valid_time: 3600

    - name: Install Apache
      apt:
        name: apache2

    - name: Install custom web page
      copy:
        dest: /var/www/html/index.html
        mode: 0644
        content: |
          <!doctype html>
          <html>
            <head>
              <meta charset="utf-8">
              <title>Test</title>
            </head>
            <body>
              <h1>Apache web server running on Debian Linux.</h1>
            </body>
          </html>

    - name: Start & enable Apache
      service:
        name: apache2
        state: started
        enabled: true
```
![image](./atelier10-1.png)

_playbook-apache-rocky.yaml_
```console
---  # playbook-apache-rocky.yaml

- hosts: rocky

  tasks:

    - name: Update package information
      dnf:
        update_cache: true

    - name: Install Apache (httpd)
      dnf:
        name: httpd

    - name: Install custom web page
      copy:
        dest: /var/www/html/index.html
        mode: 0644
        content: |
          <!doctype html>
          <html>
            <head>
              <meta charset="utf-8">
              <title>Test</title>
            </head>
            <body>
              <h1>Apache web server running on Rocky Linux.</h1>
            </body>
          </html>

    - name: Start & enable Apache
      service:
        name: httpd
        state: started
        enabled: true
```

![image](./atelier10-2.png)

_playbook-apache-suse.yaml_
```console
--- # playbook-apache-suse.yaml
- hosts: suse
  become: true  # Nécessaire pour l'installation et les services
  tasks:

    - name: Install Apache
      zypper:
        name: apache2
        state: present

    - name: Install custom web page
      copy:
        dest: /srv/www/htdocs/index.html
        mode: '0644'
        content: |
          <!doctype html>
          <html>
            <head>
              <meta charset="utf-8">
              <title>Test</title>
            </head>
            <body>
              <h1>Apache web server running on SUSE Linux.</h1>
            </body>
          </html>

    - name: Start & enable Apache
      service:
        name: apache2
        state: started
        enabled: true
```

![image](./atelier10-3.png)

Ce qui différencie les playbook est leur adaptabilité à leur distribution linux assignée. Par exemple, la distribution Debian utilise l'outil d'installation "apt" contrairement à rocky qui utilise "dnf" et SUSE qui utilise "zypper". Le nom de service de apache vari également. Il se nomme "apache2" avec Debian et SUSE contraiement avec Rocky où il se nomme "httpd".

Mise à part ces 2 détails de nommage, les trois playbook suivent tous la même structure de paramétrage en 3 phases, à commencer par l'installation du service web d'Apache, suivi de l'injection d'un code html dans un nouveau fichier index html grâce au paramètre "copy", et enfin la dernière phase redémarrant le service apache2 avec le paramètre "service" afin qu'il prenne en compte le nouveau fichier injecté.

A l'excepetion du playbook de SUSE qui utilise l'outil d'installation zypper, les autres ont besoin de rafraichir les informations de leur outil de package en amont des phases d'installation du service d'Apache avec le paramètre "update_cache: true"
