# TP BTS SIO 1 - SISR
## Création d’un réseau virtuel FRR avec Containerlab et scan avec OpenVAS

Durée : 2 séances de 2 heures  
Bloc de compétences : Bloc 3 - Cybersécurité des services informatiques  

---

# Contexte professionnel

Vous êtes technicien systèmes et réseaux au sein d’une PME spécialisée dans les services numériques.  
L’entreprise dispose de plusieurs sites interconnectés via des routeurs.

Avant de déployer une nouvelle infrastructure en production, votre responsable vous demande de :

- Simuler l’architecture réseau dans un environnement virtualisé.
- Mettre en place un routage dynamique entre les équipements.
- Identifier les vulnérabilités d’un serveur exposé sur le réseau interne.
- Proposer des mesures correctives pour sécuriser l’infrastructure.

---

# PARTIE 1 - Création du réseau virtuel

## Architecture

- 3 routeurs FRR interconnectés en triangle
- 3 PC (un derrière chaque routeur)
- 1 serveur vulnérable derrière le routeur 3

---

## 1. Configuration du fichier daemons (à réaliser par l’étudiant)

Chaque routeur doit posséder le fichier `daemons` suivant :

```
zebra=yes
ospfd=yes
bgpd=no
vtysh_enable=yes
```

### Travail demandé

- Expliquer le rôle de chaque ligne de cette configuration.
- Pourquoi zebra est-il indispensable ?
- Pourquoi BGP est-il désactivé dans ce TP ?

---

## 2. Déploiement et configuration manuelle des adresses IP

### Déployer le lab

```bash
sudo containerlab deploy -t frrlab.clab.yml
```

---

## Plan d’adressage

- R1-R2 : 10.0.12.0/30
- R1-R3 : 10.0.13.0/30
- R2-R3 : 10.0.23.0/30
- LAN1 : 192.168.1.0/24
- LAN2 : 192.168.2.0/24
- LAN3 : 192.168.3.0/24
- Serveur : 192.168.30.0/24

---

## Configuration Router1

```bash
docker exec -it clab-frrlab-router1 bash
ip addr add 10.0.12.1/30 dev eth1
ip addr add 10.0.13.1/30 dev eth2
ip addr add 192.168.1.1/24 dev eth3
ip link set eth1 up
ip link set eth2 up
ip link set eth3 up
```

---

## Configuration Router2

```bash
docker exec -it clab-frrlab-router2 bash
ip addr add 10.0.12.2/30 dev eth1
ip addr add 10.0.23.1/30 dev eth2
ip addr add 192.168.2.1/24 dev eth3
ip link set eth1 up
ip link set eth2 up
ip link set eth3 up
```

---

## Configuration Router3

```bash
docker exec -it clab-frrlab-router3 bash
ip addr add 10.0.13.2/30 dev eth1
ip addr add 10.0.23.2/30 dev eth2
ip addr add 192.168.3.1/24 dev eth3
ip addr add 192.168.30.1/24 dev eth4
ip link set eth1 up
ip link set eth2 up
ip link set eth3 up
ip link set eth4 up
```

---

## Configuration PC1

```bash
docker exec -it clab-frrlab-PC1 bash
ip addr add 192.168.1.10/24 dev eth1
ip route add default via 192.168.1.1
```

---

## Configuration PC2

```bash
docker exec -it clab-frrlab-PC2 bash
ip addr add 192.168.2.10/24 dev eth1
ip route add default via 192.168.2.1
```

---

## Configuration PC3

```bash
docker exec -it clab-frrlab-PC3 bash
ip addr add 192.168.3.10/24 dev eth1
ip route add default via 192.168.3.1
```

---

## Configuration Serveur

```bash
docker exec -it clab-frrlab-serveur bash
ip addr add 192.168.30.10/24 dev eth1
ip route add default via 192.168.30.1
```

---

# PARTIE 2 - Installation d’OpenVAS (en environnement Codespaces)

Lancer OpenVAS :

```bash
docker run -d -p 9392:9392 --name openvas mikesplain/openvas
```

Dans Codespaces :

1. Cliquer sur l’onglet "Ports"
2. Ouvrir le port 9392
3. Accéder à l’URL fournie par Codespaces :

```
https://<nom-du-codespace>-9392.app.github.dev
```

Identifiants :

```
admin / admin
```

---

# PARTIE 3 - Lancer un scan avec commandes GMP (ligne de commande)

## Créer une Target

```bash
docker exec -it openvas gvm-cli socket --xml '
<create_target>
  <name>Serveur_DVWA</name>
  <hosts>192.168.30.10</hosts>
</create_target>'
```

---

## Créer une tâche de scan rapide

```bash
docker exec -it openvas gvm-cli socket --xml '
<create_task>
  <name>Scan_Rapide_DVWA</name>
  <config id="daba56c8-73ec-11df-a475-002264764cea"/>
  <target id="ID_TARGET"/>
</create_task>'
```

---

## Lancer le scan

```bash
docker exec -it openvas gvm-cli socket --xml '
<start_task task_id="ID_TASK"/>'
```

---

## Identifier dans le rapport

- Les ports ouverts
- Les vulnérabilités détectées
- Le score CVSS
