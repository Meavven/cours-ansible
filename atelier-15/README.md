# Atelier 15

## Objectif

- Écrivez un playbook kernel.yml qui affiche les infos détaillées du noyau sur tous vos Target Hosts. Utilisez la commande uname -a et le module debug avec le paramètre msg.

- Essayez d'obtenir le même résultat en utilisant le paramètre var du module debug.

- Écrivez un playbook packages.yml qui affiche le nombre total de paquets RPM installés sur les hôtes rocky et suse (rpm -qa | wc -l).

---

## Challenge

> Lancez les 4 VM et connectez-vous à la VM ```Control Host```

```console
[vagrant@ubuntu:atelier-7] vagrant up
[vagrant@ubuntu:atelier-7] vagrant ssh ansible
vagrant@ansible:~$ 
```

### Création du playbook ```kernel.yaml``` pour afficher les infos détaillées du noyau sur tous les ```Target Hosts```

```yaml
---  # kernel.yml
- name: Affichage des infos noyau (msg)
  hosts: all
  gather_facts: false

  tasks:
    - name: Recuperer les infos uname
      command: uname -a
      register: uname_cmd
      changed_when: false

    - name: Afficher le resultat
      debug:
    msg: "Le noyau de cette machine est : {{ uname_cmd.stdout }}"
...
```

> Résultat du lancement du playbook ```kernel.yaml```

```console
[vagrant@ansible playbooks]$ ansible-playbook kernel.yml 

PLAY [Affichage des infos noyau (msg)] *******************************************************************

TASK [Recuperer les infos uname] *************************************************************************
ok: [rocky]
ok: [suse]
ok: [debian]

TASK [Afficher le resultat] ******************************************************************************
ok: [rocky] => {
    "msg": "Le noyau de cette machine est : Linux rocky 5.14.0-570.52.1.el9_6.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Oct 15 13:59:22 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux"
}
ok: [debian] => {
    "msg": "Le noyau de cette machine est : Linux debian 6.1.0-40-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.153-1 (2025-09-20) x86_64 GNU/Linux"
}
ok: [suse] => {
    "msg": "Le noyau de cette machine est : Linux suse 6.4.0-150600.23.73-default #1 SMP PREEMPT_DYNAMIC Tue Oct  7 08:43:02 UTC 2025 (46f6a23) x86_64 x86_64 x86_64 GNU/Linux"
}

PLAY RECAP ***********************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
