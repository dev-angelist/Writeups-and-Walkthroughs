---
description: https://github.com/digininja/DVWA
---

# Install & configure OWASP Mutillidae II

{% embed url="https://owasp.org/www-project-mutillidae-ii/" %}

{% embed url="https://github.com/webpwnized/mutillidae" %}

## Installation Guides

### Standard Installation - DockerHub

* [How to Run Mutillidae from DockerHub Images](https://www.youtube.com/watch?v=c1nOSp3nagw)

### Alternative Installation - Docker

* [How to Install Docker on Ubuntu](https://www.youtube.com/watch?v=Y_2JVREtDFk)
* [How to Run Mutillidae on Docker](https://www.youtube.com/watch?v=9RH4l8ff-yg)
* [How to Run Mutillidae from DockerHub Images](https://www.youtube.com/watch?v=c1nOSp3nagw)

### Alternative Installation - Google Cloud

* [How to Run Mutillidae on Google Kubernetes Engine (GKE)](https://www.youtube.com/watch?v=uU1eEjrp93c)

### Legacy Installation - LAMP Stack

If you have a LAMP stack set up already, you can skip directly to installing Mutillidae. Check out our [comprehensive installation guide](https://github.com/webpwnized/mutillidae/blob/main/README-INSTALLATION.md) for detailed instructions. Watch the video tutorial: [How to Install Mutillidae on LAMP Stack](https://www.youtube.com/watch?v=TcgeRab7ayM)

## Installation via Docker - My case

### Install Docker

If you haven't it, install Docker on your machine (debian/kali): [How to Install Docker on Ubuntu](https://www.youtube.com/watch?v=Y_2JVREtDFk)

### Install Docker Image

```bash
git clone https://github.com/webpwnized/mutillidae-docker.git
```

And build the docker file

```bash
cd mutillidae-docker
docker compose -f .build/docker-compose.yml up --build --detach
```

### Website URL

The web application should be running at localhost, then we ca go there via browser

[http://127.0.0.1/](http://127.0.0.1/)

Note: The first time the webpage is accessed, a warning webpage will be displayed referencing the database cannot be found. This is the expected behaviour. Just use the link to "rebuild" the database and it will start working normally.

### Build/Reset DB

<figure><img src="../.gitbook/assets/image (45) (1) (1).png" alt=""><figcaption></figcaption></figure>

1. [Click here](https://127.0.0.1/set-up-database.php) to attempt to setup the database. Sometimes this works.
2. Be sure the username and password to MySQL is the same as configured in includes/database-config.inc
3. Be aware that MySQL disables password authentication for root user upon installation or update in some systems. This may happen even for a minor update. Please check the username and password to MySQL is the same as configured in includes/database-config.inc
4. A [video is available](https://www.youtube.com/watch?v=sG5Z4JqhRx8) to help reset MySQL root password
5. Check the error message below for more hints
6. If you think this message is a false-positive, you can opt-out of these warnings below

Alternatively, you can trigger the database build.

```bash
# Requesting Mutillidae database be built.
curl http://127.0.0.1/set-up-database.php;
```

### Populating the LDAP database

The LDAP database is empty upon build. Add users to the LDAP database using the following command.

```bash
# Install LDAP Utilities including ldapadd
sudo apt-get update
sudo apt-get install -y ldap-utils

# Add users to the LDAP database
ldapadd -c -x -D "cn=admin,dc=mutillidae,dc=localhost" -w mutillidae -H ldap://localhost:389 -f .build/ldap/configuration/ldif/mutillidae.ldif
```

### Using a script to test the web interface

You can test if the web site is responsive

```
# This should return the index.php home page content
curl http://127.0.0.1:8888/;
```

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><a href="https://127.0.0.1/index.php?page=home.php">https://127.0.0.1/index.php?page=home.php</a></p></figcaption></figure>

### TMI

#### Running Services

Once the containers are running, the following services are available on localhost.

* Port 80, 8080: Mutillidae HTTP web interface
* Port 81: MySQL Admin HTTP web interface
* Port 82: LDAP Admin web interface
* Port 443: HTTPS web interface
* Port 389: LDAP interface
