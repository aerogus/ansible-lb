# Projet de découverte de Ansible

Dans ce projet nous allons découvrir ansible en provisionnant une architecture simple composée de 3 machines: 1 loadbalancer + 2 serveurs web.

## Conditions initiales

Nous avons 3 VPS fraîchement installés chez l’hébergeur par une image standard "Distribution uniquement" et l’OS `Debian 10`. L’utilisateur standard `debian` a été créé automatiquement, il appartient au groupe `sudo` et le mot de passe a été reçu par email.

## Inventaire

Le paramétrage `DNS` a été effectué chez le [registraire de nom de domaine](https://fr.wikipedia.org/wiki/Registraire_de_nom_de_domaine). Ces hôtes de test étaient actifs pour la durée de l’expérimentation.

| hôte              |
|-------------------|
| 1.dev.aerogus.net |
| 2.dev.aerogus.net |
| 3.dev.aerogus.net |

## Objectifs

- générer et installer les clés ssh pour l’admin via `ansible` (pas possible non ?)
- interdire l’accès ssh par mot de passe (n’autoriser que par clé)
- setter les noms d’hôtes sur les machines
- installer haproxy sur 1
- installer nginx sur 2 et 3
- installer des règles de pare-feu dédiées sur chacune des nodes
- installer des pages html de test du loadbalancing
- installer `vim`
- installer quelques `dotfiles` personnalisés

## Commandes utiles

Sur le poste client `MacOS` avec `homebrew`

```bash
brew install ansible
```

installera la commande à sa version actuelle. Plusieurs commande `ansible*` seront accessibles

```bash
$ ansible<TAB>
ansible             ansible-connection  ansible-doc         ansible-inventory   ansible-pull        ansible-vault
ansible-config      ansible-console     ansible-galaxy      ansible-playbook    ansible-test
```

Générer la paire de clé ssh qui servira à la connexion aux nodes (on utilisera la même partout)
La clé privée sera sur le client `ansible` qui "pousse", la clé publique sera dans les `~/.ssh/authorized_keys` des nodes.

```bash
ssh-keygen -t rsa -b 4096 -f ansible-id_rsa
```

Les fichiers `ansible-id_rsa` (clé privée) et `ansible-id_rsa.pub` (clé publique) sont générés

Les déployer sur les différentes nodes (mot de passe sera demandé)

```bash
ssh-copy-id -i ansible-id_rsa.pub debian@1.dev.aerogus.net
ssh-copy-id -i ansible-id_rsa.pub debian@2.dev.aerogus.net
ssh-copy-id -i ansible-id_rsa.pub debian@3.dev.aerogus.net
```

Déployer les clés d’autorisation sur tout l’inventaire, via un playbook
Bizarre de procéder comme ça, voire impossible ? Avant d’utiliser ansible, il faut que ces clés soient déjà installées "manuellement".

```bash
ansible-playbook -i inventaire.ini --user debian --key-file ansible-id_rsa --extra-vars="pubkey='$(cat ansible-id_rsa.pub)'" deploy_authorized_keys.yml
```

Installer nginx sur les 3 ?

```bash
ansible-playbook -i inventaire.ini --user debian --become --key-file ansible-id_rsa install.yml
```

## Interdire l’accès par mot de passe sur le serveur

Modifier le `/etc/ssh/sshd_config` :

```bash
ChallengeResponseAuthentication no
PasswordAuthentication no
```

(et relancer le serveur ssh)
