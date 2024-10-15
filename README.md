# Rapport de TP : Services, processus signaux

## 1. Secure Shell : SSH

### 1.1 Connexion SSH
Pour me connecter en tant que root, j'ai d'abord utilisé les commandes suivantes pour installer le service SSH :
```bash
sudo apt update
sudo apt install openssh-server
```

#### Configuration SSH
Pour permettre la connexion root distante avec mot de passe, j'ai modifié le fichier de configuration SSH (`/etc/ssh/sshd_config`). J'ai changé l'option `PermitRootLogin` de `prohibit-password` à `yes`. 

**Explication** :
- `PermitRootLogin yes` : Permet la connexion root avec un mot de passe.
- **Avantages** : Facilité d'accès pour les administrateurs.
- **Inconvénients** : Vulnérabilité aux attaques par force brute. Préférable de ne pas l'utiliser en production.

### 1.2 Authentification par clé
J'ai généré une paire de clés SSH sur ma machine hôte avec la commande suivante :
```bash
ssh-keygen -t rsa -b 2048
```
Je n'ai pas mis de passphrase pour simplifier le processus, mais c'est une mauvaise idée en raison des risques de sécurité accrus.

### 1.3 Dépôt de la clé publique sur le serveur
Pour pouvoir me connecter via la clé publique, j'ai dû créer un dossier `.ssh` dans `/root` et y ajouter ma clé publique :
```bash
mkdir -p /root/.ssh
echo "ma_clé_publique" >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
```

### 1.4 Connexion avec la clé
J'ai essayé de me connecter à mon serveur avec :
```bash
ssh -i ~/.ssh/id_rsa root@192.168.56.1 -p 3022
```
Cependant, j'ai rencontré des problèmes de connexion, notamment :
```
Permission denied (publickey).
```
Cela était dû à l'absence de la clé publique sur le serveur.

### 1.5 Sécurisation de l'accès SSH
Pour sécuriser l'accès SSH, j'ai changé `PermitRootLogin yes` à `PermitRootLogin prohibit-password` et j'ai ajouté `PasswordAuthentication no` dans le fichier `sshd_config` pour désactiver l'authentification par mot de passe.

**Attaques par force brute** : Tentatives répétées pour deviner un mot de passe. En utilisant uniquement des clés, on réduit ce risque.

## 2. Processus

### 2.1 Étude des processus UNIX
J'ai utilisé la commande `ps` pour afficher les processus :
```bash
ps aux
```
**Information TIME** : Temps CPU utilisé par le processus.

J'ai trouvé que le premier processus lancé après le démarrage est généralement `init` ou `systemd`.

### 2.2 Processus père
Pour afficher le PPID, j'ai utilisé :
```bash
ps -o pid,ppid,cmd
```
J'ai pu lister les processus ancêtres de ma commande en cours.

### 2.3 Commande pstree
Pour voir les processus sous forme d'arbre, j'ai installé et utilisé `pstree` :
```bash
sudo apt install pstree
pstree
```

### 2.4 Commande top
J'ai utilisé `top` pour visualiser les processus en temps réel et j'ai pu trier par utilisation de la mémoire. La touche `?` m'a aidé à naviguer dans l'aide de `top`.

### 2.5 Commande htop
`htop` offre une interface plus conviviale et interactive par rapport à `top`. Cependant, il n'est pas installé par défaut et nécessite une installation préalable.

## 3. Arrêt d’un processus
J'ai écrit deux scripts shell :
- `date.sh`
- `date-toto.sh`

Pour arrêter ces processus, j'ai utilisé `CTRL-Z` pour les mettre en arrière-plan, puis `fg` pour les ramener au premier plan et `CTRL-C` pour les arrêter.

## 4. Tubes
La différence entre `tee` et `cat` :
- `cat` : Lit et affiche le contenu.
- `tee` : Lit et affiche le contenu tout en l'écrivant dans un fichier.

### Commandes
```bash
$ ls | cat
$ ls -l | cat > liste
$ ls -l | tee liste
$ ls -l | tee liste | wc -l
```

## 5. Journal système rsyslog
Pour vérifier si `rsyslog` est lancé :
```bash
systemctl status rsyslog
```

### Fichiers de log
`rsyslog` écrit dans `/var/log/syslog` pour les messages standards.

### Service cron
Le service `cron` permet d'exécuter des tâches programmées.

### Commande tail -f
Cette commande affiche les nouvelles lignes ajoutées à un fichier en temps réel. J'ai pu observer les logs lors du redémarrage de `cron`.

### Logrotate
Le fichier `/etc/logrotate.conf` gère la rotation des logs pour éviter de trop gros fichiers.

### Dmesg
J'ai examiné la sortie de `dmesg` pour voir le modèle de processeur et de carte réseau détectés.

## Difficultés rencontrées
- **Clés SSH** : J'ai eu du mal à comprendre la gestion des clés privées et publiques, ainsi que la manière de les transférer correctement entre ma machine hôte et mon serveur.
- **Configuration SSH** : Modifier le fichier `sshd_config` et comprendre les implications de chaque option m'a posé problème.

---

**Conclusion** : Ce TP m'a permis de mieux comprendre la gestion des connexions SSH et des processus Unix, tout en soulignant l'importance de la sécurité dans l'administration système.
