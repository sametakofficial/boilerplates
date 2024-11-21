## In this repo, I will share some setup of reverse proxies for linux servers, incase we will use docker compose as a first step then configrue traefik, then connect other docker images such as portainer and mailcow etc.

# Traefik Intallation

#### for you to understand the file structure, I put the schema bellow;

```bash

./containers
├──./traefik
    ├── data
    │   ├── acme.json
    │   ├── config.yml
    │   └── traefik.yml
    └── cf_api_token.txt
    └── docker-compose.yml
    └── .env

```

#### lets create a new directory named containers;

```bash
    mkdir /containers
    cd /containers

```

#### next step we will create another directory for traefik installation. In contuinues steps we will create directories for portainer , mailcow and other contaiers so it wil be listed in one main directory

```bash

    mkdir /traefik
    cd /traefik

```

#### now , we will create a docker-compose.yaml (it's important to be .yaml) file for install traefik, next step we will create credentials and data directory for traefik.yml (now this is .yml not .yaml) and acme.json. What ı mean by secrets is our Cloudlare API token for let's encrypt ssl generation and a password for dashboard of traefik.

```bash

    docker create network proxy

    touch docker-compose.yaml

    cat docker-compose.yaml

```

#### then, you can edit your docker-compose.yaml file with your envorientments.

```yaml
version: "3.8"

services:
  traefik:
    image: traefik:v3.2
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      - proxy
    ports:
      - 80:80
      - 443:443
      # - 443:443/tcp # Uncomment if you want HTTP3
      # - 443:443/udp # Uncomment if you want HTTP3
    environment:
      CF_DNS_API_TOKEN_FILE: /run/secrets/cf_api_token # note using _FILE for docker secrets
      # CF_DNS_API_TOKEN: ${CF_DNS_API_TOKEN} # if using .env
      TRAEFIK_DASHBOARD_CREDENTIALS: ${TRAEFIK_DASHBOARD_CREDENTIALS}
    secrets:
      - cf_api_token
    env_file: .env # use .env
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./data/traefik.yml:/traefik.yml:ro
      - ./data/acme.json:/acme.json
      # - ./data/config.yml:/config.yml:ro
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.entrypoints=http"
      - "traefik.http.routers.traefik.rule=Host(`traefik-dashboard.your_domain_name.com`)"
      - "traefik.http.middlewares.traefik-auth.basicauth.users=${TRAEFIK_DASHBOARD_CREDENTIALS}"
      - "traefik.http.middlewares.traefik-https-redirect.redirectscheme.scheme=https"
      - "traefik.http.middlewares.sslheader.headers.customrequestheaders.X-Forwarded-Proto=https"
      - "traefik.http.routers.traefik.middlewares=traefik-https-redirect"
      - "traefik.http.routers.traefik-secure.entrypoints=https"
      - "traefik.http.routers.traefik-secure.rule=Host(`traefik-dashboard.your_domain_name.com`)"
      - "traefik.http.routers.traefik-secure.middlewares=traefik-auth"
      - "traefik.http.routers.traefik-secure.tls=true"
      - "traefik.http.routers.traefik-secure.tls.certresolver=cloudflare"
      - "traefik.http.routers.traefik-secure.tls.domains[0].main=your_domain_name.com"
      - "traefik.http.routers.traefik-secure.tls.domains[0].sans=*.your_domain_name.com"
      - "traefik.http.routers.traefik-secure.service=api@internal"

secrets:
  cf_api_token:
    file: ./cf_api_token.txt

networks:
  proxy:
    external: true
```

##### · Change "your_domain_name.com" with your domain

##### · Change the network named proxy with your network name (if you did as I did it's not necesery)

#### Now we will create our cloudflare token and add user and md5 password for traefik dashboard

```bash

#If you dont have htpasswd library run the command bellow
sudo apt install apache2-utils

#You can change user name, I'm using it as admin
echo $(htpasswd -nB admin) | sed -e s/\\$/\\$\\$/g

#It will ask you a password, enter the password that you want to use in your traefik dashboard

#Then, copy result of the command

#creating .env file for the username and password
touch .env
nano .env

```

#### Then we will enter our htpasswd result into .env file that just created. Important thing is name of the envorientmet variable is same as in docker-compose.yaml files

`  TRAEFIK_DASHBOARD_CREDENTIALS: ${TRAEFIK_DASHBOARD_CREDENTIALS}`

#### area

```.env

TRAEFIK_DASHBOARD_CREDENTIALS=user:$$2y$$05$$lSaEi.G.aIygyXRdiFpt7OqmUMW9QUG5I1N.j0bXoXxIjxQmoGOWu

```
