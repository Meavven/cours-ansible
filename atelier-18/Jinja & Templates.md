# Atelier 18 – Gestion des templates avec Jinja2 & Ansible

## Objectif

L'objectif de cet atelier est d'automatiser l'installation et la configuration du service de synchronisation temporelle Chrony sur différentes distributions Linux. La particularité ici est l'utilisation du module template de Jinja2 pour gérer dynamiquement les fichiers de configuration.

Pour cela, nous écrire un playbook chrony.yml qui installe un fichier de configuration personnalisé sur vos cibles. La première ligne de commentaire devra indiquer le chemin complet vers le fichier qui peut différer selon la distribution Linux concernée.

---

## Challenge

Initialisation de l'environnement

Nous commençons par démarrer l'infrastructure et nous positionner dans le répertoire de travail approprié sur le nœud de contrôle.

# Démarrage des VM
vagrant up

# Connexion au serveur Ansible
vagrant ssh ansible

# Accès au répertoire du projet
cd ansible/projets/ema/playbooks/

Création du Playbook principal
Le playbook chrony_playbook.yml utilise des variables dynamiques pour s'adapter à la distribution cible (Debian, Ubuntu, Rocky, OpenSUSE).

```yaml
--- 

hosts: all
tasks: 
name: Paramètres des distributions
  includevars: >
    chrony{{ansible_distribution|lower|replace(" ", "-") }}.yml
name: Update package Debian family
    apt:
      update_cache: true
      cache_valid_time: 3600
    when: ansible_os_family == "Debian"

    
name: Install Chrony
    package:
      name: "{{chrony_package}}"

    
name: Starting service chrony
    service:
      name: "{{chrony_service}}"
      state: started
      enabled: true

    
name: Install custom config file
    template:
      dest: "{{chrony_confdir}}/chrony.conf"
      mode: 0644
      src: chrony.conf
    notify: Reload chrony

  handlers:
    
name: Reload chrony
    service:
      name: "{{chrony_service}}"
      state: restarted
...
```

> **Note :** La différence majeure avec l'atelier précédent réside dans l'utilisation de template au lieu de copy, permettant d'injecter des variables directement dans le fichier de configuration.
> 
## 4. Définition des variables par distribution
Pour que le playbook soit générique, nous avons créé des fichiers de variables spécifiques dans le répertoire playbooks.

### Fichier chrony_debian.yml

```yaml
---
chrony_package: chrony
chrony_service: chrony
chrony_confdir: /etc/chrony
...
```

### Fichier chrony_opensuse-leap.yml

```yaml
---
chrony_package: chrony
chrony_service: chronyd
chrony_confdir: /etc
...
```

### Fichier chrony_rocky.yml

```yaml
---
chrony_package: chrony
chrony_service: chronyd
chrony_confdir: /etc
...
```

### Fichier chrony_ubuntu.yml

```yaml
---
chrony_package: chrony
chrony_service: chrony
chrony_confdir: /etc/chrony
...
```

## 5. Configuration du template Jinja2
Le template source est placé dans le dossier templates. Il inclut une variable Jinja2 dans l'en-tête pour identifier dynamiquement le chemin de configuration.
**Chemin :** ansible/projets/ema/playbooks/templates/chrony.conf.j2

```yaml
{{chrony_confdir}}/chrony.conf
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```
## 6. Exécution et Validation
Une fois le playbook exécuté, nous vérifions le bon déploiement sur une machine cible (Debian).
bash
Sortie du serveur Ansible
exit

Connexion à la VM cible
vagrant ssh debian

Vérification du contenu du fichier généré
cat /etc/chrony/chrony.conf


Le fichier contient bien les paramètres définis et l'en-tête a été correctement interprété par le moteur Jinja2.
