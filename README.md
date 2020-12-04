# Initial page

## Schéma de l'infrastructure virtuelle locale

## Story 1

### Instructions

**Story 001 - Acquisition de compétence théorique** \(Business Value : 3 / Dev effort : 2\)\*\*

* [Wikipedia](https://fr.wikipedia.org/wiki/Network_Time_Protocol)
* [NTP.orp](http://support.ntp.org/bin/view/Main/WebHome)
* [Frameip.com](https://www.frameip.com/ntp/)
* [Serveurs sources](https://tf.nist.gov/tf-cgi/servers.cgi)

Voici les points qui doivent être étudié \(non exaustive\):

* Quel type d'architecture ?
* Quels sont les composants du NTP ?
* Comment les serveurs NTP se protègent d'éventuelles attaques ?
* Ports, protocoles utilisés
* Objectif du NTP
* SNTP ?

### Réponses

#### Quel type d'architecture ?

L'architecture NTP prévoit

* la diffusion verticale arborescente de proche en proche d'une heure de référence à partir d'une ou plusieurs machines racines garantes d'une grande précision[4](https://fr.wikipedia.org/wiki/Network_Time_Protocol#cite_note-4). Dans cette arborescence, chaque nœud choisit parmi ses nœuds parents, celui qui présente les meilleures garanties de qualité et hérite au passage d'un attribut nommé stratum qu'il transmet à ses descendants. Les machines de stratum 1 sont les machines racines et à chaque traversée d'un nœud ce nombre augmente d'une unité. Ce stratum est une mesure de la distance d'un nœud aux machines racines, il est considéré comme un indicateur de la qualité de synchronisation qu'une machine donnée peut offrir à ses descendants.
* la diffusion latérale à des machines paires d'une heure commune. Cette diffusion vient en complément de la précédente ; elle permet à ces machines de partager une référence de temps qui leur est commune. Cette diffusion améliore la résilience de cette architecture NTP dans le sens où elle permet de suppléer une déficience locale/temporaire de connectivité vers les machines racines, voire de permettre à un groupe de machines de conserver entre elles une même référence relative en l'absence de machines racines.

Dans la terminologie NTP, les serveurs de stratum 1 sont appelés serveurs primaires, et les autres sont appelés serveurs secondaires.

Chaque nœud de cette architecture doit être configuré en lui indiquant au minimum quels sont ses serveurs parents et/ou collatéraux. C'est à la charge de chaque utilisateur de réaliser localement cette configuration. C'est cette agrégation de configurations qui, de proche en proche, crée le réseau NTP, il n'est pas préexistant ni même configuré de façon centralisée. Cette architecture est flexible extensible et robuste, mais c’est à la charge des utilisateurs d’y contribuer.



#### Quels sont les composants du NTP ?

* NTP needs some reference clock that defines the true time to operate. All clocks are set towards that true time. \(It will not just make all systems agree on some time, but will make them agree upon the true time as defined by some standard.\)

  NTP uses UTC as reference time.

* NTP is a fault-tolerant protocol that will automatically select the best of several available time sources to synchronize to. Multiple candidates can be combined to minimize the accumulated error. Temporarily or permanently insane time sources will be detected and avoided.
* NTP is highly scalable: A synchronization network may consist of several reference clocks. Each node of such a network can exchange time information either bidirectional or unidirectional. Propagating time from one node to another forms a hierarchical graph with reference clocks at the top.
* Having available several time sources, NTP can select the best candidates to build its estimate of the current time. The protocol is highly accurate, using a resolution of less than a nanosecond \(about 2^-32 seconds\). \(The popular protocol used by **rdate** and defined in \[RFC 868\] only uses a resolution of one second\).
* Even when a network connection is temporarily unavailable, NTP can use measurements from the past to estimate current time and error.
* For formal reasons NTP will also maintain estimates for the accuracy of the local time.

#### Source :

{% embed url="http://www.ntp.org/ntpfaq/NTP-s-def.htm\#AEN1298" %}

#### Comment les serveurs NTP se protègent d'éventuelles attaques 

Les serveurs NTP se protègent de différentes manières contres les attaques.

La majorité des serveurs NTP bloquent les IP des machines exécutant trop de requêtes en peu de temps \(voir Story2 &gt; Remarques &gt; Paramètre iburst

#### Ports, protocoles utilisés

 NTP est un protocole basé sur UDP et utilise le port 123.

#### Objectif du NTP

It is an Internet protocol used to synchronize the clocks of computers to some time reference

Time usually just advances. If you have communicating programs running on different computers, time still should even advance if you switch from one computer to another. Obviously if one system is ahead of the others, the others are behind that particular one. From the perspective of an external observer, switching between these systems would cause time to jump forward and back, a non-desirable effect.

L'un des effets non désirés d'un décalage dans le temps entre deux machines, pourrait modifier la date de péremption d'un certificat et donc les données ne seraient plus cryptées entre les deux machines, ce qui permettrait la lecture des données

Un autre effet peut être simplement le refus de communication des machines entre elles, ce qui aura pour cause d'isoler la machine

#### SNTP

 **Simple Network Time Protocol** \(**SNTP**\) is a less complex implementation of NTP, using the same protocol but without requiring the storage of state over extended periods of time. It is used in some embedded system and in applications where full NTP capability is not required.

#### Source :

{% embed url="https://en.wikipedia.org/wiki/Network\_Time\_Protocol\#SNTP" %}

## Story 2

### Instructions



**Story 002 - Mettre en place un lien avec une horloge atomique de notre choix \(Business Value : 5 / Dev effort : 1\)**

Description : L'instance SSH synchronise son temps avec time-d-b.nist.gov \(primaire\) et utcnist2.colorado.edu \(colorado\)

#### **given**

* Je suis connecté sur l'instance SSH.
* Je modifie l'heure système.

#### **when**

* Je lance \(force\) une synchronisation.

#### **then**

* Le log du ntp mentionne les interactions avec les horloges sources.
* Le temps est à nouveau synchronisé.

### Réponses

```text
#!/bin/bash
## Titre : story2.sh
## Description : Ce script installe et configure le client ntp de la machine courante et modifie le serveur ntp de reference vers un serveur aux us
## Date : 27.11.2020
## Auteurs : Chassot Richard, Karoui Mehdi, Angeloz Roderick
## Teste sur : debian 10
## Execute as root



## Installation de paquets aptitude et  NTP
apt-get install aptitude -y
aptitude install ntp -y

## Modification des serveurs de temps utilises
sed -i 's/pool 0.debian.pool.ntp.org iburst/server 1.north-america.pool.ntp.org/g' /etc/ntp.conf
sed -i 's/pool 1.debian.pool.ntp.org iburst/server 2.north-america.pool.ntp.org/g' /etc/ntp.conf
sed -i 's/pool 2.debian.pool.ntp.org iburst/server 3.north-america.pool.ntp.org/g' /etc/ntp.conf
sed -i 's/pool 3.debian.pool.ntp.org iburst/server 4.north-america.pool.ntp.org/g' /etc/ntp.conf

## Re synchronisation du temps avec les bons parametres
/etc/init.d/ntp stop
ntpd -gq
/etc/init.d/ntp start

## Affichage du serveur utilise pour la synchro
echo -e "\e[31mTime synced with server below : \e[0m"
ntpq -p

## Aide completion US 2
echo -e "\e[31mTo continue with the user story, run command to change time\e[0m"
echo "sudo date --set 1998-11-02"
echo "sudo /etc/init.d/ntp stop"
echo "sudo ntpd -gq"
echo "sudo /etc/init.d/ntp start"
```

#### Remarques

Le paramètre iburst 

IBURST envoie une rafale de huit paquets lorsque le serveur est inaccessible \(tente de savoir si un hôte est joignable\), puis raccourcit le temps jusqu'à la première synchronisation. Nous spécifions le NTP IBURST pour une synchronisation d'horloge plus rapide. Cette option est considérée comme «agressive» par certains serveurs NTP publics. Si un serveur NTP ne répond pas, le mode IBURST continue d'envoyer des requêtes fréquentes jusqu'à ce que le serveur réponde et que la synchronisation de l'heure démarre.



Nous n'utiliserons donc pas le paramètre iburst car nous utilisons un pool de serveurs ntp qui répondront en cas de défaillance d'un des serveurs du pool

## Story 3

### Instructions

**Story 003 - Mettre en place un client NTP se connectant à notre serveur source \(Business Value : 5 / Dev effort : 3\)**

Description : l'instance Domain Controler synchronise son temps avec l'instance SSH

**given**

* Je suis connecté sur l'instance Domain Controler.
* Je modifie l'heure.
* Je démontre que les horloges sources sont bien celles mentionnées.
* Je montre le pare-feu activé et l'exception qui a été ajoutée pour permettre aux clients de synchroniser leur horloge.

**when**

* Je lance \(force\) une synchronisation.

**then**

* Le log du ntp mentionne les interactions avec les horloges sources.
* Le temps est à nouveau synchronisé.
* Je démontre que les horloges sources sont bien celles mentionnées.
* NTP est bien un service \(persistence après un redémarrage\).

#### Scripts

Client CentOS 8

```text
#!/bin/bash
## Titre : story2centosntp.sh
## Description : Ce script installe et configure le client ntp vers un serveur specifie
## Date : 27.11.2020
## Auteurs : Chassot Richard, Karoui Mehdi, Angeloz Roderick
## Teste sur : centos 8
## Execute as root

##Installation du daemon chronyd client ntp
dnf install chronyd
systemctl enable chronyd


##Remplacement de la ligne de configuration indiquant le serveur ntp
sed -i 's/pool 2.centos.pool.ntp.org iburst/server 10.229.40.2/g' /etc/chrony.conf
systemctl restart chronyd

sleep 2

echo -e "\e[31mNTP Server list : \e[0m"
chronyc sources
```

## Story 4

### Instructions

**Story 004 - Sécuriser son serveur NTP \(Business Value : 5 / Dev Effort : 4\)**

Description : Sécuriser son serveur NTP

La sécurisation du serveur NTP passe en partie par la mise en place d'un firewall logiciel qui bloque l'accès en entrée au serveur NTP sur le port 123 en UDP par des clients étrangers au LAN:

```text
#!/bin/bash
## Titre : firewallscript.sh
## Description : Ce script installe et configure le firewall du serveur SSH
## Date : 27.11.2020
## Auteurs : Chassot Richard, Karoui Mehdi, Angeloz Roderick
## Teste sur : debian 10
## Execute as root

apt install ufw -y

## Modification  des regles relatives aux ICMP de maniere a ne plus y repondre
sed -i '/ufw-before-input.*icmp/s/ACCEPT/DROP/g' /etc/ufw/before.rules

## Configuration initiale pour autoriser le trafic sortant uniquement
systemctl start ufw
systemctl enable ufw
ufw default allow outgoing
ufw default deny incoming

## Mise en place des exceptions SSH et NTP en entree et sortie car les clients
## viendront se synchroniser dessus et nous aurons besoin d'un tunnel SSH pour
## atteindre les clients
ufw allow ssh
ufw allow proto udp from 10.229.32.0/20 to any port 123
ufw --force enable
```

{% embed url="https://Redhat.com" %}



