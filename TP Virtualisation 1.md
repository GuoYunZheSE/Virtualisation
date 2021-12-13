# **TP Virtualisation** 1

[Yunzhe GUO](mailto:yunzhe.guo@etu.univ-nantes.fr)

**Année universitaire 2021-2022**

# Part-1

## 1 Mise en place.

## 1.1 Prérequis

![LXD](https://lehn-r.univ-nantes.io/tp-virtualisation-s9/TP1/img/lxd.png)**LXD**

Vous utiliserez [LXD](https://lxd.readthedocs.io/en/latest/) comme première illustration de conteneur. LXD s’appuie sur [lxc](https://linuxcontainers.org/lxc/introduction/) -qui est une interface du noyau linux permettant d’isoler un processus ou un ensemble de processus du reste du système- et sur un démon (serveur *lxd*) accessible en ligne de commande ou au travers d’une [API REST](https://linuxcontainers.org/lxd/docs/master/rest-api). Ce serveur permet la configuration des [instances](https://linuxcontainers.org/lxd/docs/master/instances) (essentiellement des [containers](https://linuxcontainers.org/lxd/docs/master/containers) pour l’instant, même si le support de [machines virtuelles](https://linuxcontainers.org/lxd/docs/master/virtual-machines) est en cours de développement), des [images](https://images.linuxcontainers.org/) qui peuvent être préfabriquées ou [construites à la demande](https://linuxcontainers.org/distrobuilder/introduction/) de systèmes sur lesquelles s’appuient ces dernières et un certain nombre de paramètres de configuration : réseau, stockage, contrôle d’accès, etc. Il permet la [migration à chaud](https://linuxcontainers.org/lxd/docs/master/migration). Des limites peuvent être configurées sur l’utilisation des ressources (CPU, mémoire, disque, bande passante, etc). Enfin, des accès direct au matériel peuvent être configurés (par exemple pour le [GPU](https://ubuntu.com/blog/nvidia-cuda-inside-a-lxd-container) ou des périphériques branchés sur un [hub USB](https://stgraber.org/2017/03/27/usb-hotplug-with-lxd-containers/)).

### 1.1.1 ![img](https://lehn-r.univ-nantes.io/tp-virtualisation-s9/TP1/img/tux.png) Linux

- **Ubuntu**

```
sudo snap install lxd
```

![image-20211202214219619](/Users/lucas/Library/Application Support/typora-user-images/image-20211202214219619.png)

# 2 Premiers pas avec [LXD](https://lxd.readthedocs.io/en/latest/)

## 2.1 LXD et nftables

Debian 10 Buster et Ubuntu 20.04 Focal proposent `nftables` comme par-feu par défaut. LXD ne supporte pas encore `nftables`, il faut donc adapter. Pour commencer, nous allons utiliser les `iptables` “à l’ancienne” :

```SHELL
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
```

Cette solution ne fonctionne pas pour l’IPv6. Il faut donc désactiver l’IPv6 pour les conteneurs lors du `lxd init`:

```
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none

[Response]
We should use none
```

## 2.2 Initialisation du service (daemon)

L’[initialisation du service lxd](https://linuxcontainers.org/lxd/getting-started-cli/#initial-configuration) (interface en ligne de commande et points d’entrée HTTP/REST) se fait au moyen de la commande

```
lxd init
```

Le script de commandes associé à `lxd init` vous posera des questions de manière interactive (c’est son comportement par défaut), mais il est possible de l’utiliser de manière [non interactive](https://lxd.readthedocs.io/en/latest/preseed/) au moyen d’un fichier [YAML](https://yaml.org/spec/1.2/spec.html) de paramètres ([*preseed*](https://lxd.readthedocs.io/en/latest/preseed/)).

Par exemple :

```
sudo lxd init
```

avec une configuration interactive :

```
Would you like to use LXD clustering? (yes/no) [default=no]:
Do you want to configure a new storage pool? (yes/no) [default=yes]:
Name of the new storage pool [default=default]: o4ssd512g
Name of the storage backend to use (btrfs, dir, lvm, zfs, ceph) [default=btrfs]: btrfs
Would you like to create a new btrfs subvolume under /var/snap/lxd/common/lxd? (yes/no) [default=yes]:
Would you like to connect to a MAAS server? (yes/no) [default=no]:
Would you like to create a new local network bridge? (yes/no) [default=yes]:
What should the new bridge be called? [default=lxdbr0]:
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]:
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]:
Would you like LXD to be available over the network? (yes/no) [default=no]:
Would you like stale cached images to be updated automatically? (yes/no) [default=yes]
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: yes
config: {}
networks:
- config:
ipv4.address: auto
ipv6.address: auto
description: ""
name: lxdbr0
type: ""
project: default
storage_pools:
- config:
source: /var/snap/lxd/common/lxd/storage-pools/o4ssd512g
description: ""
name: o4ssd512g
driver: btrfs
profiles:
- config: {}
description: ""
devices:
eth0:
name: eth0
network: lxdbr0
type: nic
root:
path: /
pool: o4ssd512g
type: disk
name: default
cluster: null
```

Il peut être nécessaire de bénéficier des droits suffisants :

```
sudo adduser $LOGNAME lxd
newgrp lxd
```

## 2.3 Lancement des instances

Une instance de container peut être créée au moyen de la commande [`lxc launch`](https://linuxcontainers.org/lxd/getting-started-cli/#launch-an-instance) avec la syntaxe

`lxc launch` *template* *nom du container*

Par exemple

```
lxc launch images:debian/10 buster01
```

Permet de créer un container nommé `buster01` démarré sur une distribution [Debian Buster](https://wiki.debian.org/fr/DebianBuster).

![image-20211202220810655](/Users/lucas/Library/Application Support/typora-user-images/image-20211202220810655.png)

## 2.4 État des instances

La commande `lxc ls` permet de vérifier la liste des instances et leur état. Par exemple

```
lxc ls
```

devrait vous conduire à quelque chose qui ressemble à ça :

![image-20211202220921408](/Users/lucas/Library/Application Support/typora-user-images/image-20211202220921408.png)

```
+----------+---------+----------------------+-----------------------------------------------+-----------+-----------+
|   NAME   |  STATE  |         IPV4         |                     IPV6                      |   TYPE    | SNAPSHOTS |
+----------+---------+----------------------+-----------------------------------------------+-----------+-----------+
| buster01 | RUNNING | 10.138.158.46 (eth0) | fd42:8168:b5e7:1e08:216:3eff:fee3:871e (eth0) | CONTAINER | 0         |
+----------+---------+----------------------+-----------------------------------------------+-----------+-----------+
```

- L’option `-c` de `lxc ls` permet de spécifier les colonnes à afficher, par exemple

```
lxc ls -c ns
```

![image-20211202220952619](/Users/lucas/Library/Application Support/typora-user-images/image-20211202220952619.png)

n’affiche que le nom et l’état de chaque instance. Vous pouvez consulter l’aide associée à chaque commande au moyen d’un

```
lxc help ls
```

(de manière similaire aux commandes de `git` ou `iproute2`).

- L’option

  ```
  --format
  ```

  permet de spécifier un format de sortie alternatif, par exemple :

  - `json` pour avoir l’état des containers en format [JSON](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Objets_globaux/JSON)
  - `yaml` pour l’avoir en [YAML](https://yaml.org/spec/1.2/spec.html)
  - `csv` pour l’avoir en CSV

## 2.5 Arrêt / redémarrage

- `lxc stop` *nom du container* permet de stopper un container (sans le détruire),
- `lxc start` *nom du container* permet de le redémarrer.

Par exemple :

```
lxc stop buster01
lxc ls -c ns buster01
+----------+---------+
|   NAME   |  STATE  |
+----------+---------+
| buster01 | STOPPED |
+----------+---------+
lxc start buster01
lxc ls -c ns buster01
+----------+---------+
|   NAME   |  STATE  |
+----------+---------+
| buster01 | RUNNING |
+----------+---------+
```

![image-20211202221802232](/Users/lucas/Library/Application Support/typora-user-images/image-20211202221802232.png)

## 2.6 Création de processus dans un container

La commande `lxc exec` permet d’exécuter une commande dans un container. Par exemple

```
lxc exec buster01 -- /bin/bash
```

permet d’avoir un shell exécuté dans le contexte d’exécution du container `buster01`.

![image-20211202222404983](/Users/lucas/Library/Application Support/typora-user-images/image-20211202222404983.png)

## 2.7 Exercices

1. Créez un container `buster01` (si votre système hôte est linux) ou `focal01` (si votre système hôte est linux ou windows ou autre).exit

   ![image-20211202220810655](/Users/lucas/Library/Application Support/typora-user-images/image-20211202220810655.png)

2. Commentez les sorties des commandes `date`, `cat /proc/cpuinfo`, `ls /dev`, `free -h`, `df -h`, `ip link ls`, `ip route ls` et `systemctl status` exécutées dans l’environnement d’exécution de votre container.

   **[Response]**

   ##### Date

   ![image-20211202223718582](/Users/lucas/Library/Application Support/typora-user-images/image-20211202223718582.png)

   It's the time of my linux host machine.

   ##### cat /proc/cpuinfo

   ![image-20211202223803240](/Users/lucas/Library/Application Support/typora-user-images/image-20211202223803240.png)

   It's the same  CPU as my physical machine.

   ##### Ls /dev

   ![image-20211202223930293](/Users/lucas/Library/Application Support/typora-user-images/image-20211202223930293.png)

   There is one dev more than the traditional linux:**lxd**, it's because the container system should communicate with the host by this dev.

   ##### Free -h

   ![截屏2021-12-02 22.41.06](/Users/lucas/Desktop/截屏2021-12-02 22.41.06.png)

   The total memory are the same size, but the used one are different.

   ##### Df -h

   ![image-20211202224324264](/Users/lucas/Library/Application Support/typora-user-images/image-20211202224324264.png)

   The disk is different in container.

   ##### Ip link ls

   ![image-20211202224439412](/Users/lucas/Library/Application Support/typora-user-images/image-20211202224439412.png)

   The host system has 2 ip more than the container system. And they has different ip address.

   ##### Ip route ls

   ![image-20211202224705069](/Users/lucas/Library/Application Support/typora-user-images/image-20211202224705069.png)

   The host system has 2 route more than the container system. 

   ##### Systemctl status

   ![image-20211202224829876](/Users/lucas/Library/Application Support/typora-user-images/image-20211202224829876.png)

   They have different processes. But all initialed by `/sbin/init splash`

3. Installez et configurez un serveur ssh dans le container. Est-il accessible ? Si oui comment ? Si non pourquoi ? 10.172.232.153

   **[Response]**

   No, I can't.

   ![image-20211202230527575](/Users/lucas/Library/Application Support/typora-user-images/image-20211202230527575.png)

4. Créez un second container appelé `nginx01`, de la même façon que ce que vous avez fait pour les 3 exercices précédents. Installez le serveur web [nginx](http://nginx.org/) dessus.

   ![image-20211202231548098](/Users/lucas/Library/Application Support/typora-user-images/image-20211202231548098.png)

