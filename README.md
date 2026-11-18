# TP Ansible : Automatisation & DevOps

Projet réalisé dans le cadre d'une évaluation pratique : automatiser avec **Ansible** la mise en place d'une infrastructure, le déploiement d'applications et la montée en version de GitLab.

## Sommaire
- [Architecture](#architecture)
- [Structure du dépôt](#structure-du-dépôt)
- [Prérequis](#prérequis)
- [Utilisation](#utilisation)
- [Contenu des phases](#contenu-des-phases)
- [Notions Ansible utilisées](#notions-ansible-utilisées)
- [Difficultés rencontrées](#difficultés-rencontrées)

## Architecture

```
                 SSH (clé)
  controller ───────────────► node01  (Nginx, GitLab)
  (Ansible)  ───────────────► node02  (Apache)
```

| Machine | Rôle |
|---|---|
| `controller` | Machine de contrôle, seule à avoir Ansible installé |
| `node01` | Nœud géré : Nginx (port 80) et GitLab CE (port 8888) |
| `node02` | Nœud géré : Apache |

- 3 instances **AWS EC2** (Ubuntu), un security group commun.
- Mode **agentless** : aucun agent sur les nœuds, uniquement SSH.

## Structure du dépôt

```
├── inventory/hosts.example        Modèle d'inventaire
├── phase1-infrastructure/         Facts système des nœuds
├── phase2-applications/           Nginx, Apache, GitLab, rôle webserver
├── phase3-gitlab-upgrade/         Montée en version de GitLab
└── docs/captures/                 Captures d'écran des résultats
```

## Prérequis
- Ansible installé sur le controller
- Connexion SSH par clé du controller vers les nœuds
- Les nœuds doivent avoir accès à Internet
- Pour GitLab : au moins 4 Go de RAM sur node01

## Utilisation

1. Copier `inventory/hosts.example` vers `/etc/ansible/hosts` et remplacer les IP.
2. Tester la connexion :
   ```bash
   ansible all -m ping
   ```
3. Lancer un playbook :
   ```bash
   ansible-playbook phase2-applications/install_nginx.yml
   ```

Il est conseillé d'exécuter `--syntax-check` avant chaque lancement.

## Contenu des phases

### Phase 1 : Infrastructure
Création des VMs, installation d'Ansible, connexion SSH sans mot de passe, inventaire, test `ping` et affichage des facts (OS, RAM, CPU, IP) avec `facts.yml`.

### Phase 2 : Déploiement d'applications

| Fichier | Rôle |
|---|---|
| `install_nginx.yml` | Installe Nginx sur node01, déploie une page HTML, handler de redémarrage |
| `install_apache.yml` | Installe Apache sur node02, paramétrable par variables et template Jinja2 |
| `install_gitlab.yml` | Installe GitLab CE et vérifie qu'il répond depuis le controller |
| `roles/webserver` | Rôle réutilisable (Apache ou Nginx), créé avec `ansible-galaxy init` |
| `use_role.yml` | Applique le rôle `webserver` avec des variables différentes |

Choix technique : GitLab écoute sur le port **8888** pour ne pas entrer en conflit avec Nginx (port 80) sur node01.

### Phase 3 : Montée en version de GitLab
`upgrade_gitlab.yml` enchaîne : lecture de la version, sauvegarde (configuration et données), mode maintenance, mise à jour du paquet, `gitlab-ctl reconfigure`, vérification HTTP, migrations de base de données et rapport de version.

- **Idempotent** : la mise à jour n'a lieu que si une version plus récente existe.
- **Sécurisé** : la mise à jour est bloquée sans sauvegarde, et `block/rescue` relance les services en cas d'erreur.
- Le mode maintenance officiel de GitLab étant réservé aux éditions payantes, il est simulé en arrêtant Puma et Sidekiq.

## Notions Ansible utilisées
Inventaire, modules (`apt`, `copy`, `template`, `service`, `uri`), variables, templates Jinja2, handlers, rôles, idempotence, `block/rescue`, `async`, `assert`.

## Difficultés rencontrées

| Problème | Solution |
|---|---|
| Machines qui ne se voient pas | Règle du security group : tout le trafic depuis le groupe lui-même |
| IP publique qui change au redémarrage | IP privées dans l'inventaire |
| Nginx ne démarre pas | Directive en double dans la configuration, remplacée |
| Erreur de parsing YAML | Indentation corrigée, `--syntax-check` et `yamllint` |

## Résultats
[Insérez ici vos captures, par exemple :]

![Ping Ansible](docs/captures/1_ping.png)

## Auteur
[Votre nom], [formation / école], [année]
