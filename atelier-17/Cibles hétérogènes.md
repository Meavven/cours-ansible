```markdown
# Atelier 17 - Cibles hétérogènes (NTP/Chrony)

## Objectif
L'objectif est d'uniformiser la configuration du service de synchronisation temporelle **Chrony** sur un parc composé de différentes distributions Linux (Debian, Ubuntu, Rocky Linux et openSUSE). Cet atelier permet d'aborder la gestion des différences de noms de paquets, de services et de chemins de fichiers selon la famille d'OS.

---

## 1. Playbook `chrony-01.yml` : Méthode "Gros Sabots"
Cette approche consiste à multiplier les tâches en utilisant les modules natifs (`apt`, `dnf`, `zypper`) et en appliquant des conditions `when` strictes pour chaque distribution.

**Fichier : `chrony-01.yml`**
```yaml
---
- name: Installation Chrony (Méthode Gros Sabots)
  hosts: all
  become: true
  tasks:
    # Installation spécifique par gestionnaire de paquets
    - name: Installer Chrony sur Debian/Ubuntu
      ansible.builtin.apt:
        name: chrony
        update_cache: true
      when: ansible_os_family == "Debian"

    - name: Installer Chrony sur Rocky
      ansible.builtin.dnf:
        name: chrony
      when: ansible_distribution == "Rocky"

    - name: Installer Chrony sur openSUSE
      ansible.builtin.zypper:
        name: chrony
      when: ansible_distribution == "openSUSE Leap"

    # Configuration commune
    - name: Configurer Chrony
      ansible.builtin.copy:
        dest: /etc/chrony.conf
        content: |
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Redémarrer Chrony

    # Gestion du service (le nom varie selon la famille d'OS)
    - name: Démarrer Chrony (Debian/Ubuntu)
      ansible.builtin.service:
        name: chrony
        state: started
        enabled: true
      when: ansible_os_family == "Debian"

    - name: Démarrer Chrony (Rocky/SUSE)
      ansible.builtin.service:
        name: chronyd
        state: started
        enabled: true
      when: ansible_os_family != "Debian"

  handlers:
    - name: Redémarrer Chrony
      ansible.builtin.service:
        name: "{{ (ansible_os_family == 'Debian') | ternary('chrony', 'chronyd') }}"
        state: restarted
...
```

---

## 2. Playbook `chrony-02.yml` : Méthode "Subtile" (Générique)
Cette approche est plus élégante et maintenable. Elle utilise `set_fact` pour définir les variables spécifiques à chaque OS en début de play, puis utilise des modules génériques (`package`) pour les tâches d'exécution.

**Fichier : `chrony-02.yml`**
```yaml
---
- name: Installation Chrony (Méthode Subtile)
  hosts: all
  become: true
  tasks:
    # 1. Définition dynamique des variables selon l'OS
    - name: Paramètres pour Debian/Ubuntu
      ansible.builtin.set_fact:
        chrony_package: chrony
        chrony_service: chrony
        chrony_conf: /etc/chrony/chrony.conf
      when: ansible_os_family == "Debian"

    - name: Paramètres pour Rocky / openSUSE
      ansible.builtin.set_fact:
        chrony_package: chrony
        chrony_service: chronyd
        chrony_conf: /etc/chrony.conf
      when: ansible_os_family != "Debian"

    # 2. Tâches génériques utilisant les variables
    - name: Mettre à jour le cache (Debian seulement)
      ansible.builtin.apt:
        update_cache: true
      when: ansible_os_family == "Debian"

    - name: Installer le paquet Chrony
      ansible.builtin.package:
        name: "{{ chrony_package }}"
        state: present

    - name: Déployer la configuration
      ansible.builtin.copy:
        dest: "{{ chrony_conf }}"
        content: |
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Relancer Chrony

    - name: Activer et démarrer le service
      ansible.builtin.service:
        name: "{{ chrony_service }}"
        state: started
        enabled: true

  handlers:
    - name: Relancer Chrony
      ansible.builtin.service:
        name: "{{ chrony_service }}"
        state: restarted
...
```

---

## Vérification du fonctionnement
Après l'exécution de l'un ou l'autre playbook, la synchronisation est vérifiée sur tous les hôtes avec la commande suivante :

```bash
ansible all -a "chronyc sources"
```

**Points d'attention identifiés :**
1. **Noms de services** : `chrony` sur les bases Debian, `chronyd` sur RedHat/SUSE.
2. **Chemins de configuration** : `/etc/chrony/chrony.conf` sur Debian/Ubuntu contre `/etc/chrony.conf` sur les autres.
3. **Module `package`** : Très utile pour l'abstraction, il appelle automatiquement le bon module (`apt`, `dnf`, etc.) selon l'hôte.
```
