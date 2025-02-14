---
description: https://owasp.org/www-project-securebank/
---

# Install & configure Secure Bank

{% embed url="https://owasp.org/www-project-securebank/" %}

{% embed url="https://github.com/ssrdio/SecureBank" %}

## Setup

> You can setup SecureBank application from source code, or simply pull it from [Docker Hub](https://hub.docker.com/r/ssrd/securebank).

### From source

> Make sure that you have Microsoft SQL Server DB available. You can install or run it inside docker.

1. Install [.NET 5.0 SDK](https://dotnet.microsoft.com/download/dotnet/5.0)
2. Install [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) or just run with [Visual Studio Code](https://code.visualstudio.com/download)
3. Clone from GitHub
4. Navigate to directory SecureBank -> src
5. `dotnet run` or open solution in IDE and run there

### From Docker

1. Install [Docker](https://docs.docker.com/get-docker/)
2. Execute&#x20;

```
docker run -d -p 80:80 -p 5000:5000 -p 1080:1080 -e 'SeedingSettings:Admin=admin@ssrd.io' -e 'SeedingSettings:AdminPassword=admin' ssrd/securebank
```

3. Open [http://localhost:80](http://localhost/) or http:127.0.0.1:80 or add it into /etc/hosts file

<figure><img src="../.gitbook/assets/image (450).png" alt=""><figcaption></figcaption></figure>

### Docker with multiple containers

1. Install [Docker](https://docs.docker.com/get-docker/)
2. Install [Docker Compose](https://docs.docker.com/compose/install/)
3. Clone SecureBank `git clone https://github.com/ssrdio/SecureBank.git`
4. Run `docker-compose up`

### Docker with single container

1. Install [Docker](https://docs.docker.com/get-docker/)
2. Install [Docker Compose](https://docs.docker.com/compose/install/)
3. Create `docker-compose.yml`

```
version: '3'
services:
    securebank:
        image: ssrd/securebank
        environment: 
            - AppSettings:BaseUrl=http://localhost:80
            - AppSettings:Ctf:Enabled=true
            - AppSettings:Ctf:Seed=example
            - AppSettings:Ctf:GenerateCtfdExport=false
            - AppSettings:Ctf:FlagFormat=ctf{{{0}}}
            - AppSettings:Ctf:UseRealChallengeName=true
            - AppSettings:Ctf:Challenges:SqlInjection=true
            - AppSettings:Ctf:Challenges:WeakPassword=true
            - AppSettings:Ctf:Challenges:SensitiveDataExposureStore=true
            - AppSettings:Ctf:Challenges:SensitiveDataExposureBalance=true
            - AppSettings:Ctf:Challenges:SensitiveDataExposureProfileImage=true
            - AppSettings:Ctf:Challenges:PathTraversal=true
            - AppSettings:Ctf:Challenges:Enumeration=true
            - AppSettings:Ctf:Challenges:XxeInjection=true
            - AppSettings:Ctf:Challenges:MissingAuthentication=true
            - AppSettings:Ctf:Challenges:RegistrationRoleSet=true
            - AppSettings:Ctf:Challenges:ChangeRoleInCookie=true
            - AppSettings:Ctf:Challenges:UnconfirmedLogin=true
            - AppSettings:Ctf:Challenges:ExceptionHandlingTransactionCreate=true
            - AppSettings:Ctf:Challenges:ExceptionHandlingTransactionUpload=true
            - AppSettings:Ctf:Challenges:TableXss=true
            - AppSettings:Ctf:Challenges:PortalSearchXss=true
            - AppSettings:Ctf:Challenges:InvalidModelStore=true
            - AppSettings:Ctf:Challenges:InvalidModelTransaction=true
            - AppSettings:Ctf:Challenges:UnknownGeneration=true
            - AppSettings:Ctf:Challenges:HiddenPageRegisterAdmin=true
            - AppSettings:Ctf:Challenges:HiddenPageLoginAdmin=true
            - AppSettings:Ctf:Challenges:InvalidRedirect=true
            - AppSettings:Ctf:Challenges:DirectoryBrowsing=true
            - AppSettings:Ctf:Challenges:Swagger=true
            - AppSettings:Ctf:Challenges:Base2048Content=true
            - AppSettings:Ctf:Challenges:SimultaneousRequest=true
            - AppSettings:Ctf:Challenges:reDOS=true
            - AppSettings:Ctf:Challenges:FreeCredit=true
            - SeedingSettings:Seed=true
            - SeedingSettings:Admin=admin@ssrd.io
            - SeedingSettings:AdminPassword=admin
            - SeedingSettings:UserPassword=test
        ports: 
            - 80:80
            - 1080:1080
        volumes: 
            -  ./logs/securebank:/app/SecureBank/logs
            -  ./logs/storeapi:/app/StoreApi/logs
            - ./ctf:/SecureBank/Ctf
            - ./data:/var/opt/mssql/data
```

4. Run `docker-compose up`

***

### Default users:

```
admin@ssrd.io:admin
developer@ssrd.io:test
yoda@ssrd.io:test
tester@ssrd.io:test
```

### Ports

* 80 on this port SecureBank is accessible
* 1080 is maildev server for user registration
* 5000 is hidden API

### CTF-Mode

If you want to run SecureBank in CTF mode we have also prepared this option. It will create CTFd compatible export file.

Run `docker run -d -p 80:80 -p 5000:5000 -p 1080:1080 -e 'AppSettings:Ctf:Enabled=true' -e 'AppSettings:Ctf:Seed=example' -e 'SeedingSettings:Admin=admin@ssrd.io' -e 'SeedingSettings:AdminPassword=admin' ssrd/securebank`
