# kong docker startup (DB LESS)
## create kong network (for future usw cases)
    docker network create kong-net
## create docker volume
    docker volume create kong-vol
## create king config file
/var/lib/docker/volumes/kong-vol/_data/kong.yml

https://docs.konghq.com/1.2.x/db-less-and-declarative-config/#the-declarative-configuration-format

stored into /config/kong/kong.yml

could be checked:
    kong config -c kong.conf parse kong.yml
    
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
         -p 8000:8000 \
         -p 8443:8443 \
         -p 8001:8001 \
         -p 8444:8444 \
         kong:latest