
# Atelier 17 - Cibles hétérogènes (NTP/Chrony)

## Objectif
L'objectif est d'uniformiser la configuration du service de synchronisation temporelle **Chrony** sur un parc composé de différentes distributions Linux (Debian, Ubuntu, Rocky Linux et openSUSE). Cet atelier permet d'aborder la gestion des différences de noms de paquets, de services et de chemins de fichiers selon la famille d'OS.

---

## 1. Playbook `chrony-01.yml` : Méthode "Gros Sabots"
Cette méthode consiste à multiplier les tâches en utilisant les modules natifs (`apt`, `dnf`, `zypper`) et en appliquant des conditions `when` strictes pour chaque distribution.

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
> Verification de la configuration chrony du playbook _chrony-01.yml_ sur les VM Targets :

```console
[vagrant@ansible playbooks]$ ansible all -a "chronyc sources"
debian | CHANGED | rc=0 >>
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
+ 27.ip-51-68-44.eu             4   6   377    14   -421us[ -421us] +/-   25ms
* neel.ch                       1   6   377    15   +333us[ -654us] +/-   22ms
- ns3161189.ip-51-91-67.eu      3   6   377    13  +2600us[+2600us] +/-   75ms
+ ciran28.fr                    3   6   377    16  -1210us[-2194us] +/-   62ms
suse | CHANGED | rc=0 >>
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
+ 82-64-45-50.subs.proxad.>     1   6   377    14  -2512us[-2512us] +/-   22ms
+ 200-177-190-109.dsl.ovh.>     2   6   377    14  +1676us[+1676us] +/-   18ms
+ main.arcanite.ch              2   6   377    14   -134us[ -134us] +/-   55ms
* dns.freewebworld.fr           1   6   377    15  -3457us[-4675us] +/-   21ms
rocky | CHANGED | rc=0 >>
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
- ntp.tuxfamily.net             2   7    15     6  -5625us[-5625us] +/-   40ms
* 164-102-58-31.static.rev>     2   6   377    15  -4743us[-5649us] +/-   18ms
- y.ns.gin.ntt.net              2   6   377    15    -68us[  -68us] +/-  100ms
+ meshflow.net                  2   6   377    17  -1606us[-2512us] +/-   30ms
ubuntu | CHANGED | rc=0 >>
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
? prod-ntp-3.ntp4.ps5.cano>     0   6     0     -     +0ns[   +0ns] +/-    0ns
- alphyn.canonical.com          2   6   175    10    +38ms[  +38ms] +/-  118ms
+ prod-ntp-4.ntp1.ps5.cano>     2   6   377    13  -2125us[-2125us] +/-   24ms
+ prod-ntp-3.ntp1.ps5.cano>     2   6   377    13  -1356us[-1356us] +/-   25ms
+ 82-65-248-56.subs.proxad>     1   6   377    14    +23ms[  +24ms] +/-   58ms
* time.cloudflare.com           3   6   377    13  +3599us[+4910us] +/-   22ms
+ edge1.haeiven.fr              3   6   377    14   -787us[ +521us] +/-   21ms
+ silas.dioptre.fr              2   6   377    16   -833us[ +473us] +/-   49ms
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
> Verification de la configuration chrony du playbook _chrony-02.yml_ sur les VM Targets :

```console
[vagrant@ansible playbooks]$ ansible all -a "chronyc sources"
suse | CHANGED | rc=0 >>
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
+ 82-64-45-50.subs.proxad.>     1   6   377     6   +992us[ +722us] +/-   18ms
+ 200-177-190-109.dsl.ovh.>     2   6   377     3  +2694us[+2694us] +/-   20ms
- main.arcanite.ch              2   7   377    68  -3533us[-3796us] +/-   55ms
* dns.freewebworld.fr           1   7   377     6   +443us[ +173us] +/-   18ms
rocky | CHANGED | rc=0 >>
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
- ntp.tuxfamily.net             2   7   277   123    -21ms[  -21ms] +/-   57ms
* 164-102-58-31.static.rev>     2   7   377     7  -4421us[-4494us] +/-   19ms
- y.ns.gin.ntt.net              2   7   377     5  -5466us[-5466us] +/-  115ms
+ meshflow.net                  2   7   377     6  +1540us[+1540us] +/-   27ms
ubuntu | CHANGED | rc=0 >>
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
* edge1.haeiven.fr              3   6    17    18   +884us[+1661us] +/-   22ms
- 129.151.225.244               2   6    17    20    +40ms[  +40ms] +/-   88ms
- web1.ciran28.fr               2   6    17    19  -1297us[-1297us] +/-   60ms
- 82-64-45-50.subs.proxad.>     1   6    17    19    -39ms[  -39ms] +/-   56ms
debian | CHANGED | rc=0 >>
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
* edge1.haeiven.fr              3   6    17    20  -1036us[ -684us] +/-   23ms
- 37.59.63.125                  2   6    17    19    +36ms[  +36ms] +/-   64ms
- web1.ciran28.fr               2   6    17    19   +105us[ +105us] +/-   64ms
- ciran28.fr                    3   6    17    19  +4343us[+4343us] +/-   60ms
**Points d'attention identifiés :**
1. **Noms de services** : `chrony` sur les bases Debian, `chronyd` sur RedHat/SUSE.
2. **Chemins de configuration** : `/etc/chrony/chrony.conf` sur Debian/Ubuntu contre `/etc/chrony.conf` sur les autres.
3. **Module `package`** : Très utile pour l'abstraction, il appelle automatiquement le bon module (`apt`, `dnf`, etc.) selon l'hôte.
```
