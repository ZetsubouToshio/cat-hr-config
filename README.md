#preparations
## create network
    docker network create kong-net

# webui startup
## create docker volume
    docker run -itd --name cat-hr-webui \
        --network=kong-net \
        -p 3000:80 \
        --rm registry.gitlab.com/cat-hr/cat-hr-webui:latest

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