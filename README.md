# NiFi Stack

### Stack overview

* Zookeeper
* NiFi Node 1, 2 & 3
* NiFi Certificate Authority (CA)
* NiFi Registry
* Nginx

## Prerequisites
* Install [Docker](https://www.docker.com/)
* Install [Docker Compose](https://docs.docker.com/compose/install/)

### Design
![Low Level Design](./design/low-level.png)

## Custom NiFi Docker
Le conteneur Docker NiFi est une personnalisation de l'image officielle Apache Nifi Docker](https://hub.docker.com/r/apache/nifi) pour étendre la prise en charge du mécanisme d'authentification OpenID Connect (OIDC), permettre l'évolutivité des fichiers de configuration pour les nœuds dynamiques et intégrer le binaire de la boîte à outils TLS NiFi exécuté en mode client pour la génération de certificats de nœud.

### 1. Build Docker
Switch to `nifi` directory
```bash
$ cd nifi
$ docker build -t custom-nifi .
```

## Custom NiFi Registry
Le conteneur Docker NiFi Registry est une personnalisation de l'image Docker officielle du registre Apache Nifi (https://hub.docker.com/r/apache/nifi-registry) pour étendre la prise en charge du mécanisme d'authentification OpenID Connect (OIDC) et du binaire de la boîte à outils TLS nifi intégrée, exécuté en mode client, pour la génération de certificats de nœuds.

### 1. Build Docker
Switch to `nifi-registry` directory
```bash
$ cd nifi-registry
$ docker build -t custom-registry .
```

## Custom NiFi CA Docker
Le conteneur Docker d'autorité de certification NiFi est une personnalisation de l'image Docker officielle d'Apache Nifi Toolkit (https://hub.docker.com/r/apache/nifi-toolkit).
Le cluster NiFi s'appuie aujourd'hui sur NiFi TLS-Toolkit pour générer des keystores, des truststores et des fichiers de configuration sécurisés. Cela simplifie considérablement la génération de certificats et permet une évolutivité vers un nombre illimité de nœuds à la demande, évitant ainsi des processus fastidieux et sujets aux erreurs. Tous les nœuds NiFi intègrent un client d'autorité de certification NiFi (exécutant NiFi Toolkit en mode client) pour demander un certificat de nœud à un serveur d'autorité de certification NiFi indépendant (exécutant NiFi Toolkit en mode serveur). Le client envoie des demandes de signature de certificat (CSR) au serveur, qui les signe ensuite. Cela permet au client et au serveur de valider mutuellement leur identité via un certificat partagé, permettant ainsi une communication TLS mutuelle entre les nœuds et avec les applications externes sans avoir à se faire mutuellement confiance.

### 1. Build Docker
Switch to `nifi-ca` directory
```bash
$ cd nifi-ca
$ docker build -t custom-toolkit .
```

## Deploy NiFi Stack
### Usage (Development)
Pour tester cette pile prête à l'emploi, vous avez besoin d'un fichier `.env` local où sont spécifiées les variables d'environnement qui seront disponibles pour les conteneurs.

```bash
# Pre-install changes, this will perform necessary instructions to run our compose services 
$ ./compose-run.sh
# Start the Docker Compose service
$ docker-compose up
```

```bash
# Check if all containers are up and running
$ docker ps
```

### Usage (Production)
Pour la production, vous devez utiliser le mode Docker Swarm qui orchestre et gère la pile. Pour tester cette pile, exécutez simplement

```bash
# Pre-install changes, this will perform necessary instructions to run our swarm services such as creating secrets
$ ./swarm-run.sh
# Start the Docker Swarm service
$ docker stack deploy -c docker-swarm.yml nifi-stack
```

```bash
# Check if all services are up and running
$ docker service ls
ID             NAME                       MODE         REPLICAS   IMAGE                         PORTS
m0qo6oj54k5i   nifi-stack_init-service    replicated   0/1        busybox:latest
ix29qu4c8vmc   nifi-stack_nginx           replicated   1/1        nginx:1.21.6                  *:80->80/tcp, *:443->443/tcp
oraxq4ha2d7w   nifi-stack_nifi-1          replicated   1/1        custom-nifi:latest            *:30006->8443/tcp
zdtoff9h07my   nifi-stack_nifi-2          replicated   1/1        custom-nifi:latest            *:30007->8443/tcp
jp0ac4jhn6vy   nifi-stack_nifi-3          replicated   1/1        custom-nifi:latest            *:30008->8443/tcp
9bvsuohrsy03   nifi-stack_nifi-ca         replicated   1/1        custom-toolkit:latest         *:30009->9999/tcp
lv7uevazcyty   nifi-stack_nifi-registry   replicated   1/1        apache/nifi-registry:1.15.3   *:18080->18080/tcp
t2hbu9984rkx   nifi-stack_zookeeper       replicated   1/1        zookeeper:3.6.2               *:30005->2181/tcp
```

Après avoir déployé l'environnement logiciel, vous devriez avoir accès à l'interface utilisateur.
- NiFi: `https://localhost/nifi/`
- NiFi Registry: `http://localhost/nifi-registry/`


## References

- [How to start using NiFi](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#how-to-install-and-start-nifi)
- [How to start using NiFi Registry](https://nifi.apache.org/docs/nifi-registry-docs/html/administration-guide.html#how-to-install-and-start-nifi-registry)
