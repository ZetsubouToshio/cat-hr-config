# preparations
## create network
    docker network create kong-net

# keycloak startup
## create volume 
    docker volume create keycloak-vol
## copy cert and key to cert folder
    sudo cp cert/fullchain.pem /var/lib/docker/volumes/keycloak-vol/_data/tls.crt && \
    sudo cp cert/privkey.pem /var/lib/docker/volumes/keycloak-vol/_data/tls.key
## run keycloack
(config for test purrrrposes)

    docker run -d --name keycloak \
    --network=kong-net \
    -v "keycloak-vol:/etc/x509/https" \
    -e PROXY_ADDRESS_FORWARDING=true \
    -e KEYCLOAK_HOSTNAME=zetsuboutoshio.tk \
    -e KEYCLOAK_USER=admin \
    -e KEYCLOAK_PASSWORD=admin \
    -e DB_VENDOR=H2 \
    -p 8443:8443 \
    -p 8080:8080 \
    jboss/keycloak

## run keycloack
(sam kinda of prod)

    docker run -d --name keycloak \
    --network=kong-net \
    -v "keycloak-vol:/etc/x509/https" \
    -e PROXY_ADDRESS_FORWARDING=true \
    -e KEYCLOAK_HOSTNAME=zetsuboutoshio.tk \
    -e KEYCLOAK_USER=admin \
    -e KEYCLOAK_PASSWORD=admin \
    -e DB_VENDOR=postgres \
    -e DB_ADDR= \
    -e DB_USER= \
    -e DB_PASSWORD= \
    -p 8443:8443 \
    -p 8080:8080 \
    jboss/keycloak    

## client config
https://scalac.io/user-authentication-keycloak-1/

## admin dashboard
https://hostname:8443/auth/admin/

# webui startup
## run webui
    docker run -itd --name cat-hr-webui \
        --network=kong-net \
        -p 3000:80 \
        --rm registry.gitlab.com/cat-hr/cat-hr-webui:latest

# backend startup
## run backend 

    docker run -itd --name search-orchestrator \
    --network=kong-net \
    -e SPRING_DATASOURCE_URL= \
    -e SPRING_DATASOURCE_USERNAME= \
    -e SPRING_DATASOURCE_PASSWORD= \
    -p 3001:3001 \
    registry.gitlab.com/cat-hr/search-orchestrator:latest

# kong docker startup (DB LESS)
## create docker volume
    docker volume create kong-vol
## create king config file
/var/lib/docker/volumes/kong-vol/_data/kong.yml

https://docs.konghq.com/1.2.x/db-less-and-declarative-config/#the-declarative-configuration-format

stored into /config/kong/kong.yml

could be checked:
    
    kong config -c kong.conf parse kong.yml
## ssl

copy sertificate (fullchain.pem) and private key (privkey.pem) to /var/lib/docker/volumes/kong-vol/_data/

sudo cp fullchain.pem /var/lib/docker/volumes/kong-vol/_data/tls.crt
sudo cp privkey.pem /var/lib/docker/volumes/kong-vol/_data/tls.key

## run kong    
    docker run -d --name kong \
     --network=kong-net \
     -v "kong-vol:/usr/local/kong/declarative" \
     -e "KONG_DATABASE=off" \
     -e "KONG_DECLARATIVE_CONFIG=/usr/local/kong/declarative/kong.yml" \
     -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
     -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
     -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
     -e "KONG_SSL=on" \
     -e "KONG_SSL_CERT=/usr/local/kong/declarative/fullchain.pem" \
     -e "KONG_SSL_CERT_KEY=/usr/local/kong/declarative/privkey.pem" \
     -p 80:8000 \
     -p 443:8443 \
     -p 8001:8001 \
     -p 8444:8444 \
     kong:latest