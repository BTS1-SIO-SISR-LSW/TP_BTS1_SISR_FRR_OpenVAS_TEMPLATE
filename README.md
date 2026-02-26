# TP BTS SIO 1 – SISR  
## Réseau virtuel FRR + Scan OpenVAS

---

## Objectifs

- Déployer un réseau virtuel avec Containerlab
- Configurer manuellement les adresses IP
- Comprendre le rôle du fichier `daemons`
- Scanner un serveur vulnérable avec OpenVAS
- Analyser un rapport de vulnérabilité

---

## Architecture

- 3 routeurs FRR
- 3 PC
- 1 serveur vulnérable

---

# PARTIE 1 – Configuration

## 1. Fichier daemons

Le fichier doit contenir :

zebra=yes
ospfd=yes
bgpd=no
vtysh_enable=yes

Travail demandé :
- Expliquer le rôle de chaque ligne.

---

## 2. Déploiement

sudo containerlab deploy -t frrlab.clab.yml

---

## 3. Configuration IP (exemple Router1)

docker exec -it clab-frrlab-router1 bash
ip addr add 10.0.12.1/30 dev eth1
ip addr add 10.0.13.1/30 dev eth2
ip addr add 192.168.1.1/24 dev eth3
ip link set eth1 up
ip link set eth2 up
ip link set eth3 up

Répéter pour les autres routeurs et PC selon le plan d’adressage fourni en cours.

---

# PARTIE 2 – OpenVAS (Codespaces)

docker run -d -p 9392:9392 --name openvas mikesplain/openvas

Ouvrir l’onglet Ports → Port 9392  
Accéder à :
https://<nom-du-codespace>-9392.app.github.dev

Identifiants : admin / admin

---

# PARTIE 3 – Scan

1. Créer une Target (192.168.30.10)
2. Créer une Task (Full and Fast)
3. Lancer le scan
4. Identifier :
   - Ports ouverts
   - Vulnérabilités
   - Score CVSS

---

Travail à rendre :
- Schéma réseau
- Explication du fichier daemons
- Capture du rapport
- Mesures correctives

⚠️ Environnement local uniquement.
